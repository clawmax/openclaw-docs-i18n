

  概念内部

  
# 输入指示器

输入指示器会在运行活动时发送到聊天频道。使用 `agents.defaults.typingMode` 来控制输入指示**何时**开始，使用 `typingIntervalSeconds` 来控制其**刷新频率**。

## 默认设置

当 `agents.defaults.typingMode` **未设置**时，OpenClaw 保持旧版行为：

-   **直接聊天**：模型循环一开始，输入指示立即开始。
-   **带提及的群聊**：输入指示立即开始。
-   **不带提及的群聊**：仅当消息文本开始流式传输时，输入指示才开始。
-   **心跳运行**：输入指示被禁用。

## 模式

将 `agents.defaults.typingMode` 设置为以下之一：

-   `never` — 从不显示输入指示。
-   `instant` — **模型循环一开始**就立即开始输入指示，即使运行稍后仅返回静默回复令牌。
-   `thinking` — 在**第一个推理增量**时开始输入指示（要求运行设置 `reasoningLevel: "stream"`）。
-   `message` — 在**第一个非静默文本增量**时开始输入指示（忽略 `NO_REPLY` 静默令牌）。

“触发早晚”的顺序：`never` → `message` → `thinking` → `instant`

## 配置

```json
{
  agent: {
    typingMode: "thinking",
    typingIntervalSeconds: 6,
  },
}
```

您可以按会话覆盖模式或频率：

```json
{
  session: {
    typingMode: "message",
    typingIntervalSeconds: 4,
  },
}
```

## 注意事项

-   `message` 模式不会为纯静默回复（例如用于抑制输出的 `NO_REPLY` 令牌）显示输入指示。
-   `thinking` 仅在运行流式传输推理（`reasoningLevel: "stream"`）时触发。如果模型不发出推理增量，输入指示将不会开始。
-   无论模式如何，心跳运行从不显示输入指示。
-   `typingIntervalSeconds` 控制**刷新频率**，而非开始时间。默认值为 6 秒。

[Markdown 格式化](./markdown-formatting.md)[使用情况追踪](./usage-tracking.md)