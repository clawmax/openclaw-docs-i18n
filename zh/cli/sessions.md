

  CLI 命令

  
# sessions

列出存储的对话会话。

```bash
openclaw sessions
openclaw sessions --agent work
openclaw sessions --all-agents
openclaw sessions --active 120
openclaw sessions --json
```

范围选择：

-   default: 配置的默认智能体存储
-   `--agent `: 一个已配置的智能体存储
-   `--all-agents`: 聚合所有已配置的智能体存储
-   `--store `: 显式指定存储路径（不能与 `--agent` 或 `--all-agents` 组合使用）

JSON 示例：`openclaw sessions --all-agents --json`：

```json
{
  "path": null,
  "stores": [
    { "agentId": "main", "path": "/home/user/.openclaw/agents/main/sessions/sessions.json" },
    { "agentId": "work", "path": "/home/user/.openclaw/agents/work/sessions/sessions.json" }
  ],
  "allAgents": true,
  "count": 2,
  "activeMinutes": null,
  "sessions": [
    { "agentId": "main", "key": "agent:main:main", "model": "gpt-5" },
    { "agentId": "work", "key": "agent:work:main", "model": "claude-opus-4-5" }
  ]
}
```

## 清理维护

立即运行维护（而不是等待下一个写入周期）：

```bash
openclaw sessions cleanup --dry-run
openclaw sessions cleanup --agent work --dry-run
openclaw sessions cleanup --all-agents --dry-run
openclaw sessions cleanup --enforce
openclaw sessions cleanup --enforce --active-key "agent:main:telegram:dm:123"
openclaw sessions cleanup --json
```

`openclaw sessions cleanup` 使用配置文件中的 `session.maintenance` 设置：

-   范围说明：`openclaw sessions cleanup` 仅维护会话存储/记录。它不会清理 cron 运行日志 (`cron/runs/.jsonl`)，这些日志由 [Cron 配置](../automation/cron-jobs.md#configuration) 中的 `cron.runLog.maxBytes` 和 `cron.runLog.keepLines` 管理，并在 [Cron 维护](../automation/cron-jobs.md#maintenance) 中说明。
-   `--dry-run`: 预览将有多少条目会被修剪/限制，而不实际写入。
    -   在文本模式下，dry-run 会打印一个按会话的操作表（`Action`, `Key`, `Age`, `Model`, `Flags`），以便您查看哪些会被保留，哪些会被移除。
-   `--enforce`: 即使 `session.maintenance.mode` 为 `warn`，也强制执行维护。
-   `--active-key `: 保护特定的活动键，使其免受磁盘预算驱逐。
-   `--agent `: 为一个已配置的智能体存储运行清理。
-   `--all-agents`: 为所有已配置的智能体存储运行清理。
-   `--store `: 针对特定的 `sessions.json` 文件运行。
-   `--json`: 打印 JSON 摘要。使用 `--all-agents` 时，输出包含每个存储的摘要。

`openclaw sessions cleanup --all-agents --dry-run --json`：

```json
{
  "allAgents": true,
  "mode": "warn",
  "dryRun": true,
  "stores": [
    {
      "agentId": "main",
      "storePath": "/home/user/.openclaw/agents/main/sessions/sessions.json",
      "beforeCount": 120,
      "afterCount": 80,
      "pruned": 40,
      "capped": 0
    },
    {
      "agentId": "work",
      "storePath": "/home/user/.openclaw/agents/work/sessions/sessions.json",
      "beforeCount": 18,
      "afterCount": 18,
      "pruned": 0,
      "capped": 0
    }
  ]
}
```

相关：

-   会话配置：[配置参考](../gateway/configuration-reference.md#session)

[security](./security.md)[setup](./setup.md)