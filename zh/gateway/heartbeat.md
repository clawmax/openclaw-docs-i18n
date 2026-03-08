

  配置与操作

  
# Heartbeat

> **Heartbeat 与 Cron 的区别？** 关于何时使用各自的指导，请参阅 [Cron vs Heartbeat](../automation/cron-vs-heartbeat.md)。

Heartbeat 在主会话中运行**周期性代理轮询**，以便模型能够发现任何需要注意的事项，而不会对您造成信息轰炸。故障排除：[/automation/troubleshooting](../automation/troubleshooting.md)

## 快速开始（新手）

1.  保持心跳启用（默认 `30m`，或 Anthropic OAuth/setup-token 为 `1h`）或设置您自己的节奏。
2.  在代理工作区创建一个简短的 `HEARTBEAT.md` 检查清单（可选但推荐）。
3.  决定心跳消息应发送到哪里（默认 `target: "none"`；设置 `target: "last"` 可路由到最后联系人）。
4.  可选：启用心跳推理交付以提高透明度。
5.  可选：如果心跳运行仅需要 `HEARTBEAT.md`，则使用轻量级引导上下文。
6.  可选：将心跳限制在活跃时段（本地时间）。

配置示例：

```json
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last", // 显式发送到最后联系人（默认为 "none"）
        directPolicy: "allow", // 默认：允许直接/DM 目标；设置为 "block" 以抑制
        lightContext: true, // 可选：仅从引导文件注入 HEARTBEAT.md
        // activeHours: { start: "08:00", end: "24:00" },
        // includeReasoning: true, // 可选：同时发送单独的 `Reasoning:` 消息
      },
    },
  },
}
```

## 默认值

-   间隔：`30m`（或当检测到 Anthropic OAuth/setup-token 作为认证模式时为 `1h`）。设置 `agents.defaults.heartbeat.every` 或每个代理的 `agents.list[].heartbeat.every`；使用 `0m` 禁用。
-   提示正文（可通过 `agents.defaults.heartbeat.prompt` 配置）：`Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`
-   心跳提示**逐字**作为用户消息发送。系统提示包含一个“Heartbeat”部分，并且运行在内部被标记。
-   活跃时段 (`heartbeat.activeHours`) 在配置的时区中检查。在窗口之外，心跳将被跳过，直到下一个在窗口内的触发点。

## 心跳提示的用途

默认提示有意设计得宽泛：

-   **后台任务**：“考虑未完成的任务”促使代理审查待办事项（收件箱、日历、提醒、排队工作）并发现任何紧急事项。
-   **人工签到**：“在白天有时检查一下您的人类”促使偶尔发送轻量级的“您需要什么吗？”消息，但通过使用您配置的本地时区来避免夜间骚扰（参见 [/concepts/timezone](../concepts/timezone.md)）。

如果您希望心跳执行非常具体的操作（例如“检查 Gmail PubSub 统计信息”或“验证网关健康状态”），请将 `agents.defaults.heartbeat.prompt`（或 `agents.list[].heartbeat.prompt`）设置为自定义正文（逐字发送）。

## 响应约定

-   如果无需关注任何事项，请回复 **`HEARTBEAT_OK`**。
-   在心跳运行期间，当 `HEARTBEAT_OK` 出现在回复的**开头或结尾**时，OpenClaw 将其视为确认。该令牌会被剥离，并且如果剩余内容**≤ `ackMaxChars`**（默认值：300），则回复将被丢弃。
-   如果 `HEARTBEAT_OK` 出现在回复的**中间**，则不会被特殊处理。
-   对于警报，**请勿**包含 `HEARTBEAT_OK`；仅返回警报文本。

在心跳之外，消息开头/结尾的游离 `HEARTBEAT_OK` 会被剥离并记录；仅包含 `HEARTBEAT_OK` 的消息将被丢弃。

## 配置

```json
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m", // 默认：30m（0m 禁用）
        model: "anthropic/claude-opus-4-6",
        includeReasoning: false, // 默认：false（在可用时交付单独的 Reasoning: 消息）
        lightContext: false, // 默认：false；true 仅保留工作区引导文件中的 HEARTBEAT.md
        target: "last", // 默认：none | 选项：last | none | <channel id>（核心或插件，例如 "bluebubbles"）
        to: "+15551234567", // 可选通道特定覆盖
        accountId: "ops-bot", // 可选多账户通道 id
        prompt: "Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.",
        ackMaxChars: 300, // HEARTBEAT_OK 后允许的最大字符数
      },
    },
  },
}
```

### 作用域和优先级

-   `agents.defaults.heartbeat` 设置全局心跳行为。
-   `agents.list[].heartbeat` 在其基础上合并；如果任何代理有 `heartbeat` 块，**则只有那些代理**运行心跳。
-   `channels.defaults.heartbeat` 为所有通道设置可见性默认值。
-   `channels..heartbeat` 覆盖通道默认值。
-   `channels..accounts..heartbeat`（多账户通道）覆盖每个通道的设置。

### 每个代理的心跳

如果任何 `agents.list[]` 条目包含 `heartbeat` 块，**则只有那些代理**运行心跳。每个代理的块在 `agents.defaults.heartbeat` 的基础上合并（因此您可以设置一次共享默认值，然后按代理覆盖）。示例：两个代理，只有第二个代理运行心跳。

```json
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last", // 显式发送到最后联系人（默认为 "none"）
      },
    },
    list: [
      { id: "main", default: true },
      {
        id: "ops",
        heartbeat: {
          every: "1h",
          target: "whatsapp",
          to: "+15551234567",
          prompt: "Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.",
        },
      },
    ],
  },
}
```

### 活跃时段示例

将心跳限制在特定时区的营业时间内：

```json
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last", // 显式发送到最后联系人（默认为 "none"）
        activeHours: {
          start: "09:00",
          end: "22:00",
          timezone: "America/New_York", // 可选；如果设置了 userTimezone 则使用，否则使用主机时区
        },
      },
    },
  },
}
```

在此窗口之外（东部时间上午 9 点之前或晚上 10 点之后），心跳将被跳过。窗口内下一个计划的触发点将正常运行。

### 24/7 设置

如果您希望心跳全天运行，请使用以下模式之一：

-   完全省略 `activeHours`（没有时间窗口限制；这是默认行为）。
-   设置全天窗口：`activeHours: { start: "00:00", end: "24:00" }`。

请勿将 `start` 和 `end` 时间设置为相同（例如 `08:00` 到 `08:00`）。这被视为零宽度窗口，因此心跳将始终被跳过。

### 多账户示例

使用 `accountId` 来定位多账户通道（如 Telegram）上的特定账户：

```json
{
  agents: {
    list: [
      {
        id: "ops",
        heartbeat: {
          every: "1h",
          target: "telegram",
          to: "12345678:topic:42", // 可选：路由到特定主题/线程
          accountId: "ops-bot",
        },
      },
    ],
  },
  channels: {
    telegram: {
      accounts: {
        "ops-bot": { botToken: "YOUR_TELEGRAM_BOT_TOKEN" },
      },
    },
  },
}
```

### 字段说明

-   `every`：心跳间隔（持续时间字符串；默认单位 = 分钟）。
-   `model`：心跳运行的可选模型覆盖（`provider/model`）。
-   `includeReasoning`：启用后，在可用时也交付单独的 `Reasoning:` 消息（与 `/reasoning on` 形状相同）。
-   `lightContext`：为 true 时，心跳运行使用轻量级引导上下文，并且仅保留工作区引导文件中的 `HEARTBEAT.md`。
-   `session`：心跳运行的可选会话键。
    -   `main`（默认）：代理主会话。
    -   显式会话键（从 `openclaw sessions --json` 或 [sessions CLI](../cli/sessions.md) 复制）。
    -   会话键格式：参见 [Sessions](../concepts/session.md) 和 [Groups](../channels/groups.md)。
-   `target`：
    -   `last`：发送到最后使用的外部通道。
    -   显式通道：`whatsapp` / `telegram` / `discord` / `googlechat` / `slack` / `msteams` / `signal` / `imessage`。
    -   `none`（默认）：运行心跳但**不**进行外部交付。
-   `directPolicy`：控制直接/DM 交付行为：
    -   `allow`（默认）：允许直接/DM 心跳交付。
    -   `block`：抑制直接/DM 交付（`reason=dm-blocked`）。
-   `to`：可选收件人覆盖（通道特定 id，例如 WhatsApp 的 E.164 或 Telegram 聊天 id）。对于 Telegram 主题/线程，使用 `:topic:`。
-   `accountId`：多账户通道的可选账户 id。当 `target: "last"` 时，如果解析出的最后通道支持账户，则账户 id 应用于该通道；否则将被忽略。如果账户 id 与解析出的通道的配置账户不匹配，则跳过交付。
-   `prompt`：覆盖默认提示正文（不合并）。
-   `ackMaxChars`：在交付之前，`HEARTBEAT_OK` 之后允许的最大字符数。
-   `suppressToolErrorWarnings`：为 true 时，抑制心跳运行期间的工具错误警告负载。
-   `activeHours`：将心跳运行限制在时间窗口内。包含 `start`（HH:MM，包含；使用 `00:00` 表示一天开始）、`end`（HH:MM 不包含；允许 `24:00` 表示一天结束）和可选 `timezone` 的对象。
    -   省略或 `"user"`：如果设置了 `agents.defaults.userTimezone` 则使用，否则回退到主机系统时区。
    -   `"local"`：始终使用主机系统时区。
    -   任何 IANA 标识符（例如 `America/New_York`）：直接使用；如果无效，则回退到上述 `"user"` 行为。
    -   `start` 和 `end` 不得相等以形成活动窗口；相等的值被视为零宽度（始终在窗口外）。
    -   在活动窗口之外，心跳将被跳过，直到窗口内的下一个触发点。

## 交付行为

-   心跳默认在代理的主会话中运行（`agent::`），或当 `session.scope = "global"` 时在 `global` 中运行。设置 `session` 以覆盖到特定通道会话（Discord/WhatsApp/等）。
-   `session` 仅影响运行上下文；交付由 `target` 和 `to` 控制。
-   要交付到特定通道/收件人，请设置 `target` + `to`。使用 `target: "last"` 时，交付使用该会话的最后外部通道。
-   心跳交付默认允许直接/DM 目标。设置 `directPolicy: "block"` 以抑制直接目标发送，同时仍运行心跳轮询。
-   如果主队列繁忙，心跳将被跳过并稍后重试。
-   如果 `target` 解析为没有外部目的地，运行仍会发生，但不会发送出站消息。
-   仅心跳回复**不会**保持会话活动；`updatedAt` 会恢复，因此空闲过期行为正常。

## 可见性控制

默认情况下，`HEARTBEAT_OK` 确认被抑制，而警报内容会被交付。您可以按通道或按账户调整此设置：

```
channels:
  defaults:
    heartbeat:
      showOk: false # 隐藏 HEARTBEAT_OK（默认）
      showAlerts: true # 显示警报消息（默认）
      useIndicator: true # 发出指示器事件（默认）
  telegram:
    heartbeat:
      showOk: true # 在 Telegram 上显示 OK 确认
  whatsapp:
    accounts:
      work:
        heartbeat:
          showAlerts: false # 为此账户抑制警报交付
```

优先级：每个账户 → 每个通道 → 通道默认值 → 内置默认值。

### 每个标志的作用

-   `showOk`：当模型返回仅 OK 的回复时，发送 `HEARTBEAT_OK` 确认。
-   `showAlerts`：当模型返回非 OK 回复时，发送警报内容。
-   `useIndicator`：为 UI 状态界面发出指示器事件。

如果**所有三个**都为 false，OpenClaw 将完全跳过心跳运行（不调用模型）。

### 每个通道与每个账户示例

```yaml
channels:
  defaults:
    heartbeat:
      showOk: false
      showAlerts: true
      useIndicator: true
  slack:
    heartbeat:
      showOk: true # 所有 Slack 账户
    accounts:
      ops:
        heartbeat:
          showAlerts: false # 仅抑制 ops 账户的警报
  telegram:
    heartbeat:
      showOk: true
```

### 常见模式

| 目标 | 配置 |
| --- | --- |
| 默认行为（静默 OK，警报开启） | *（无需配置）* |
| 完全静默（无消息，无指示器） | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: false }` |
| 仅指示器（无消息） | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: true }` |
| 仅在一个通道中显示 OK | `channels.telegram.heartbeat: { showOk: true }` |

## HEARTBEAT.md（可选）

如果工作区中存在 `HEARTBEAT.md` 文件，默认提示会告诉代理读取它。将其视为您的“心跳检查清单”：小巧、稳定且每 30 分钟包含一次是安全的。如果 `HEARTBEAT.md` 存在但实际上是空的（仅空行和 Markdown 标题如 `# Heading`），OpenClaw 将跳过心跳运行以节省 API 调用。如果文件缺失，心跳仍会运行，模型将决定做什么。保持其小巧（简短的检查清单或提醒）以避免提示膨胀。`HEARTBEAT.md` 示例：

```bash
# Heartbeat checklist

- Quick scan: anything urgent in inboxes?
- If it’s daytime, do a lightweight check-in if nothing else is pending.
- If a task is blocked, write down _what is missing_ and ask Peter next time.
```

### 代理可以更新 HEARTBEAT.md 吗？

可以——如果您要求它的话。`HEARTBEAT.md` 只是代理工作区中的一个普通文件，因此您可以（在正常聊天中）告诉代理类似以下内容：

-   “更新 `HEARTBEAT.md` 以添加每日日历检查。”
-   “重写 `HEARTBEAT.md`，使其更短并专注于收件箱待办事项。”

如果您希望这主动发生，也可以在您的心跳提示中包含一个明确的指令，例如：“如果检查清单变得过时，请用更好的清单更新 HEARTBEAT.md。”安全提示：请勿将机密信息（API 密钥、电话号码、私人令牌）放入 `HEARTBEAT.md` 中——它会成为提示上下文的一部分。

## 手动唤醒（按需）

您可以通过以下方式入队系统事件并触发即时心跳：

```bash
openclaw system event --text "Check for urgent follow-ups" --mode now
```

如果多个代理配置了 `heartbeat`，手动唤醒会立即运行每个代理的心跳。使用 `--mode next-heartbeat` 等待下一个计划的触发点。

## 推理交付（可选）

默认情况下，心跳仅交付最终的“答案”负载。如果您需要透明度，请启用：

-   `agents.defaults.heartbeat.includeReasoning: true`

启用后，心跳还将交付一个以 `Reasoning:` 为前缀的单独消息（与 `/reasoning on` 形状相同）。当代理管理多个会话/代码库并且您想了解它为何决定通知您时，这可能很有用——但它也可能泄露比您想要的更多的内部细节。在群聊中建议保持关闭。

## 成本意识

心跳运行完整的代理轮询。较短的间隔会消耗更多令牌。保持 `HEARTBEAT.md` 小巧，并考虑使用更便宜的 `model` 或 `target: "none"`，如果您只想要内部状态更新。

[健康检查](./health.md)[Doctor](./doctor.md)