

  CLI 命令

  
# status

针对频道和会话的诊断。

```bash
openclaw status
openclaw status --all
openclaw status --deep
openclaw status --usage
```

说明：

-   `--deep` 运行实时探测（WhatsApp Web + Telegram + Discord + Google Chat + Slack + Signal）。
-   当配置了多个代理时，输出包含每个代理的会话存储信息。
-   概览包含网关和节点主机服务的安装/运行时状态（如果可用）。
-   概览包含更新频道和 git SHA（针对源码检出）。
-   更新信息显示在概览中；如果有可用更新，status 会提示运行 `openclaw update`（参见[更新](../install/updating.md)）。
-   只读状态表面（`status`、`status --json`、`status --all`）会尽可能解析其目标配置路径所支持的 SecretRefs。
-   如果配置了支持的频道 SecretRef 但在当前命令路径中不可用，status 将保持只读并报告降级的输出，而不是崩溃。人工输出会显示警告，例如“配置的令牌在此命令路径中不可用”，JSON 输出则包含 `secretDiagnostics`。

[skills](./skills.md)[system](./system.md)

---