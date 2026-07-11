# RainbowBridge

Based on [ACL4SSR Rule](https://github.com/ACL4SSR/ACL4SSR/tree/master)

## OpenClash DNS 分流

Clash 和 OpenClash 统一使用 `RainbowBridge.ini`。
其引用的 [Mihomo DNS 配置](yaml/RainbowBridgeClashConfig.yaml) 正在分阶段验证：

完整的现象、排查过程、当前配置、风险和回滚方法见 [OpenClash DNS 泄露排查与修复记录](docs/openclash-dns-leak-notes.md)。

- OpenClash 必须关闭 `respect-rules`，避免上游 DNS 连接被二次分流；
- 普通域名的默认 `nameserver` 已从国内 DoH 改为 Cloudflare/Google DoH；
- `proxy-server-nameserver` 仅使用 `8.8.8.8/1.1.1.1` 解析代理节点域名，避免节点冷启动依赖 DoH；它不参与普通网站查询；
- 国内 `nameserver-policy`、`direct-nameserver`、fallback 和 fallback-filter 均保持 `main` 的配置；
- 继续观察是否会由明文 fallback 或其他旁路再次暴露大陆 DNS 解析器；
- 节点健康检查改用 `https://cp.cloudflare.com/generate_204`，避免 Apple HTTP 测速地址受 DNS/CDN 调度影响。

`codex/dns-routing-test` 为 DNS 分流验证分支，其 `RainbowBridge.ini` 仅引用该分支内的 YAML，不影响 `main` 用户。
