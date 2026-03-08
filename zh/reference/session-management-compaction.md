

  压缩内部机制

  
# 会话管理深度解析

本文档解释了 OpenClaw 如何端到端管理会话：

-   **会话路由**（入站消息如何映射到 `sessionKey`）
-   **会话存储**（`sessions.json`）及其跟踪内容
-   **转录本持久化**（`*.jsonl`）及其结构
-   **转录本清理**（运行前针对特定提供商的修复）
-   **上下文限制**（上下文窗口与跟踪的令牌数）
-   **压缩**（手动 + 自动压缩）以及在哪里挂钩压缩前工作
-   **静默后台维护**（例如，不应产生用户可见输出的内存写入）

如果您想先了解更高层次的概述，请从以下开始：

-   [/concepts/session](../concepts/session.md)
-   [/concepts/compaction](../concepts/compaction.md)
-   [/concepts/session-pruning](../concepts/session-pruning.md)
-   [/reference/transcript-hygiene](./transcript-hygiene.md)

* * *

## 事实来源：网关

OpenClaw 围绕一个拥有会话状态的单一**网关进程**设计。

-   UI（macOS 应用、网页控制 UI、TUI）应向网关查询会话列表和令牌计数。
-   在远程模式下，会话文件位于远程主机上；“检查本地 Mac 文件”不会反映网关正在使用的内容。

* * *

## 两个持久化层

OpenClaw 在两个层中持久化会话：

1.  **会话存储 (`sessions.json`)**
    -   键/值映射：`sessionKey -> SessionEntry`
    -   小型、可变、可安全编辑（或删除条目）
    -   跟踪会话元数据（当前会话 ID、最后活动时间、开关、令牌计数器等）
2.  **转录本 (`.jsonl`)**
    -   具有树状结构的仅追加转录本（条目包含 `id` + `parentId`）
    -   存储实际的对话 + 工具调用 + 压缩摘要
    -   用于为未来的轮次重建模型上下文

* * *

## 磁盘位置

每个代理，在网关主机上：

-   存储：`~/.openclaw/agents//sessions/sessions.json`
-   转录本：`~/.openclaw/agents//sessions/.jsonl`
    -   Telegram 话题会话：`.../-topic-.jsonl`

OpenClaw 通过 `src/config/sessions.ts` 解析这些路径。

* * *

## 存储维护和磁盘控制

会话持久化具有针对 `sessions.json` 和转录本产物的自动维护控制 (`session.maintenance`)：

-   `mode`：`warn`（默认）或 `enforce`
-   `pruneAfter`：陈旧条目的年龄截止点（默认 `30d`）
-   `maxEntries`：`sessions.json` 中的条目上限（默认 `500`）
-   `rotateBytes`：当 `sessions.json` 过大时轮换（默认 `10mb`）
-   `resetArchiveRetention`：`*.reset.` 转录本归档的保留时间（默认：与 `pruneAfter` 相同；`false` 禁用清理）
-   `maxDiskBytes`：可选的会话目录预算
-   `highWaterBytes`：清理后的可选目标（默认为 `maxDiskBytes` 的 `80%`）

磁盘预算清理的执行顺序 (`mode: "enforce"`)：

1.  首先删除最旧的归档或孤立的转录本产物。
2.  如果仍然高于目标，则逐出最旧的会话条目及其转录本文件。
3.  持续进行，直到使用量达到或低于 `highWaterBytes`。

在 `mode: "warn"` 下，OpenClaw 报告潜在的逐出操作但不修改存储/文件。按需运行维护：

```bash
openclaw sessions cleanup --dry-run
openclaw sessions cleanup --enforce
```

* * *

## 定时任务会话和运行日志

独立的定时任务运行也会创建会话条目/转录本，并且它们有专门的保留控制：

-   `cron.sessionRetention`（默认 `24h`）从会话存储中清理旧的独立定时任务运行会话（`false` 禁用）。
-   `cron.runLog.maxBytes` + `cron.runLog.keepLines` 清理 `~/.openclaw/cron/runs/.jsonl` 文件（默认值：`2_000_000` 字节和 `2000` 行）。

* * *

## 会话键 (sessionKey)

`sessionKey` 标识*您所在的对话桶*（路由 + 隔离）。常见模式：

-   主/直接聊天（每个代理）：`agent::`（默认 `main`）
-   群组：`agent:::group:`
-   房间/频道（Discord/Slack）：`agent:::channel:` 或 `...:room:`
-   定时任务：`cron:<job.id>`
-   Webhook：`hook:`（除非被覆盖）

规范规则记录在 [/concepts/session](../concepts/session.md)。

* * *

## 会话 ID (sessionId)

每个 `sessionKey` 指向一个当前的 `sessionId`（继续对话的转录本文件）。经验法则：

-   **重置**（`/new`、`/reset`）为该 `sessionKey` 创建一个新的 `sessionId`。
-   **每日重置**（默认在网关主机本地时间凌晨 4:00）在重置边界后的下一条消息处创建一个新的 `sessionId`。
-   **空闲过期**（`session.reset.idleMinutes` 或旧版 `session.idleMinutes`）在消息在空闲窗口后到达时创建一个新的 `sessionId`。当每日和空闲都配置时，哪个先过期哪个生效。
-   **线程父分支防护**（`session.parentForkMaxTokens`，默认 `100000`）在父会话已经太大时跳过父转录本分支；新线程从头开始。设置为 `0` 以禁用。

实现细节：该决策发生在 `src/auto-reply/reply/session.ts` 中的 `initSessionState()`。

* * *

## 会话存储模式 (sessions.json)

存储的值类型是 `src/config/sessions.ts` 中的 `SessionEntry`。关键字段（非详尽）：

-   `sessionId`：当前转录本 ID（除非设置了 `sessionFile`，否则文件名由此派生）
-   `updatedAt`：最后活动时间戳
-   `sessionFile`：可选显式转录本路径覆盖
-   `chatType`：`direct | group | room`（有助于 UI 和发送策略）
-   `provider`、`subject`、`room`、`space`、`displayName`：用于群组/频道标签的元数据
-   开关：
    -   `thinkingLevel`、`verboseLevel`、`reasoningLevel`、`elevatedLevel`
    -   `sendPolicy`（每个会话的覆盖）
-   模型选择：
    -   `providerOverride`、`modelOverride`、`authProfileOverride`
-   令牌计数器（尽力而为 / 依赖于提供商）：
    -   `inputTokens`、`outputTokens`、`totalTokens`、`contextTokens`
-   `compactionCount`：此会话键自动压缩完成的频率
-   `memoryFlushAt`：上次压缩前内存刷新的时间戳
-   `memoryFlushCompactionCount`：上次刷新运行时的压缩计数

存储可以安全编辑，但网关是权威：它可能会在会话运行时重写或重新水合条目。

* * *

## 转录本结构 (\*.jsonl)

转录本由 `@mariozechner/pi-coding-agent` 的 `SessionManager` 管理。文件是 JSONL：

-   第一行：会话头（`type: "session"`，包括 `id`、`cwd`、`timestamp`，可选的 `parentSession`）
-   然后：带有 `id` + `parentId` 的会话条目（树状）

值得注意的条目类型：

-   `message`：用户/助手/toolResult 消息
-   `custom_message`：扩展注入的、*会*进入模型上下文的消息（可以从 UI 隐藏）
-   `custom`：*不*进入模型上下文的扩展状态
-   `compaction`：持久化的压缩摘要，包含 `firstKeptEntryId` 和 `tokensBefore`
-   `branch_summary`：导航树分支时持久化的摘要

OpenClaw 有意*不*“修复”转录本；网关使用 `SessionManager` 来读取/写入它们。

* * *

## 上下文窗口与跟踪的令牌数

两个不同的概念很重要：

1.  **模型上下文窗口**：每个模型的硬性上限（模型可见的令牌）
2.  **会话存储计数器**：写入 `sessions.json` 的滚动统计信息（用于 /status 和仪表板）

如果您正在调整限制：

-   上下文窗口来自模型目录（并且可以通过配置覆盖）。
-   存储中的 `contextTokens` 是运行时估计/报告值；不要将其视为严格保证。

更多信息，请参阅 [/token-use](./token-use.md)。

* * *

## 压缩：它是什么

压缩将较早的对话总结为转录本中持久化的 `compaction` 条目，并保持最近的消息完整。压缩后，未来的轮次会看到：

-   压缩摘要
-   `firstKeptEntryId` 之后的消息

压缩是**持久性的**（与会话修剪不同）。请参阅 [/concepts/session-pruning](../concepts/session-pruning.md)。

* * *

## 自动压缩何时发生（Pi 运行时）

在嵌入式 Pi 代理中，自动压缩在两种情况下触发：

1.  **溢出恢复**：模型返回上下文溢出错误 → 压缩 → 重试。
2.  **阈值维护**：在成功轮次之后，当：

`contextTokens > contextWindow - reserveTokens` 其中：

-   `contextWindow` 是模型的上下文窗口
-   `reserveTokens` 是为提示词 + 下一个模型输出保留的余量

这些是 Pi 运行时语义（OpenClaw 消费事件，但 Pi 决定何时压缩）。

* * *

## 压缩设置 (reserveTokens, keepRecentTokens)

Pi 的压缩设置位于 Pi 设置中：

```json
{
  compaction: {
    enabled: true,
    reserveTokens: 16384,
    keepRecentTokens: 20000,
  },
}
```

OpenClaw 还为嵌入式运行强制执行安全下限：

-   如果 `compaction.reserveTokens < reserveTokensFloor`，OpenClaw 会提升它。
-   默认下限是 `20000` 个令牌。
-   设置 `agents.defaults.compaction.reserveTokensFloor: 0` 以禁用下限。
-   如果已经更高，OpenClaw 则保持不变。

原因：为多轮“后台维护”（如内存写入）留出足够的余量，然后压缩才变得不可避免。实现：`src/agents/pi-settings.ts` 中的 `ensurePiCompactionReserveTokens()`（从 `src/agents/pi-embedded-runner.ts` 调用）。

* * *

## 用户可见界面

您可以通过以下方式观察压缩和会话状态：

-   `/status`（在任何聊天会话中）
-   `openclaw status`（CLI）
-   `openclaw sessions` / `sessions --json`
-   详细模式：`🧹 Auto-compaction complete` + 压缩计数

* * *

## 静默后台维护 (NO_REPLY)

OpenClaw 支持用于后台任务的“静默”轮次，用户不应看到中间输出。约定：

-   助手在其输出开头使用 `NO_REPLY` 来表示“不要向用户传递回复”。
-   OpenClaw 在传递层剥离/抑制此内容。

截至 `2026.1.10`，OpenClaw 还在部分区块以 `NO_REPLY` 开头时抑制**草稿/打字流**，因此静默操作不会在轮次中泄露部分输出。

* * *

## 压缩前“内存刷新”（已实现）

目标：在自动压缩发生之前，运行一个静默的代理轮次，将持久状态写入磁盘（例如，代理工作区中的 `memory/YYYY-MM-DD.md`），这样压缩就无法擦除关键上下文。OpenClaw 使用**阈值前刷新**方法：

1.  监控会话上下文使用情况。
2.  当它超过“软阈值”（低于 Pi 的压缩阈值）时，向代理运行一个静默的“立即写入内存”指令。
3.  使用 `NO_REPLY` 以便用户看不到任何内容。

配置 (`agents.defaults.compaction.memoryFlush`)：

-   `enabled`（默认：`true`）
-   `softThresholdTokens`（默认：`4000`）
-   `prompt`（刷新轮次的用户消息）
-   `systemPrompt`（为刷新轮次附加的额外系统提示词）

注意：

-   默认提示词/系统提示词包含 `NO_REPLY` 提示以抑制传递。
-   每个压缩周期运行一次刷新（在 `sessions.json` 中跟踪）。
-   仅针对嵌入式 Pi 会话运行刷新（CLI 后端跳过它）。
-   当会话工作区为只读时跳过刷新（`workspaceAccess: "ro"` 或 `"none"`）。
-   有关工作区文件布局和写入模式，请参阅[内存](../concepts/memory.md)。

Pi 还在扩展 API 中公开了 `session_before_compact` 钩子，但 OpenClaw 的刷新逻辑目前位于网关端。

* * *

## 故障排除清单

-   会话键错误？从 [/concepts/session](../concepts/session.md) 开始，并确认 `/status` 中的 `sessionKey`。
-   存储与转录本不匹配？确认网关主机和 `openclaw status` 中的存储路径。
-   压缩频繁发生？检查：
    -   模型上下文窗口（太小）
    -   压缩设置（`reserveTokens` 相对于模型窗口过高可能导致较早压缩）
    -   工具结果膨胀：启用/调整会话修剪
-   静默轮次泄露？确认回复以 `NO_REPLY`（确切令牌）开头，并且您使用的版本包含流抑制修复。

[Node.js](../install/node.md)[设置](../start/setup.md)