# RainbowBridge

Based on [ACL4SSR Rule](https://github.com/ACL4SSR/ACL4SSR/tree/master)

## OpenClash DNS 分流

Clash 和 OpenClash 统一使用 `RainbowBridge.ini`。
其引用的 [Mihomo DNS 配置](yaml/RainbowBridgeClashConfig.yaml) 当前处于分阶段验证：

- 第一阶段恢复 `main` 的 nameserver / fallback / direct-nameserver 行为；
- fallback 仅移除 `8.8.8.8` 和 `1.1.1.1` 的明文 UDP 入口；
- 其他国内 DNS、DoT/DoH 和 fallback-filter 均保持不变，用于确认兼容性。

`codex/dns-routing-test` 为 DNS 分流验证分支，其 `RainbowBridge.ini` 仅引用该分支内的 YAML，不影响 `main` 用户。
