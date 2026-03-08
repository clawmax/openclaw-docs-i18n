

  自动化

  
# Cron 与 心跳

心跳和cron作业都允许您按计划运行任务。本指南帮助您为您的用例选择正确的机制。

## 快速决策指南

| 用例 | 推荐 | 原因 |
| --- | --- | --- |
| 每30分钟检查收件箱 | 心跳 | 与其他检查批量处理，上下文感知 |
| 每天上午9点整发送报告 | Cron（独立） | 需要精确计时 |
| 监控日历中的即将发生事件 | 心跳 | 适合周期性感知 |
| 运行每周深度分析 | Cron（独立） | 独立任务，可使用不同模型 |
| 20分钟后提醒我 | Cron（主会话，`--at`） | 一次性任务，精确计时 |
| 后台项目健康检查 | 心跳 | 利用现有周期 |

## 心跳：周期性感知

心跳在**主会话**中以固定间隔（默认：30分钟）运行。它们的设计目的是让智能体检查事务并呈现任何重要信息。

### 何时使用心跳

-   **多个周期性检查**：与其使用5个独立的cron作业来检查收件箱、日历、天气、通知和项目状态，不如使用单个心跳来批量处理所有这些。
-   **上下文感知决策**：智能体拥有完整的主会话上下文，因此可以智能地决定哪些紧急，哪些可以等待。
-   **对话连续性**：心跳运行共享同一会话，因此智能体记得最近的对话并可以自然地跟进。
-   **低开销监控**：一个心跳可以替代许多小的轮询任务。

### 心跳优势

-   **批量处理多个检查**：一次智能体轮次可以同时审查收件箱、日历和通知。
-   **减少API调用**：单个心跳比5个独立的cron作业成本更低。
-   **上下文感知**：智能体知道您一直在处理什么，并可以相应地进行优先级排序。
-   **智能抑制**：如果无需关注，智能体回复`HEARTBEAT_OK`且不发送消息。
-   **自然计时**：根据队列负载略有漂移，这对大多数监控来说是可以接受的。

### 心跳示例：HEARTBEAT.md 清单

```bash
# Heartbeat checklist

- Check email for urgent messages
- Review calendar for events in next 2 hours
- If a background task finished, summarize results
- If idle for 8+ hours, send a brief check-in
```

智能体在每次心跳时读取此清单，并在一次轮次中处理所有项目。

### 配置心跳

```json
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m", // 间隔
        target: "last", // 明确的警报传递目标（默认为"none"）
        activeHours: { start: "08:00", end: "22:00" }, // 可选
      },
    },
  },
}
```

完整配置请参见[心跳](../gateway/heartbeat.md)。

## Cron：精确调度

Cron作业在精确时间运行，并且可以在独立会话中运行，不影响主上下文。重复的整点计划会自动通过确定性的每作业偏移量在0-5分钟窗口内分散。

### 何时使用cron

-   **需要精确计时**：“每周一上午9:00发送此内容”（而不是“9点左右”）。
-   **独立任务**：不需要对话上下文的任务。
-   **不同模型/思考级别**：需要更强大模型的重度分析。
-   **一次性提醒**：使用`--at`参数设置“20分钟后提醒我”。
-   **嘈杂/频繁任务**：会干扰主会话历史记录的任务。
-   **外部触发**：无论智能体是否处于活动状态都应独立运行的任务。

### Cron优势

-   **精确计时**：支持时区的5字段或6字段（秒）cron表达式。
-   **内置负载分散**：默认情况下，重复的整点计划会错开最多5分钟。
-   **每作业控制**：使用`--stagger `覆盖错开，或使用`--exact`强制精确计时。
-   **会话隔离**：在`cron:`中运行，不会污染主历史记录。
-   **模型覆盖**：每个作业可以使用更便宜或更强大的模型。
-   **交付控制**：独立作业默认为`announce`（摘要）；根据需要选择`none`。
-   **即时交付**：公告模式直接发布，无需等待心跳。
-   **无需智能体上下文**：即使主会话空闲或已压缩，也能运行。
-   **一次性支持**：`--at`用于精确的未来时间戳。

### Cron示例：每日早间简报

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

这将在纽约时间上午7:00整运行，使用Opus模型保证质量，并直接将摘要公告到WhatsApp。

### Cron示例：一次性提醒

```bash
openclaw cron add \
  --name "Meeting reminder" \
  --at "20m" \
  --session main \
  --system-event "Reminder: standup meeting starts in 10 minutes." \
  --wake now \
  --delete-after-run
```

完整CLI参考请参见[Cron作业](./cron-jobs.md)。

## 决策流程图

```
任务是否需要在一个精确时间运行？
  是 -> 使用cron
  否 -> 继续...

任务是否需要与主会话隔离？
  是 -> 使用cron（独立）
  否 -> 继续...

此任务是否可以与其他周期性检查批量处理？
  是 -> 使用心跳（添加到HEARTBEAT.md）
  否 -> 使用cron

这是一个一次性提醒吗？
  是 -> 使用带 --at 的cron
  否 -> 继续...

是否需要不同的模型或思考级别？
  是 -> 使用带 --model/--thinking 的cron（独立）
  否 -> 使用心跳
```

## 结合使用两者

最高效的设置是**两者结合**使用：

1.  **心跳**每30分钟在一个批量轮次中处理常规监控（收件箱、日历、通知）。
2.  **Cron**处理精确计划（每日报告、每周回顾）和一次性提醒。

### 示例：高效自动化设置

**HEARTBEAT.md**（每30分钟检查）：

```bash
# Heartbeat checklist

- Scan inbox for urgent emails
- Check calendar for events in next 2h
- Review any pending tasks
- Light check-in if quiet for 8+ hours
```

**Cron作业**（精确计时）：

```bash
# Daily morning briefing at 7am
openclaw cron add --name "Morning brief" --cron "0 7 * * *" --session isolated --message "..." --announce

# Weekly project review on Mondays at 9am
openclaw cron add --name "Weekly review" --cron "0 9 * * 1" --session isolated --message "..." --model opus

# One-shot reminder
openclaw cron add --name "Call back" --at "2h" --session main --system-event "Call back the client" --wake now
```

## Lobster：具有审批功能的确定性工作流

Lobster是用于**多步骤工具管道**的工作流运行时，需要确定性执行和显式审批。当任务不仅仅是单次智能体轮次，并且您希望有一个可恢复的工作流和人工检查点时，请使用它。

### Lobster适用场景

-   **多步骤自动化**：您需要一个固定的工具调用管道，而不是一次性的提示。
-   **审批关卡**：副作用应暂停直到您批准，然后恢复。
-   **可恢复运行**：继续暂停的工作流，无需重新运行早期步骤。

### 如何与心跳和cron配合使用

-   **心跳/cron**决定*何时*运行。
-   **Lobster**定义运行开始后*执行哪些步骤*。

对于计划工作流，使用cron或心跳来触发调用Lobster的智能体轮次。对于临时工作流，直接调用Lobster。

### 操作说明（来自代码）

-   Lobster作为**本地子进程**（`lobster` CLI）在工具模式下运行，并返回一个**JSON信封**。
-   如果工具返回`needs_approval`，您可以使用`resumeToken`和`approve`标志恢复。
-   该工具是一个**可选插件**；建议通过`tools.alsoAllow: ["lobster"]`以附加方式启用。
-   Lobster期望`lobster` CLI在`PATH`上可用。

完整用法和示例请参见[Lobster](../tools/lobster.md)。

## 主会话与独立会话

心跳和cron都可以与主会话交互，但方式不同：

|  | 心跳 | Cron（主会话） | Cron（独立） |
| --- | --- | --- | --- |
| 会话 | 主会话 | 主会话（通过系统事件） | `cron:` |
| 历史记录 | 共享 | 共享 | 每次运行都是新的 |
| 上下文 | 完整 | 完整 | 无（从头开始） |
| 模型 | 主会话模型 | 主会话模型 | 可以覆盖 |
| 输出 | 如果不是`HEARTBEAT_OK`则交付 | 心跳提示 + 事件 | 公告摘要（默认） |

### 何时使用主会话cron

当您希望以下情况时，使用`--session main`和`--system-event`：

-   提醒/事件出现在主会话上下文中
-   智能体在下次心跳时处理它，并拥有完整上下文
-   没有单独的独立运行

```bash
openclaw cron add \
  --name "Check project" \
  --every "4h" \
  --session main \
  --system-event "Time for a project health check" \
  --wake now
```

### 何时使用独立cron

当您希望以下情况时，使用`--session isolated`：

-   没有先前上下文的干净状态
-   不同的模型或思考设置
-   直接将摘要公告到频道
-   历史记录不会干扰主会话

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

| 机制 | 成本概况 |
| --- | --- |
| 心跳 | 每N分钟一次轮次；随HEARTBEAT.md大小扩展 |
| Cron（主会话） | 将事件添加到下一次心跳（无独立轮次） |
| Cron（独立） | 每个作业完整的智能体轮次；可以使用更便宜的模型 |

**技巧**：

-   保持`HEARTBEAT.md`小巧以最小化令牌开销。
-   将类似检查批量处理到心跳中，而不是使用多个cron作业。
-   如果只希望内部处理，在心跳上使用`target: "none"`。
-   对于常规任务，使用带有更便宜模型的独立cron。

## 相关链接

-   [心跳](../gateway/heartbeat.md) - 完整心跳配置
-   [Cron作业](./cron-jobs.md) - 完整cron CLI和API参考
-   [系统](../cli/system.md) - 系统事件 + 心跳控制

[Cron作业](./cron-jobs.md)[自动化故障排除](./troubleshooting.md)