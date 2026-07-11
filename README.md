# RainbowBridge

Based on [ACL4SSR Rule](https://github.com/ACL4SSR/ACL4SSR/tree/master)

## OpenClash DNS 分流

Clash 和 OpenClash 统一使用 `RainbowBridge.ini`。
其引用的 [Mihomo DNS 配置](yaml/RainbowBridgeClashConfig.yaml) 采用：

- 国内及私有域名通过阿里/DNSPod UDP DNS 直连解析；
- 国外及未分类域名通过 Cloudflare DoH/443 解析，并强制经 `🚀 节点选择` 出口；
- 代理节点域名使用国内 DNS 直连解析，避免启动循环；
- 不使用 `fallback` 和 `direct-nameserver`，防止国外域名同时触达国内 DNS。

DoH 查询经代理节点的 HTTPS/443 连接发送，优先保证各类节点出口的兼容性。
如果 `🚀 节点选择` 没有可用节点，国外域名将无法解析。

`codex/dns-routing-test` 为 DNS 分流验证分支，其 `RainbowBridge.ini` 仅引用该分支内的 YAML，不影响 `main` 用户。
