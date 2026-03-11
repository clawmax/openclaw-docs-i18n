

  自动化

  
# 定时任务（Cron）与心跳（Heartbeat）

心跳和定时任务都能帮你按计划执行任务，但适用场景不同。本指南帮你快速做出正确选择。

## 快速决策指南

| 场景 | 推荐方案 | 原因 |
| --- | --- | --- |
| 每30分钟检查收件箱 | 心跳 | 可与其他检查合并，具备上下文感知能力 |
| 每天上午9点准时发送日报 | 定时任务（独立会话） | 需要精确的时间点 |
| 监控日历中的即将到来的事件 | 心跳 | 天然适合周期性感知任务 |
| 每周运行一次深度分析 | 定时任务（独立会话） | 独立任务，可使用不同的模型 |
| 20分钟后提醒我 | 定时任务（主会话，`--at`） | 一次性任务，需要精确计时 |
| 后台项目健康检查 | 心跳 | 复用现有的心跳周期 |

## 心跳：周期性感知

心跳在**主会话**中以固定间隔运行（默认30分钟），让智能体定期检查各项事务并及时提醒你注意重要信息。

### 什么时候用心跳

- **多个周期性检查**：与其创建5个独立的定时任务分别检查收件箱、日历、天气、通知和项目状态，不如用一个心跳批量完成所有检查。
- **需要上下文感知**：智能体拥有完整的主会话上下文，能智能判断哪些事情紧急、哪些可以等等。
- **保持对话连贯**：心跳共享同一个会话，智能体能记住最近的对话内容并自然跟进。
- **低开销监控**：一个心跳替代多个小型轮询任务。

### 心跳的优势

- **批量检查**：一次智能体轮次就能同时处理收件箱、日历和通知。
- **节省 API 调用**：一个心跳比5个独立定时任务成本更低。
- **上下文感知**：智能体知道你最近在做什么，能据此调整优先级。
- **智能静默**：如果没有需要关注的事项，智能体会返回 `HEARTBEAT_OK`，不会发送任何消息打扰你。
- **弹性时间**：执行时间会根据队列负载略有浮动，这对大多数监控场景来说完全没问题。

### 心跳示例：HEARTBEAT.md 清单

```bash
# Heartbeat checklist

- Check email for urgent messages
- Review calendar for events in next 2 hours
- If a background task finished, summarize results
- If idle for 8+ hours, send a brief check-in
```

智能体在每次心跳时读取这份清单，一次性处理所有条目。

### 配置心跳

```json
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m", // 执行间隔
        target: "last", // 指定提醒发送目标（默认为 "none"）
        activeHours: { start: "08:00", end: "22:00" }, // 可选，限定活跃时段
      },
    },
  },
}
```

完整配置说明请参阅 [心跳](../gateway/heartbeat.md)。

## 定时任务（Cron）：精确调度

定时任务在精确的时间点运行，可以在独立会话中执行而不影响主会话上下文。对于重复执行的整点任务，系统会自动在0-5分钟的窗口内按确定性偏移量分散执行，避免负载集中。

### 什么时候用定时任务

- **必须精确计时**：比如"每周一上午9:00整发送"，而不是"9点左右"。
- **独立任务**：不需要对话上下文的任务。
- **使用不同模型或思考深度**：比如需要 Opus 进行深度分析的任务。
- **一次性提醒**：用 `--at` 参数设置"20分钟后提醒我"。
- **频繁或嘈杂的任务**：可能会刷屏主会话历史的任务。
- **外部触发**：不管智能体是否活跃都应该独立运行的任务。

### 定时任务的优势

- **精确计时**：支持5字段或6字段（含秒）的 cron 表达式，支持时区设置。
- **内置负载分散**：重复的整点任务默认会错开最多5分钟执行。
- **精细控制**：用 `--stagger ` 自定义错开时长，或用 `--exact` 强制精确执行。
- **会话隔离**：在 `cron:` 会话中运行，不会污染主会话历史。
- **模型覆盖**：每个任务可以单独指定更便宜或更强大的模型。
- **交付控制**：独立任务默认使用 `announce` 模式（发送摘要），也可设置为 `none`。
- **即时推送**：announce 模式会直接发送，不用等下次心跳。
- **不依赖主会话**：即使主会话空闲或被压缩也能正常运行。
- **支持一次性任务**：用 `--at` 指定精确的未来时间戳。

### 定时任务示例：每日早间简报

```bash
openclaw cron add \
  --name "Morning briefing" \
  --cron "0 7 * * *" \
  --tz "America/New_York" \
  --session isolated \
  --message "Generate today's briefing: weather, calendar, top emails, news summary." \
  --model opus \
  --announce \
  --channel whatsapp \
  --to "+15551234567"
```

这条命令会在纽约时间早上7:00整运行，使用 Opus 模型确保质量，并将摘要直接发送到 WhatsApp。

### 定时任务示例：一次性提醒

```bash
openclaw cron add \
  --name "Meeting reminder" \
  --at "20m" \
  --session main \
  --system-event "Reminder: standup meeting starts in 10 minutes." \
  --wake now \
  --delete-after-run
```

完整 CLI 参考请参阅 [定时任务](./cron-jobs.md)。

## 决策流程图

```
任务需要在精确时间点运行？
  是 -> 使用定时任务
  否 -> 继续...

任务需要与主会话隔离？
  是 -> 使用定时任务（独立会话）
  否 -> 继续...

这个任务能和其他周期性检查合并吗？
  是 -> 使用心跳（加入 HEARTBEAT.md）
  否 -> 使用定时任务

这是一次性提醒？
  是 -> 使用定时任务（带 --at 参数）
  否 -> 继续...

需要使用不同的模型或思考深度？
  是 -> 使用定时任务（独立会话，带 --model/--thinking 参数）
  否 -> 使用心跳
```

## 两者结合使用

最高效的方案是**两者配合**：

1. **心跳**每30分钟批量处理日常监控（收件箱、日历、通知）。
2. **定时任务**处理精确时间点的任务（日报、周报）和一次性提醒。

### 示例：高效的自动化配置

**HEARTBEAT.md**（每30分钟检查）：

```bash
# Heartbeat checklist

- Scan inbox for urgent emails
- Check calendar for events in next 2h
- Review any pending tasks
- Light check-in if quiet for 8+ hours
```

**定时任务**（精确计时）：

```bash
# 每天早上7点发送早间简报
openclaw cron add --name "Morning brief" --cron "0 7 * * *" --session isolated --message "..." --announce

# 每周一早上9点进行项目复盘
openclaw cron add --name "Weekly review" --cron "0 9 * * 1" --session isolated --message "..." --model opus

# 一次性提醒
openclaw cron add --name "Call back" --at "2h" --session main --system-event "Call back the client" --wake now
```

## Lobster：带审批的确定性工作流

Lobster 是专为**多步骤工具流水线**设计的工作流运行时，支持确定性执行和显式审批。当任务不只是一次智能体交互，而是需要可恢复的工作流和人工检查点时，就适合用 Lobster。

### Lobster 的适用场景

- **多步骤自动化**：需要固定的工具调用流水线，而不是一次性的提示词。
- **审批关卡**：涉及副作用的操作应该暂停等待你的批准后再继续。
- **可恢复执行**：继续之前暂停的工作流，无需重跑前面的步骤。

### 与心跳和定时任务的关系

- **心跳/定时任务**决定*何时*触发执行。
- **Lobster**定义执行开始后*运行哪些步骤*。

对于定时工作流，用定时任务或心跳触发一个调用 Lobster 的智能体轮次。对于临时工作流，直接调用 Lobster 即可。

### 技术说明

- Lobster 作为**本地子进程**运行（`lobster` CLI），工作在工具模式下，返回 **JSON 信封**格式。
- 如果工具返回 `needs_approval`，你可以用 `resumeToken` 和 `approve` 标志恢复执行。
- Lobster 是**可选插件**，建议通过 `tools.alsoAllow: ["lobster"]` 按需启用。
- 系统需要在 `PATH` 中能找到 `lobster` CLI。

完整用法和示例请参阅 [Lobster](../tools/lobster.md)。

## 主会话 vs 独立会话

心跳和定时任务都可以与主会话交互，但方式不同：

|  | 心跳 | 定时任务（主会话） | 定时任务（独立会话） |
| --- | --- | --- | --- |
| 会话 | 主会话 | 主会话（通过系统事件） | `cron:` |
| 历史记录 | 共享 | 共享 | 每次运行都是全新的 |
| 上下文 | 完整 | 完整 | 无（从零开始） |
| 模型 | 使用主会话模型 | 使用主会话模型 | 可单独覆盖 |
| 输出 | 非 `HEARTBEAT_OK` 时发送 | 心跳提示 + 事件 | announce 摘要（默认） |

### 什么时候用主会话定时任务

当你希望以下情况时，使用 `--session main` 配合 `--system-event`：

- 提醒/事件出现在主会话上下文中
- 智能体在下次心跳时处理它，拥有完整上下文
- 不需要单独的独立会话运行

```bash
openclaw cron add \
  --name "Check project" \
  --every "4h" \
  --session main \
  --system-event "Time for a project health check" \
  --wake now
```

### 什么时候用独立会话定时任务

当你希望以下情况时，使用 `--session isolated`：

- 干净的初始状态，不携带历史上下文
- 使用不同的模型或思考深度设置
- 将摘要直接公告到频道
- 历史记录不影响主会话

```bash
openclaw cron add \
  --name "Deep analysis" \
  --cron "0 6 * * 0" \
  --session isolated \
  --message "Weekly codebase analysis..." \
  --model opus \
  --thinking high \
  --announce
```

## 成本考量

| 机制 | 成本特点 |
| --- | --- |
| 心跳 | 每 N 分钟执行一次轮次；成本随 HEARTBEAT.md 大小增长 |
| 定时任务（主会话） | 仅在下次心跳添加事件（无独立轮次） |
| 定时任务（独立会话） | 每个任务一次完整智能体轮次；可使用更便宜的模型 |

**优化建议**：

- 保持 `HEARTBEAT.md` 简洁，减少 token 开销。
- 把类似的检查合并到心跳中，而不是创建多个定时任务。
- 如果只需要内部处理，在心跳配置中设置 `target: "none"`。
- 对于常规任务，使用独立会话定时任务配合更便宜的模型。

## 相关链接

- [心跳](../gateway/heartbeat.md) - 完整的心跳配置说明
- [定时任务](./cron-jobs.md) - 完整的定时任务 CLI 和 API 参考
- [系统](../cli/system.md) - 系统事件与心跳控制

[定时任务](./cron-jobs.md)[自动化故障排除](./troubleshooting.md)