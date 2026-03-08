

  CLI 命令

  
# health

从正在运行的网关获取健康状态信息。

```bash
openclaw health
openclaw health --json
openclaw health --verbose
```

说明：

-   `--verbose` 选项会运行实时探测，并在配置了多个账户时打印各账户的计时信息。
-   当配置了多个代理时，输出会包含各代理的会话存储信息。

[gateway](./gateway.md)[hooks](./hooks.md)

---