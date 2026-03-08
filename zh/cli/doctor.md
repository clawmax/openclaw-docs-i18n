

  CLI 命令

  
# doctor

针对网关和通道的健康检查与快速修复。相关链接：

-   故障排除：[故障排除](../gateway/troubleshooting.md)
-   安全审计：[安全](../gateway/security.md)

## 示例

```bash
openclaw doctor
openclaw doctor --repair
openclaw doctor --deep
```

注意事项：

-   交互式提示（如钥匙串/OAuth 修复）仅在标准输入是 TTY 且**未**设置 `--non-interactive` 时运行。无头运行（cron、Telegram、无终端）将跳过提示。
-   `--fix`（`--repair` 的别名）会将备份写入 `~/.openclaw/openclaw.json.bak` 并删除未知的配置键，列出每个被移除的项。
-   状态完整性检查现在可以检测会话目录中的孤立转录文件，并能将其归档为 `.deleted.<时间戳>` 以安全回收空间。
-   Doctor 包含内存搜索就绪检查，并能在缺少嵌入凭据时推荐运行 `openclaw configure --section model`。
-   如果启用了沙箱模式但 Docker 不可用，doctor 会报告一个高优先级警告并提供修复建议（`安装 Docker` 或 `openclaw config set agents.defaults.sandbox.mode off`）。

## macOS：launchctl 环境变量覆盖

如果您之前运行过 `launchctl setenv OPENCLAW_GATEWAY_TOKEN ...`（或 `...PASSWORD`），该值将覆盖您的配置文件，并可能导致持续的“未授权”错误。

```bash
launchctl getenv OPENCLAW_GATEWAY_TOKEN
launchctl getenv OPENCLAW_GATEWAY_PASSWORD

launchctl unsetenv OPENCLAW_GATEWAY_TOKEN
launchctl unsetenv OPENCLAW_GATEWAY_PASSWORD
```

[文档](./docs.md)[网关](./gateway.md)