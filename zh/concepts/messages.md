

  消息与投递

  
# 消息

本页面综合介绍了 OpenClaw 如何处理入站消息、会话、队列、流式传输和推理可见性。

## 消息流（高层级）

```
入站消息
  -> 路由/绑定 -> 会话键
  -> 队列（如果运行处于活动状态）
  -> 智能体运行（流式传输 + 工具调用）
  -> 出站回复（通道限制 + 分块）
```

关键配置项位于配置中：

-   `messages.*` 用于前缀、队列和群组行为。
-   `agents.defaults.*` 用于块流式传输和分块的默认设置。
-   通道覆盖项（`channels.whatsapp.*`、`channels.telegram.*` 等）用于限制和流式传输开关。

完整模式请参见[配置](../gateway/configuration.md)。

## 入站去重

通道在重连后可能会重新投递相同的消息。OpenClaw 维护一个短期的缓存，键由通道/账户/对端/会话/消息ID组成，因此重复的投递不会触发另一次智能体运行。

## 入站防抖

来自**同一发送者**的快速连续消息可以通过 `messages.inbound` 配置批处理为单个智能体回合。防抖作用域为每个通道 + 对话，并使用最新的消息进行回复线程/ID关联。配置（全局默认值 + 每通道覆盖项）：

```json
{
  messages: {
    inbound: {
      debounceMs: 2000,
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
        discord: 1500,
      },
    },
  },
}
```

注意：

-   防抖适用于**纯文本**消息；媒体/附件会立即刷新。
-   控制命令绕过防抖，因此它们保持独立。

## 会话与设备

会话由网关拥有，而非客户端。

-   直接聊天会合并到智能体主会话键中。
-   群组/频道拥有自己的会话键。
-   会话存储和转录记录位于网关主机上。

多个设备/通道可以映射到同一会话，但历史记录不会完全同步回所有客户端。建议：对于长对话，使用一个主设备以避免上下文分歧。控制界面和 TUI 始终显示网关支持的会话转录记录，因此它们是事实来源。详情请见：[会话管理](./session.md)。

## 入站正文与历史上下文

OpenClaw 将**提示正文**与**命令正文**分开：

-   `Body`：发送给智能体的提示文本。这可能包括通道信封和可选的历史包装器。
-   `CommandBody`：用于指令/命令解析的原始用户文本。
-   `RawBody`：`CommandBody` 的旧别名（为兼容性而保留）。

当通道提供历史记录时，它使用一个共享的包装器：

-   `[自您上次回复以来的聊天消息 - 供上下文参考]`
-   `[当前消息 - 请回复此消息]`

对于**非直接聊天**（群组/频道/房间），**当前消息正文**会加上发送者标签前缀（与历史条目使用的样式相同）。这确保了智能体提示中实时消息和排队/历史消息的一致性。历史缓冲区是**仅待处理的**：它们包含*未*触发运行的群组消息（例如，提及门控消息），并**排除**已在会话转录记录中的消息。指令剥离仅适用于**当前消息**部分，因此历史记录保持不变。包装历史记录的通道应将 `CommandBody`（或 `RawBody`）设置为原始消息文本，并将 `Body` 保持为组合后的提示。历史缓冲区可通过 `messages.groupChat.historyLimit`（全局默认值）和每通道覆盖项（如 `channels.slack.historyLimit` 或 `channels.telegram.accounts..historyLimit`）进行配置（设置为 `0` 以禁用）。

## 队列与后续跟进

如果已有运行处于活动状态，入站消息可以被排队、引导到当前运行中，或收集起来用于后续回合。

-   通过 `messages.queue`（和 `messages.queue.byChannel`）配置。
-   模式：`interrupt`、`steer`、`followup`、`collect`，以及积压变体。

详情请见：[队列](./queue.md)。

## 流式传输、分块与批处理

块流式传输在模型生成文本块时发送部分回复。分块功能会遵守通道文本限制并避免拆分围栏代码块。关键设置：

-   `agents.defaults.blockStreamingDefault` (`on|off`，默认 off)
-   `agents.defaults.blockStreamingBreak` (`text_end|message_end`)
-   `agents.defaults.blockStreamingChunk` (`minChars|maxChars|breakPreference`)
-   `agents.defaults.blockStreamingCoalesce`（基于空闲时间的批处理）
-   `agents.defaults.humanDelay`（块回复之间类似人类的暂停）
-   通道覆盖项：`*.blockStreaming` 和 `*.blockStreamingCoalesce`（非 Telegram 通道需要显式设置 `*.blockStreaming: true`）

详情请见：[流式传输 + 分块](./streaming.md)。

## 推理可见性与令牌

OpenClaw 可以暴露或隐藏模型推理：

-   `/reasoning on|off|stream` 控制可见性。
-   推理内容在被模型生成时仍计入令牌使用量。
-   Telegram 支持将推理流式传输到草稿气泡中。

详情请见：[思考 + 推理指令](../tools/thinking.md) 和 [令牌使用](../reference/token-use.md)。

## 前缀、线程与回复

出站消息格式化集中在 `messages` 中：

-   `messages.responsePrefix`、`channels..responsePrefix` 和 `channels..accounts..responsePrefix`（出站前缀级联），以及 `channels.whatsapp.messagePrefix`（WhatsApp 入站前缀）
-   通过 `replyToMode` 和每通道默认值进行回复线程关联

详情请见：[配置](../gateway/configuration.md#messages) 和通道文档。

[在线状态](./presence.md)[流式传输与分块](./streaming.md)