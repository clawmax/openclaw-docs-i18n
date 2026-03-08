

  CLI 命令

  
# cron

管理 Gateway 调度器的 cron 任务。相关：

-   Cron 任务：[Cron 任务](../automation/cron-jobs.md)

提示：运行 `openclaw cron --help` 查看完整的命令界面。注意：隔离的 `cron add` 任务默认使用 `--announce` 交付。使用 `--no-deliver` 将输出保留在内部。`--deliver` 作为 `--announce` 的已弃用别名保留。注意：一次性（`--at`）任务默认在成功后删除。使用 `--keep-after-run` 来保留它们。注意：循环任务现在在连续错误后使用指数退避重试（30s → 1m → 5m → 15m → 60m），然后在下次成功运行后恢复正常计划。注意：保留/清理由配置控制：

-   `cron.sessionRetention`（默认 `24h`）清理已完成的隔离运行会话。
-   `cron.runLog.maxBytes` + `cron.runLog.keepLines` 清理 `~/.openclaw/cron/runs/.jsonl`。

## 常见编辑

在不更改消息的情况下更新交付设置：

```bash
openclaw cron edit <job-id> --announce --channel telegram --to "123456789"
```

禁用隔离任务的交付：

```bash
openclaw cron edit <job-id> --no-deliver
```

为隔离任务启用轻量级引导上下文：

```bash
openclaw cron edit <job-id> --light-context
```

向特定频道通知：

```bash
openclaw cron edit <job-id> --announce --channel slack --to "channel:C1234567890"
```

创建具有轻量级引导上下文的隔离任务：

```bash
openclaw cron add \
  --name "轻量级晨间简报" \
  --cron "0 7 * * *" \
  --session isolated \
  --message "总结夜间更新。" \
  --light-context \
  --no-deliver
```

`--light-context` 仅适用于隔离的 agent-turn 任务。对于 cron 运行，轻量级模式保持引导上下文为空，而不是注入完整的工作空间引导集。

[配置](./configure.md)[守护进程](./daemon.md)