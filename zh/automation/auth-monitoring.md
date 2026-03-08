

  自动化

  
# 认证监控

OpenClaw 通过 `openclaw models status` 暴露 OAuth 过期健康状态。可将其用于自动化和告警；脚本是针对手机工作流的可选附加项。

## 首选：CLI 检查（便携）

```bash
openclaw models status --check
```

退出码：

-   `0`: 正常
-   `1`: 凭证已过期或缺失
-   `2`: 即将过期（24 小时内）

此方法适用于 cron/systemd，且无需额外脚本。

## 可选脚本（运维 / 手机工作流）

这些脚本位于 `scripts/` 目录下，是**可选的**。它们假设对网关主机具有 SSH 访问权限，并针对 systemd + Termux 进行了调优。

-   `scripts/claude-auth-status.sh` 现在使用 `openclaw models status --json` 作为事实来源（如果 CLI 不可用则回退到直接读取文件），因此请确保 `openclaw` 在 `PATH` 中以便定时器使用。
-   `scripts/auth-monitor.sh`: cron/systemd 定时器的目标；发送告警（ntfy 或手机）。
-   `scripts/systemd/openclaw-auth-monitor.{service,timer}`: systemd 用户定时器。
-   `scripts/claude-auth-status.sh`: Claude Code + OpenClaw 认证检查器（完整/json/简单模式）。
-   `scripts/mobile-reauth.sh`: 通过 SSH 的引导式重新认证流程。
-   `scripts/termux-quick-auth.sh`: 一键小部件状态检查 + 打开认证 URL。
-   `scripts/termux-auth-widget.sh`: 完整的引导式小部件流程。
-   `scripts/termux-sync-widget.sh`: 同步 Claude Code 凭证 → OpenClaw。

如果您不需要手机自动化或 systemd 定时器，可以跳过这些脚本。

[轮询](./poll.md)[节点](../nodes.md)