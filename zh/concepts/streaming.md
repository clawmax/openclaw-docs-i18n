

  消息与传递

  
# 流式传输与分块

OpenClaw 有两个独立的流式传输层：

-   **块流式传输（频道）：** 在助手书写时发出已完成的**块**。这些是普通的频道消息（不是令牌增量）。
-   **预览流式传输（Telegram/Discord/Slack）：** 在生成时更新临时的**预览消息**。

目前**没有真正的令牌增量流式传输**到频道消息。预览流式传输是基于消息的（发送 + 编辑/追加）。

## 块流式传输（频道消息）

块流式传输在助手输出可用时，以粗粒度的块形式发送。

```
Model output
  └─ text_delta/events
       ├─ (blockStreamingBreak=text_end)
       │    └─ chunker emits blocks as buffer grows
       └─ (blockStreamingBreak=message_end)
            └─ chunker flushes at message_end
                   └─ channel send (block replies)
```

图例：

-   `text_delta/events`：模型流事件（对于非流式模型可能很稀疏）。
-   `chunker`：应用最小/最大边界 + 中断偏好的 `EmbeddedBlockChunker`。
-   `channel send`：实际的外发消息（块回复）。

**控制项：**

-   `agents.defaults.blockStreamingDefault`：`"on"`/`"off"`（默认关闭）。
-   频道覆盖：`*.blockStreaming`（以及每个账户的变体）以强制每个频道为 `"on"`/`"off"`。
-   `agents.defaults.blockStreamingBreak`：`"text_end"` 或 `"message_end"`。
-   `agents.defaults.blockStreamingChunk`：`{ minChars, maxChars, breakPreference? }`。
-   `agents.defaults.blockStreamingCoalesce`：`{ minChars?, maxChars?, idleMs? }`（在发送前合并流式块）。
-   频道硬性限制：`*.textChunkLimit`（例如，`channels.whatsapp.textChunkLimit`）。
-   频道分块模式：`*.chunkMode`（默认 `length`，`newline` 在长度分块前按空行（段落边界）分割）。
-   Discord 软性限制：`channels.discord.maxLinesPerMessage`（默认 17）分割过长的回复以避免 UI 裁剪。

**边界语义：**

-   `text_end`：一旦分块器发出就流式传输块；在每个 `text_end` 时刷新。
-   `message_end`：等待助手消息完成，然后刷新缓冲的输出。

如果缓冲的文本超过 `maxChars`，`message_end` 仍然使用分块器，因此它可以在最后发出多个块。

## 分块算法（低/高边界）

块分块由 `EmbeddedBlockChunker` 实现：

-   **低边界：** 除非强制，否则在缓冲区 >= `minChars` 之前不发出。
-   **高边界：** 倾向于在 `maxChars` 之前分割；如果强制，则在 `maxChars` 处分割。
-   **中断偏好：** `paragraph` → `newline` → `sentence` → `whitespace` → 硬中断。
-   **代码围栏：** 绝不在围栏内分割；当在 `maxChars` 处强制分割时，关闭并重新打开围栏以保持 Markdown 有效。

`maxChars` 被限制在频道 `textChunkLimit` 内，因此您不能超过每个频道的上限。

## 合并（合并流式块）

当块流式传输启用时，OpenClaw 可以在发送之前**合并连续的块分块**。这减少了“单行垃圾信息”，同时仍提供渐进式输出。

-   合并等待**空闲间隙**（`idleMs`）后才刷新。
-   缓冲区受 `maxChars` 限制，如果超过则会刷新。
-   `minChars` 防止微小片段发送，直到累积足够的文本（最终刷新始终发送剩余文本）。
-   连接符源自 `blockStreamingChunk.breakPreference`（`paragraph` → `\n\n`，`newline` → `\n`，`sentence` → 空格）。
-   频道覆盖可通过 `*.blockStreamingCoalesce` 使用（包括每个账户的配置）。
-   除非被覆盖，否则 Signal/Slack/Discord 的默认合并 `minChars` 会提升到 1500。

## 块之间的人性化节奏

当块流式传输启用时，您可以在块回复之间（第一个块之后）添加**随机暂停**。这使得多气泡回复感觉更自然。

-   配置：`agents.defaults.humanDelay`（通过 `agents.list[].humanDelay` 覆盖每个代理）。
-   模式：`off`（默认），`natural`（800–2500ms），`custom`（`minMs`/`maxMs`）。
-   仅适用于**块回复**，不适用于最终回复或工具摘要。

## “流式分块或全部”

这映射为：

-   **流式分块：** `blockStreamingDefault: "on"` + `blockStreamingBreak: "text_end"`（边生成边发出）。非 Telegram 频道还需要 `*.blockStreaming: true`。
-   **结束时流式传输全部：** `blockStreamingBreak: "message_end"`（一次性刷新，如果非常长可能产生多个块）。
-   **无块流式传输：** `blockStreamingDefault: "off"`（仅最终回复）。

**频道注意：** 除非 `*.blockStreaming` 显式设置为 `true`，否则块流式传输**是关闭的**。频道可以在没有块回复的情况下流式传输实时预览（`channels..streaming`）。配置位置提醒：`blockStreaming*` 默认值位于 `agents.defaults` 下，而不是根配置。

## 预览流式传输模式

规范键：`channels..streaming` 模式：

-   `off`：禁用预览流式传输。
-   `partial`：单个预览，被最新文本替换。
-   `block`：预览以分块/追加步骤更新。
-   `progress`：生成期间的进度/状态预览，完成后给出最终答案。

### 频道映射

| 频道 | `off` | `partial` | `block` | `progress` |
| --- | --- | --- | --- | --- |
| Telegram | ✅ | ✅ | ✅ | 映射到 `partial` |
| Discord | ✅ | ✅ | ✅ | 映射到 `partial` |
| Slack | ✅ | ✅ | ✅ | ✅ |

仅限 Slack：

-   `channels.slack.nativeStreaming` 在 `streaming=partial` 时切换 Slack 原生流式传输 API 调用（默认：`true`）。

旧键迁移：

-   Telegram：`streamMode` + 布尔值 `streaming` 自动迁移到 `streaming` 枚举。
-   Discord：`streamMode` + 布尔值 `streaming` 自动迁移到 `streaming` 枚举。
-   Slack：`streamMode` 自动迁移到 `streaming` 枚举；布尔值 `streaming` 自动迁移到 `nativeStreaming`。

### 运行时行为

Telegram：

-   在可用时，在私信中使用 Bot API `sendMessageDraft`，在群组/话题预览更新中使用 `sendMessage` + `editMessageText`。
-   当 Telegram 块流式传输显式启用时，会跳过预览流式传输（以避免双重流式传输）。
-   `/reasoning stream` 可以将推理写入预览。

Discord：

-   使用发送 + 编辑预览消息。
-   `block` 模式使用草稿分块（`draftChunk`）。
-   当 Discord 块流式传输显式启用时，会跳过预览流式传输。

Slack：

-   `partial` 在可用时可以使用 Slack 原生流式传输（`chat.startStream`/`append`/`stop`）。
-   `block` 使用追加式草稿预览。
-   `progress` 使用状态预览文本，然后是最终答案。

[消息](./messages.md)[重试策略](./retry.md)