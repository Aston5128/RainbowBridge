# RainbowBridge

基于 [ACL4SSR](https://github.com/ACL4SSR/ACL4SSR) 调整的个人分流配置，适用于通过 subconverter 生成 Clash、OpenClash 和 Stash 配置。

## 配置入口

| 客户端 | 远程配置 |
| --- | --- |
| Clash / OpenClash | [`RainbowBridge.ini`](https://raw.githubusercontent.com/Aston5128/RainbowBridge/main/RainbowBridge.ini) |
| Stash | [`RainbowBridgeForStash.ini`](https://raw.githubusercontent.com/Aston5128/RainbowBridge/main/RainbowBridgeForStash.ini) |

将对应链接填入 subconverter 的远程配置地址即可。OpenClash 与普通 Clash 共用 `RainbowBridge.ini`。

## 主要特性

- 常用服务、AI、流媒体、游戏平台及地区节点分组；
- 自定义直连、代理、广告拦截和固定 IP 规则；
- Clash / OpenClash 对国内域名使用国内 DNS，其他域名默认使用 Cloudflare / Google DoH；
- Clash / OpenClash 使用 HTTPS 地址进行节点健康检查；Stash 保留独立且已验证的 DNS 与测速配置。

## OpenClash 注意事项

- 保持 DNS 劫持和 Fake-IP 启用；
- 关闭 `respect-rules`，避免 DNS 上游连接被二次分流；
- 关闭“自定义上游 DNS 服务器”总开关，让配置文件中的 DNS 设置生效；
- 如果局域网没有完整代理 IPv6，建议同时关闭 IPv6 DNS。

本项目以个人使用需求为主，规则更新后请先自行验证。上游规则版权归原项目所有。
