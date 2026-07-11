# RainbowBridge

Based on [ACL4SSR Rule](https://github.com/ACL4SSR/ACL4SSR/tree/master)

## OpenClash DNS 分流

Clash 和 OpenClash 统一使用 `RainbowBridge.ini`。
其引用的 [Mihomo DNS 配置](yaml/RainbowBridgeClashConfig.yaml) 已恢复为 `main` 的稳定基线：

- 移除明文国际 fallback 的实验导致 X/YouTube 解析异常，表明当前网络中的 DoT/DoH fallback 未成功工作；
- 此分支暂时仅用于保留验证记录，不应在解决加密 fallback 可达性之前合并。

`codex/dns-routing-test` 为 DNS 分流验证分支，其 `RainbowBridge.ini` 仅引用该分支内的 YAML，不影响 `main` 用户。
