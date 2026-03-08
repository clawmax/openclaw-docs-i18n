

  自动化

  
# 自动化故障排除

本页面用于调度器和交付问题 (`cron` + `heartbeat`)。

## 命令阶梯

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

然后运行自动化检查：

```bash
openclaw cron status
openclaw cron list
openclaw system heartbeat last
```

## Cron 未触发

```bash
openclaw cron status
openclaw cron list
openclaw cron runs --id <jobId> --limit 20
openclaw logs --follow
```

良好的输出应如下所示：

-   `cron status` 报告为启用状态，并显示未来的 `nextWakeAtMs`。
-   任务已启用并具有有效的计划/时区。
-   `cron runs` 显示 `ok` 或明确的跳过原因。

常见特征：

-   `cron: scheduler disabled; jobs will not run automatically` → cron 在配置/环境变量中被禁用。
-   `cron: timer tick failed` → 调度器计时器触发失败；检查周围的堆栈/日志上下文。
-   `reason: not-due` 出现在运行输出中 → 手动运行未使用 `--force` 参数，且任务尚未到期。

## Cron 已触发但无交付

```bash
openclaw cron runs --id <jobId> --limit 20
openclaw cron list
openclaw channels status --probe
openclaw logs --follow
```

良好的输出应如下所示：

-   运行状态为 `ok`。
-   独立任务已设置交付模式/目标。
-   通道探测报告目标通道已连接。

常见特征：

-   运行成功但交付模式为 `none` → 预期不会产生外部消息。
-   交付目标缺失或无效 (`channel`/`to`) → 运行可能在内部成功，但跳过了外发。
-   通道认证错误 (`unauthorized`, `missing_scope`, `Forbidden`) → 交付被通道凭据/权限阻止。

## Heartbeat 被抑制或跳过

```bash
openclaw system heartbeat last
openclaw logs --follow
openclaw config get agents.defaults.heartbeat
openclaw channels status --probe
```

良好的输出应如下所示：

-   Heartbeat 已启用，且间隔时间非零。
-   最后一次 heartbeat 结果为 `ran`（或跳过的原因明确）。

常见特征：

-   `heartbeat skipped` 且 `reason=quiet-hours` → 处于 `activeHours` 之外。
-   `requests-in-flight` → 主通道繁忙；heartbeat 被推迟。
-   `empty-heartbeat-file` → 间隔 heartbeat 被跳过，因为 `HEARTBEAT.md` 没有可操作内容且没有标记的 cron 事件在队列中。
-   `alerts-disabled` → 可见性设置抑制了外发的 heartbeat 消息。

## 时区和 activeHours 陷阱

```bash
openclaw config get agents.defaults.heartbeat.activeHours
openclaw config get agents.defaults.heartbeat.activeHours.timezone
openclaw config get agents.defaults.userTimezone || echo "agents.defaults.userTimezone not set"
openclaw cron list
openclaw logs --follow
```

快速规则：

-   `Config path not found: agents.defaults.userTimezone` 表示该键未设置；heartbeat 回退到主机时区（如果设置了 `activeHours.timezone` 则使用该时区）。
-   没有 `--tz` 参数的 cron 使用网关主机时区。
-   Heartbeat 的 `activeHours` 使用配置的时区解析（`user`、`local` 或显式的 IANA 时区）。
-   没有时区的 ISO 时间戳在 cron 的 `at` 计划中被视为 UTC。

常见特征：

-   主机时区更改后，任务在错误的实际时间运行。
-   由于 `activeHours.timezone` 设置错误，在您的白天时段 heartbeat 总是被跳过。

相关链接：

-   [/automation/cron-jobs](./cron-jobs.md)
-   [/gateway/heartbeat](../gateway/heartbeat.md)
-   [/automation/cron-vs-heartbeat](./cron-vs-heartbeat.md)
-   [/concepts/timezone](../concepts/timezone.md)

[Cron 与 Heartbeat 对比](./cron-vs-heartbeat.md)[Webhooks](./webhook.md)