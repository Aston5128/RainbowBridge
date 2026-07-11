# OpenClash DNS 泄露排查与修复记录

> 记录日期：2026-07-11  
> 验证分支：`codex/dns-routing-test`  
> 状态：继续观察，暂不合并到 `main`

## 背景与目标

RainbowBridge 的 Clash 与 OpenClash 已统一使用 `RainbowBridge.ini`。这次调整希望同时满足以下目标：

1. OpenClash 统一接管局域网 DNS，浏览器和终端不绕过 Mihomo；
2. 国内域名继续使用国内 DNS，以保留低延迟和较好的 CDN 调度；
3. 其他域名默认使用国际加密 DNS，避免检测站看到中国大陆运营商解析器；
4. 不破坏代理节点启动、节点延迟测试，以及 X、YouTube、Google、ChatGPT 等服务。

这不是“所有域名只使用境外 DNS”的方案。国内域名仍会按策略交给国内 DNS，这是为了响应速度而保留的设计取舍。

## 最初现象

出口 IP 位于香港，但 DNS 泄露检测同时发现了中国移动解析器，例如：

```text
111.1.165.55 / China Mobile / China Jiaxing
```

这说明检测域名的查询仍有机会进入国内 DNS 路径。需要注意，检测站所说的“泄露”通常表示解析器所在地与代理出口不一致；它不能完整证明每一种域名的 DNS 路由，也不能证明浏览器之外不存在其他 DNS 请求。

## 排查过程中遇到的问题

### 1. 让国际 DNS 强制经过代理组

早期尝试把国际 DNS 的连接强制交给代理节点，希望查询从节点出口发往 Cloudflare 或 Google。结果形成了启动依赖：

```text
解析节点域名 -> 需要国际 DoH -> DoH 需要代理节点 -> 节点尚未完成解析
```

表现为 X 无法打开、YouTube 页面或缩略图加载失败、ChatGPT 无法访问。这个方向已撤回。

### 2. `respect-rules` 与 DNS 的二次分流

OpenClash 开启 `respect-rules` 后，上游 DNS 连接也会遵循配置中的流量规则，同时要求存在 `proxy-server-nameserver`。在这套配置里，它会让 DNS 启动顺序和代理组选择相互影响，并触发 OpenClash 自动覆写相关设置。

最终验证中必须关闭：

```text
覆写设置 -> DNS 设置 -> 遵循规则（respect-rules）
```

### 3. 节点可以使用，但延迟测不出来

节点域名的解析与普通网站 DNS 混在一起时，代理节点可能能复用已有连接，却无法稳定完成新一轮健康检查。为此增加了独立的 `proxy-server-nameserver`，只负责代理服务器域名的冷启动解析。

此外，原来的 Apple HTTP 测速地址容易受 DNS、CDN 调度或 HTTP 路径影响，所有 `url-test` 和 CFW 延迟测试已统一改为：

```text
https://cp.cloudflare.com/generate_204
```

### 4. 同名节点不一定代表同一个实际出口

ChatGPT 曾返回 Cloudflare 403。对比 `https://chatgpt.com/cdn-cgi/trace` 后发现，虽然界面都显示“新加坡_03”，实际出口并不相同：

```text
异常时：155.254.106.69，colo=HKG，loc=HK
正常时：141.11.43.117，colo=SIN，loc=SG
```

最终测试分支重新回到 `141.11.43.117` 后，ChatGPT 恢复正常。因此这次 403 更接近出口 IP、节点选择状态或 Cloudflare 风控问题，不能简单归因于 DNS。

### 5. QUIC 不是本次根因

OpenClash 和浏览器中的 QUIC 原本就处于关闭状态。关闭 QUIC 一度让 YouTube 页面框架可以打开，但无法修复缩略图和视频请求，因此它只是排除了 UDP/443 路径问题，不是最终修复项。

## 当前有效配置

实际配置位于 [`yaml/RainbowBridgeClashConfig.yaml`](../yaml/RainbowBridgeClashConfig.yaml)。核心结构如下：

```yaml
dns:
  enable: true
  ipv6: false
  enhanced-mode: fake-ip

  nameserver-policy:
    geosite:private,cn: &cn_dns
      - 223.5.5.5
      - 119.29.29.29
      - 117.50.10.10
      - 114.114.114.114

  nameserver:
    - https://cloudflare-dns.com/dns-query
    - https://dns.google/dns-query

  proxy-server-nameserver:
    - 8.8.8.8
    - 1.1.1.1

  direct-nameserver:
    - 223.5.5.5
    - 119.29.29.29
    - 117.50.10.10
    - 114.114.114.114
  direct-nameserver-follow-policy: true
```

各字段的职责：

| 配置项 | 用途 | 当前取舍 |
| --- | --- | --- |
| `nameserver-policy` | 国内、私有及指定腾讯/微信域名 | 国内 DNS，优先速度和 CDN 命中 |
| `nameserver` | 未命中专用策略的普通域名 | Cloudflare/Google DoH |
| `direct-nameserver` | 直连流量的域名解析 | 国内 DNS |
| `proxy-server-nameserver` | 只解析代理节点服务器域名 | 明文公共 DNS，解除节点冷启动依赖 |
| `default-nameserver` | 解析 DoH 服务器自身域名等引导场景 | 保留国内 DNS，保证启动可用性 |

`proxy-server-nameserver` 不参与 X、YouTube、ChatGPT 等普通网站的 DNS 查询，但它使用的是明文 UDP DNS。这意味着代理节点的服务器域名仍可能被本地网络观察或干预，是当前为了稳定启动和测速接受的边界，不应把本方案描述成“全链路、全域名无明文 DNS”。

## OpenClash 配套设置

验证时使用以下设置：

- DNS 劫持与 Fake-IP 保持启用；
- IPv6 DNS 关闭；如果局域网没有完整代理 IPv6，也不要让客户端通过 IPv6 DNS 绕过；
- `respect-rules` 关闭；
- “自定义上游 DNS 服务器”总开关关闭，让 YAML 中的 DNS 配置生效；
- 页面下方保存但未启用的自定义 DNS 行不会生效，不需要仅因为这些残留行而删除配置；
- QUIC 保持关闭，但它不是这次 DNS 修复的必要条件。

## 最终验证结果

在 `codex/dns-routing-test` 分支上，当前已确认：

- DNS 泄露检测未再发现中国大陆运营商解析器，检测结果显示 Cloudflare 境外解析器；
- 节点延迟测试恢复；
- X、YouTube、Google 可正常访问；
- ChatGPT 可正常访问；
- ChatGPT trace 为 `ip=141.11.43.117`、`colo=SIN`、`loc=SG`，与 `main` 正常状态一致。

这只是当前网络、节点订阅和 OpenClash 版本下的实测结果。检测站变绿不代表国内域名不再使用国内 DNS，也不代表所有设备和所有协议都已覆盖。

## 风险与观察项

1. 国际 DoH 在部分国内网络可能出现连接慢、握手失败或间歇性不可达；
2. `proxy-server-nameserver` 的明文查询会暴露代理节点域名，并可能受到本地网络干预；
3. 关闭 `respect-rules` 后，DNS 上游连接不按业务分流规则选择代理组，隐私边界依赖具体网络路径；
4. `fallback` 中仍保留历史的明文与加密服务器组合，后续若要严格保证“普通境外域名不产生明文查询”，需要单独进行一次只调整 fallback 的验证；
5. 自动选择组可能切换到地理位置、出口信誉不同的节点，ChatGPT 等受风控服务可能再次返回 403；
6. OpenClash 更新、Mihomo DNS 行为变化或覆写设置变化，都可能改变最终生成的 `Combined.yaml`。

观察期间建议定期检查：

- DNS 检测结果是否重新出现大陆运营商解析器；
- `Combined.yaml` 是否仍包含预期的 `nameserver`、`proxy-server-nameserver` 和 `direct-nameserver`；
- 节点延迟是否可以批量刷新；
- X、YouTube 图片/视频、Google、ChatGPT 是否同时可用；
- ChatGPT 出现 403 时，先记录 `cdn-cgi/trace` 的 `ip`、`colo` 和 `loc`，再判断是 DNS 还是出口问题。

## 合并到 `main` 前的检查清单

- 连续观察一段时间，覆盖重启 OpenClash、更新订阅、切换节点和自动选择；
- 确认至少一个 OpenClash 以外的 Clash/Mihomo 客户端能够读取同一配置；
- 将 `RainbowBridge.ini` 中的 `clash_rule_base` 从测试分支恢复为 `main`；
- 保留 `RainbowBridgeForOpenClash.ini` 的删除，因为 OpenClash 与普通 Clash 已统一使用 `RainbowBridge.ini`；
- 审核 README 和本笔记中的实验状态，再决定是否把“测试中”改为“已发布”；
- 合并后再次从 GitHub Raw 地址生成配置，避免只验证本地文件。

## 回滚方式

测试期间如果再次出现大面积站点故障，可把 OpenClash 配置订阅切回 `main`，重新生成并应用配置。不要只在面板中切换策略组后判断已经回滚，因为旧 DNS 缓存、已有连接和自动选择状态仍可能继续影响结果。
