# RainbowBridge

Based on [ACL4SSR Rule](https://github.com/ACL4SSR/ACL4SSR/tree/master)

## OpenClash DNS 分流

Clash 和 OpenClash 统一使用 `RainbowBridge.ini`。
其引用的 [Mihomo DNS 配置](yaml/RainbowBridgeClashConfig.yaml) 正在进行单变量验证：

- OpenClash 必须关闭 `respect-rules`，避免上游 DNS 连接被二次分流；
- 当前只将默认 `nameserver` 从国内 DoH 改为 Cloudflare/Google DoH；
- 新增 `proxy-server-nameserver` 使用与 `main` 原 nameserver 一致的阿里/360 DoH 解析代理节点域名，不参与普通网站查询；
- 国内 `nameserver-policy`、`direct-nameserver`、fallback 和 fallback-filter 均保持 `main` 的配置；
- 用于判断大陆 DNS 记录来自默认 nameserver，还是明文 fallback 被运营商接管。

`codex/dns-routing-test` 为 DNS 分流验证分支，其 `RainbowBridge.ini` 仅引用该分支内的 YAML，不影响 `main` 用户。
