# RainbowBridge

Based on [ACL4SSR Rule](https://github.com/ACL4SSR/ACL4SSR/tree/master)

## OpenClash DNS 分流

Clash 和 OpenClash 统一使用 `RainbowBridge.ini`。
其引用的 [Mihomo DNS 配置](yaml/RainbowBridgeClashConfig.yaml) 采用：

- 国内及私有域名通过阿里/DNSPod UDP DNS 直连解析；
- 国外及未分类域名通过 Cloudflare/Google DoH/443 直连解析；
- 代理节点域名使用国内 DNS 直连解析，避免启动循环；
- 不使用 `fallback` 和 `direct-nameserver`，防止国外域名同时触达国内 DNS。

DoH 通过 HTTPS/443 加密查询，不依赖代理组，避免切换或故障节点中断国外 DNS。

`codex/dns-routing-test` 为 DNS 分流验证分支，其 `RainbowBridge.ini` 仅引用该分支内的 YAML，不影响 `main` 用户。
