

  自动化

  
# Cron 任务

> **Cron 还是 Heartbeat？** 关于何时使用各自的指导，请参阅 [Cron vs Heartbeat](./cron-vs-heartbeat.md)。

Cron 是网关内置的调度器。它持久化任务，在正确的时间唤醒代理，并可以选择性地将输出发送回聊天。如果你想要 *“每天早上运行这个”* 或 *“20分钟后提醒代理”*，cron 就是实现机制。故障排除：[/automation/troubleshooting](./troubleshooting.md)

## 摘要

-   Cron 在**网关内部**运行（不在模型内部）。
-   任务持久化存储在 `~/.openclaw/cron/` 下，因此重启不会丢失计划。
-   两种执行风格：
    -   **主会话**：将系统事件加入队列，然后在下一个心跳时运行。
    -   **隔离**：在 `cron:` 中运行一个专用的代理轮次，可选择交付（默认宣布或无交付）。
-   唤醒是一等公民：任务可以请求“立即唤醒”或“下次心跳”。
-   每个任务可通过 `delivery.mode = "webhook"` + `delivery.to = ""` 进行 Webhook 投递。
-   当设置了 `cron.webhook` 时，对于存储的带有 `notify: true` 的遗留任务，仍保留回退机制，请将这些任务迁移到 webhook 交付模式。

## 快速开始（可操作）

创建一个一次性提醒，验证其存在，并立即运行：

```bash
openclaw cron add \
  --name "Reminder" \
  --at "2026-02-01T16:00:00Z" \
  --session main \
  --system-event "Reminder: check the cron docs draft" \
  --wake now \
  --delete-after-run

openclaw cron list
openclaw cron run <job-id>
openclaw cron runs --id <job-id>
```

安排一个带交付的重复隔离任务：

```bash
openclaw cron add \
  --name "Morning brief" \
  --cron "0 7 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize overnight updates." \
  --announce \
  --channel slack \
  --to "channel:C1234567890"
```

## 工具调用等效项（网关 cron 工具）

关于规范的 JSON 结构和示例，请参阅 [工具调用的 JSON 模式](./cron-jobs.md#json-schema-for-tool-calls)。

## Cron 任务存储位置

Cron 任务默认持久化存储在网关主机的 `~/.openclaw/cron/jobs.json`。网关将文件加载到内存中，并在更改时写回，因此手动编辑仅在网关停止时是安全的。建议使用 `openclaw cron add/edit` 或 cron 工具调用 API 进行更改。

## 面向初学者的概述

将 cron 任务视为：**何时**运行 + **做什么**。

1.  **选择计划**
    -   一次性提醒 → `schedule.kind = "at"` (CLI: `--at`)
    -   重复任务 → `schedule.kind = "every"` 或 `schedule.kind = "cron"`
    -   如果你的 ISO 时间戳省略了时区，则被视为 **UTC**。
2.  **选择运行位置**
    -   `sessionTarget: "main"` → 在下一个心跳时运行，使用主上下文。
    -   `sessionTarget: "isolated"` → 在 `cron:` 中运行一个专用的代理轮次。
3.  **选择有效载荷**
    -   主会话 → `payload.kind = "systemEvent"`
    -   隔离会话 → `payload.kind = "agentTurn"`

可选：一次性任务 (`schedule.kind = "at"`) 默认在成功后删除。设置 `deleteAfterRun: false` 以保留它们（它们将在成功后禁用）。

## 概念

### 任务

一个 cron 任务是一个存储的记录，包含：

-   一个**计划**（何时应运行），
-   一个**有效载荷**（应做什么），
-   可选的**交付模式** (`announce`、`webhook` 或 `none`)。
-   可选的**代理绑定** (`agentId`)：在特定代理下运行任务；如果缺失或未知，网关将回退到默认代理。

任务由稳定的 `jobId` 标识（由 CLI/网关 API 使用）。在代理工具调用中，`jobId` 是规范的；为兼容性也接受遗留的 `id`。一次性任务默认在成功后自动删除；设置 `deleteAfterRun: false` 以保留它们。

### 计划

Cron 支持三种计划类型：

-   `at`：通过 `schedule.at` 指定一次性时间戳（ISO 8601）。
-   `every`：固定间隔（毫秒）。
-   `cron`：5 字段 cron 表达式（或带秒的 6 字段），带有可选的 IANA 时区。

Cron 表达式使用 `croner`。如果省略时区，则使用网关主机的本地时区。为了减少跨多个网关的整点负载峰值，OpenClaw 对重复的整点表达式（例如 `0 * * * *`、`0 */2 * * *`）应用了确定性的每任务交错窗口，最多 5 分钟。固定小时表达式如 `0 7 * * *` 保持精确。对于任何 cron 计划，你可以使用 `schedule.staggerMs` 设置显式的交错窗口（`0` 保持精确计时）。CLI 快捷方式：

-   `--stagger 30s`（或 `1m`、`5m`）以设置显式交错窗口。
-   `--exact` 以强制 `staggerMs = 0`。

### 主会话与隔离执行

#### 主会话任务（系统事件）

主任务将系统事件加入队列，并可选择唤醒心跳运行器。它们必须使用 `payload.kind = "systemEvent"`。

-   `wakeMode: "now"`（默认）：事件触发立即的心跳运行。
-   `wakeMode: "next-heartbeat"`：事件等待下一个计划的心跳。

当你想要正常的心跳提示 + 主会话上下文时，这是最佳选择。参见 [Heartbeat](../gateway/heartbeat.md)。

#### 隔离任务（专用的 cron 会话）

隔离任务在会话 `cron:` 中运行一个专用的代理轮次。关键行为：

-   提示前缀为 `[cron: ]` 以便追踪。
-   每次运行启动一个**新的会话 ID**（无先前对话延续）。
-   默认行为：如果省略 `delivery`，隔离任务会宣布摘要 (`delivery.mode = "announce"`)。
-   `delivery.mode` 选择发生的情况：
    -   `announce`：将摘要发送到目标频道，并向主会话发布简短摘要。
    -   `webhook`：当完成事件包含摘要时，将完成的事件有效载荷 POST 到 `delivery.to`。
    -   `none`：仅内部（无交付，无主会话摘要）。
-   `wakeMode` 控制主会话摘要何时发布：
    -   `now`：立即心跳。
    -   `next-heartbeat`：等待下一个计划的心跳。

将隔离任务用于嘈杂、频繁或“后台杂务”，这些任务不应污染你的主聊天历史记录。

### 有效载荷结构（运行内容）

支持两种有效载荷类型：

-   `systemEvent`：仅主会话，通过心跳提示路由。
-   `agentTurn`：仅隔离会话，运行一个专用的代理轮次。

常见的 `agentTurn` 字段：

-   `message`：必需的文本提示。
-   `model` / `thinking`：可选覆盖（见下文）。
-   `timeoutSeconds`：可选的超时覆盖。
-   `lightContext`：可选轻量级引导模式，适用于不需要工作区引导文件注入的任务。

交付配置：

-   `delivery.mode`: `none` | `announce` | `webhook`。
-   `delivery.channel`: `last` 或特定频道。
-   `delivery.to`: 频道特定目标（宣布）或 webhook URL（webhook 模式）。
-   `delivery.bestEffort`: 如果宣布交付失败，避免任务失败。

宣布交付会抑制运行期间的消息工具发送；使用 `delivery.channel`/`delivery.to` 来定位聊天。当 `delivery.mode = "none"` 时，不会向主会话发布摘要。如果为隔离任务省略了 `delivery`，OpenClaw 默认为 `announce`。

#### 宣布交付流程

当 `delivery.mode = "announce"` 时，cron 直接通过出站频道适配器交付。主代理不会被启动来创建或转发消息。行为细节：

-   内容：交付使用隔离运行的出站有效载荷（文本/媒体），具有正常的块处理和频道格式化。
-   仅心跳响应（`HEARTBEAT_OK` 无实际内容）不会被交付。
-   如果隔离运行已通过消息工具向同一目标发送了消息，则跳过交付以避免重复。
-   缺失或无效的交付目标会导致任务失败，除非 `delivery.bestEffort = true`。
-   仅当 `delivery.mode = "announce"` 时，才会向主会话发布简短摘要。
-   主会话摘要遵循 `wakeMode`：`now` 触发立即心跳，`next-heartbeat` 等待下一个计划的心跳。

#### Webhook 交付流程

当 `delivery.mode = "webhook"` 时，cron 在完成事件包含摘要时将完成的事件有效载荷 POST 到 `delivery.to`。行为细节：

-   端点必须是有效的 HTTP(S) URL。
-   在 webhook 模式下不会尝试频道交付。
-   在 webhook 模式下不会向主会话发布摘要。
-   如果设置了 `cron.webhookToken`，认证头为 `Authorization: Bearer <cron.webhookToken>`。
-   已弃用的回退：存储的遗留任务如果设置了 `notify: true`，仍会发布到 `cron.webhook`（如果已配置），并带有警告，以便你可以迁移到 `delivery.mode = "webhook"`。

### 模型和思考级别覆盖

隔离任务 (`agentTurn`) 可以覆盖模型和思考级别：

-   `model`：提供商/模型字符串（例如，`anthropic/claude-sonnet-4-20250514`）或别名（例如，`opus`）
-   `thinking`：思考级别（`off`、`minimal`、`low`、`medium`、`high`、`xhigh`；仅限 GPT-5.2 + Codex 模型）

注意：你也可以在主会话任务上设置 `model`，但这会改变共享的主会话模型。我们建议仅对隔离任务使用模型覆盖，以避免意外的上下文切换。解析优先级：

1.  任务有效载荷覆盖（最高）
2.  钩子特定默认值（例如，`hooks.gmail.model`）
3.  代理配置默认值

### 轻量级引导上下文

隔离任务 (`agentTurn`) 可以设置 `lightContext: true` 以使用轻量级引导上下文运行。

-   将此用于不需要工作区引导文件注入的计划杂务。
-   实际上，嵌入式运行时以 `bootstrapContextMode: "lightweight"` 运行，这特意保持 cron 引导上下文为空。
-   CLI 等效项：`openclaw cron add --light-context ...` 和 `openclaw cron edit --light-context`。

### 交付（频道 + 目标）

隔离任务可以通过顶层 `delivery` 配置将输出交付到频道：

-   `delivery.mode`: `announce`（频道交付）、`webhook`（HTTP POST）或 `none`。
-   `delivery.channel`: `whatsapp` / `telegram` / `discord` / `slack` / `mattermost`（插件）/ `signal` / `imessage` / `last`。
-   `delivery.to`: 频道特定的接收者目标。

`announce` 交付仅对隔离任务 (`sessionTarget: "isolated"`) 有效。`webhook` 交付对主任务和隔离任务都有效。如果省略 `delivery.channel` 或 `delivery.to`，cron 可以回退到主会话的“最后路由”（代理最后回复的地方）。目标格式提醒：

-   Slack/Discord/Mattermost（插件）目标应使用显式前缀（例如 `channel:`、`user:`）以避免歧义。
-   Telegram 主题应使用 `:topic:` 形式（见下文）。

#### Telegram 交付目标（主题 / 论坛线程）

Telegram 通过 `message_thread_id` 支持论坛主题。对于 cron 交付，你可以将主题/线程编码到 `to` 字段中：

-   `-1001234567890`（仅聊天 ID）
-   `-1001234567890:topic:123`（推荐：显式主题标记）
-   `-1001234567890:123`（简写：数字后缀）

也接受带前缀的目标，如 `telegram:...` / `telegram:group:...`：

-   `telegram:group:-1001234567890:topic:123`

## 工具调用的 JSON 模式

直接调用网关 `cron.*` 工具时（代理工具调用或 RPC）使用这些结构。CLI 标志接受人类可读的持续时间如 `20m`，但工具调用应使用 ISO 8601 字符串表示 `schedule.at`，使用毫秒表示 `schedule.everyMs`。

### cron.add 参数

一次性，主会话任务（系统事件）：

```json
{
  "name": "Reminder",
  "schedule": { "kind": "at", "at": "2026-02-01T16:00:00Z" },
  "sessionTarget": "main",
  "wakeMode": "now",
  "payload": { "kind": "systemEvent", "text": "Reminder text" },
  "deleteAfterRun": true
}
```

重复，带交付的隔离任务：

```json
{
  "name": "Morning brief",
  "schedule": { "kind": "cron", "expr": "0 7 * * *", "tz": "America/Los_Angeles" },
  "sessionTarget": "isolated",
  "wakeMode": "next-heartbeat",
  "payload": {
    "kind": "agentTurn",
    "message": "Summarize overnight updates.",
    "lightContext": true
  },
  "delivery": {
    "mode": "announce",
    "channel": "slack",
    "to": "channel:C1234567890",
    "bestEffort": true
  }
}
```

注意：

-   `schedule.kind`: `at` (`at`)、`every` (`everyMs`) 或 `cron` (`expr`，可选 `tz`)。
-   `schedule.at` 接受 ISO 8601（时区可选；省略时视为 UTC）。
-   `everyMs` 是毫秒。
-   `sessionTarget` 必须是 `"main"` 或 `"isolated"`，并且必须与 `payload.kind` 匹配。
-   可选字段：`agentId`、`description`、`enabled`、`deleteAfterRun`（默认为 `at` 类型为 true）、`delivery`。
-   省略时 `wakeMode` 默认为 `"now"`。

### cron.update 参数

```json
{
  "jobId": "job-123",
  "patch": {
    "enabled": false,
    "schedule": { "kind": "every", "everyMs": 3600000 }
  }
}
```

注意：

-   `jobId` 是规范的；为兼容性也接受 `id`。
-   在补丁中使用 `agentId: null` 来清除代理绑定。

### cron.run 和 cron.remove 参数

```json
{ "jobId": "job-123", "mode": "force" }
```

```json
{ "jobId": "job-123" }
```

## 存储与历史

-   任务存储：`~/.openclaw/cron/jobs.json`（网关管理的 JSON）。
-   运行历史：`~/.openclaw/cron/runs/.jsonl`（JSONL，按大小和行数自动修剪）。
-   `sessions.json` 中的隔离 cron 运行会话由 `cron.sessionRetention` 修剪（默认 `24h`；设置为 `false` 以禁用）。
-   覆盖存储路径：配置中的 `cron.store`。

## 重试策略

当任务失败时，OpenClaw 将错误分类为**瞬态**（可重试）或**永久**（立即禁用）。

### 瞬态错误（重试）

-   速率限制（429，请求过多，资源耗尽）
-   提供商过载（例如 Anthropic `529 overloaded_error`，过载回退摘要）
-   网络错误（超时、ECONNRESET、获取失败、套接字）
-   服务器错误（5xx）
-   Cloudflare 相关错误

### 永久错误（不重试）

-   认证失败（无效的 API 密钥，未授权）
-   配置或验证错误
-   其他非瞬态错误

### 默认行为（无配置）

**一次性任务 (`schedule.kind: "at"`):**

-   瞬态错误：最多重试 3 次，使用指数退避（30s → 1m → 5m）。
-   永久错误：立即禁用。
-   成功或跳过：禁用（或如果 `deleteAfterRun: true` 则删除）。

**重复任务 (`cron` / `every`):**

-   任何错误：在下一次计划运行前应用指数退避（30s → 1m → 5m → 15m → 60m）。
-   任务保持启用；在下一次成功运行后重置退避。

配置 `cron.retry` 以覆盖这些默认值（参见 [配置](./cron-jobs.md#configuration)）。

## 配置

```json
{
  cron: {
    enabled: true, // 默认 true
    store: "~/.openclaw/cron/jobs.json",
    maxConcurrentRuns: 1, // 默认 1
    // 可选：覆盖一次性任务的重试策略
    retry: {
      maxAttempts: 3,
      backoffMs: [60000, 120000, 300000],
      retryOn: ["rate_limit", "overloaded", "network", "server_error"],
    },
    webhook: "https://example.invalid/legacy", // 已弃用，用于存储的 notify:true 任务的回退
    webhookToken: "replace-with-dedicated-webhook-token", // webhook 模式的可选承载令牌
    sessionRetention: "24h", // 持续时间字符串或 false
    runLog: {
      maxBytes: "2mb", // 默认 2_000_000 字节
      keepLines: 2000, // 默认 2000
    },
  },
}
```

运行日志修剪行为：

-   `cron.runLog.maxBytes`：修剪前的最大运行日志文件大小。
-   `cron.runLog.keepLines`：修剪时，仅保留最新的 N 行。
-   两者都适用于 `cron/runs/.jsonl` 文件。

Webhook 行为：

-   推荐：为每个任务设置 `delivery.mode: "webhook"` 和 `delivery.to: "https://..."`。
-   Webhook URL 必须是有效的 `http://` 或 `https://` URL。
-   发布时，有效载荷是 cron 完成事件 JSON。
-   如果设置了 `cron.webhookToken`，认证头为 `Authorization: Bearer <cron.webhookToken>`。
-   如果未设置 `cron.webhookToken`，则不发送 `Authorization` 头。
-   已弃用的回退：存储的遗留任务如果设置了 `notify: true`，在 `cron.webhook` 存在时仍会使用它。

完全禁用 cron：

-   `cron.enabled: false`（配置）
-   `OPENCLAW_SKIP_CRON=1`（环境变量）

## 维护

Cron 有两个内置的维护路径：隔离运行会话保留和运行日志修剪。

### 默认值

-   `cron.sessionRetention`: `24h`（设置为 `false` 以禁用运行会话修剪）
-   `cron.runLog.maxBytes`: `2_000_000` 字节
-   `cron.runLog.keepLines`: `2000`

### 工作原理

-   隔离运行创建会话条目 (`...:cron::run:`) 和转录文件。
-   清理器会删除超过 `cron.sessionRetention` 期限的过期运行会话条目。
-   对于会话存储中不再引用的已移除运行会话，OpenClaw 会归档转录文件，并在相同的保留窗口内清除旧的已删除归档。
-   每次运行追加后，会检查 `cron/runs/.jsonl` 的大小：
    -   如果文件大小超过 `runLog.maxBytes`，则将其修剪为最新的 `runLog.keepLines` 行。

### 高频率调度器的性能注意事项

高频率的 cron 设置可能会产生大量的运行会话和运行日志足迹。维护是内置的，但宽松的限制仍可能产生可避免的 IO 和清理工作。需要注意：

-   许多隔离运行配合较长的 `cron.sessionRetention` 窗口
-   较高的 `cron.runLog.keepLines` 配合较大的 `runLog.maxBytes`
-   许多嘈杂的重复任务写入同一个 `cron/runs/.jsonl`

建议：

-   将 `cron.sessionRetention` 保持在调试/审计所需的最短时间
-   使用适中的 `runLog.maxBytes` 和 `runLog.keepLines` 限制运行日志
-   将嘈杂的后台任务移至隔离模式，并设置避免不必要闲聊的交付规则
-   定期使用 `openclaw cron runs` 检查增长情况，并在日志变大前调整保留策略

### 自定义示例

保留运行会话一周并允许更大的运行日志：

```json
{
  cron: {
    sessionRetention: "7d",
    runLog: {
      maxBytes: "10mb",
      keepLines: 5000,
    },
  },
}
```

禁用隔离运行会话修剪但保留运行日志修剪：

```json
{
  cron: {
    sessionRetention: false,
    runLog: {
      maxBytes: "5mb",
      keepLines: 3000,
    },
  },
}
```

针对高频率 cron 使用进行调整（示例）：

```json
{
  cron: {
    sessionRetention: "12h",
    runLog: {
      maxBytes: "3mb",
      keepLines: 1500,
    },
  },
}
```

## CLI 快速开始

一次性提醒（UTC ISO，成功后自动删除）：

```bash
openclaw cron add \
  --name "Send reminder" \
  --at "2026-01-12T18:00:00Z" \
  --session main \
  --system-event "Reminder: submit expense report." \
  --wake now \
  --delete-after-run
```

一次性提醒（主会话，立即唤醒）：

```bash
openclaw cron add \
  --name "Calendar check" \
  --at "20m" \
  --session main \
  --system-event "Next heartbeat: check calendar." \
  --wake now
```

重复隔离任务（宣布到 WhatsApp）：

```bash
openclaw cron add \
  --name "Morning status" \
  --cron "0 7 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize inbox + calendar for today." \
  --announce \
  --channel whatsapp \
  --to "+15551234567"
```

具有显式 30 秒交错的重复 cron 任务：

```bash
openclaw cron add \
  --name "Minute watcher" \
  --cron "0 * * * * *" \
  --tz "UTC" \
  --stagger 30s \
  --session isolated \
  --message "Run minute watcher checks." \
  --announce
```

重复隔离任务（交付到 Telegram 主题）：

```bash
openclaw cron add \
  --name "Nightly summary (topic)" \
  --cron "0 22 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize today; send to the nightly topic." \
  --announce \
  --channel telegram \
  --to "-1001234567890:topic:123"
```

具有模型和思考级别覆盖的隔离任务：

```bash
openclaw cron add \
  --name "Deep analysis" \
  --cron "0 6 * * 1" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Weekly deep analysis of project progress." \
  --model "opus" \
  --thinking high \
  --announce \
  --channel whatsapp \
  --to "+15551234567"
```

代理选择（多代理设置）：

```bash
# 将任务固定到代理 "ops"（如果该代理缺失，则回退到默认代理）
openclaw cron add --name "Ops sweep" --cron "0 6 * * *" --session isolated --message "Check ops queue" --agent ops

# 切换或清除现有任务的代理
openclaw cron edit <jobId> --agent ops
openclaw cron edit <jobId> --clear-agent
```

手动运行（force 是默认值，使用 `--due` 仅当到期时运行）：

```bash
openclaw cron run <jobId>
openclaw cron run <jobId> --due
```

编辑现有任务（修补字段）：

```bash
openclaw cron edit <jobId> \
  --message "Updated prompt" \
  --model "opus" \
  --thinking low
```

强制现有 cron 任务完全按计划运行（无交错）：

```bash
openclaw cron edit <jobId> --exact
```

运行历史：

```bash
openclaw cron runs --id <jobId> --limit 50
```

立即系统事件（不创建任务）：

```bash
openclaw system event --mode now --text "Next heartbeat: check battery."
```

## 网关 API 接口

-   `cron.list`、`cron.status`、`cron.add`、`cron.update`、`cron.remove`
-   `cron.run`（强制或到期）、`cron.runs` 对于无需任务的立即系统事件，请使用 [`openclaw system event`](../cli/system.md)。

## 故障排除

### “什么都不运行”

-   检查 cron 是否启用：`cron.enabled` 和 `OPENCLAW_SKIP_CRON`。
-   检查网关是否持续运行（cron 在网关进程内部运行）。
-   对于 `cron` 计划：确认时区 (`--tz`) 与主机时区。

### 重复任务在失败后持续延迟

-   OpenClaw 在连续错误后对重复任务应用指数重试退避：重试之间间隔 30s、1m、5m、15m，然后 60m。
-   退避在下一次成功运行后自动重置。
-   一次性 (`at`) 任务对瞬态错误（速率限制、过载、网络、server\_error）最多重试 3 次并退避；永久错误立即禁用。参见 [重试策略](./cron-jobs.md#retry-policy)。

### Telegram 交付到错误的地方

-   对于论坛主题，使用 `-100…:topic:` 使其明确且无歧义。
-   如果你在日志或存储的“最后路由”目标中看到 `telegram:...` 前缀，这是正常的；cron 交付接受它们，并且仍然正确解析主题 ID。

### 子代理宣布交付重试

-   当子代理运行完成时，网关向请求者会话宣布结果。
-   如果宣布流程返回 `false`（例如请求者会话繁忙），网关最多重试 3 次，并通过 `announceRetryCount` 跟踪。
-   超过 `endedAt` 5 分钟的宣布将被强制过期，以防止陈旧条目无限循环。
-   如果你在日志中看到重复的宣布交付，请检查子代理注册表中 `announceRetryCount` 值较高的条目。

[Hooks](./hooks.md)[Cron vs Heartbeat](./cron-vs-heartbeat.md)