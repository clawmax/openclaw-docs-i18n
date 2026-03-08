

  会话与内存

  
# 压缩

每个模型都有一个**上下文窗口**（它能看到的令牌最大数量）。长时间运行的聊天会累积消息和工具结果；一旦窗口紧张，OpenClaw 就会**压缩**较早的历史记录以保持在限制范围内。

## 什么是压缩

压缩**将较早的对话总结**为一个紧凑的摘要条目，并保持最近的消息完整。摘要存储在会话历史中，因此未来的请求使用：

-   压缩摘要
-   压缩点之后的最近消息

压缩会**持久化**保存在会话的 JSONL 历史中。

## 配置

在 `openclaw.json` 中使用 `agents.defaults.compaction` 设置来配置压缩行为（模式、目标令牌数等）。压缩摘要默认保留不透明标识符（`identifierPolicy: "strict"`）。您可以通过 `identifierPolicy: "off"` 覆盖此设置，或使用 `identifierPolicy: "custom"` 并提供 `identifierInstructions` 来自定义文本。

## 自动压缩（默认开启）

当会话接近或超过模型的上下文窗口时，OpenClaw 会触发自动压缩，并可能使用压缩后的上下文重试原始请求。您会看到：

-   详细模式下显示 `🧹 Auto-compaction complete`
-   `/status` 显示 `🧹 Compactions: `

在压缩之前，OpenClaw 可以运行一个**静默内存刷新**回合，将持久性笔记存储到磁盘。有关详细信息和配置，请参阅[内存](./memory.md)。

## 手动压缩

使用 `/compact`（可选附带指令）来强制执行一次压缩：

```bash
/compact Focus on decisions and open questions
```

## 上下文窗口来源

上下文窗口是模型特定的。OpenClaw 使用来自配置的提供商目录的模型定义来确定限制。

## 压缩与修剪

-   **压缩**：总结并**持久化**到 JSONL。
-   **会话修剪**：仅修剪旧的**工具结果**，**在内存中**，按请求进行。

有关修剪的详细信息，请参阅 [/concepts/session-pruning](./session-pruning.md)。

## OpenAI 服务器端压缩

OpenClaw 还支持 OpenAI Responses 服务器端压缩提示，适用于兼容的直接 OpenAI 模型。这与本地的 OpenClaw 压缩是分开的，并且可以同时运行。

-   本地压缩：OpenClaw 总结并持久化到会话 JSONL。
-   服务器端压缩：当启用 `store` + `context_management` 时，OpenAI 在提供商端压缩上下文。

有关模型参数和覆盖，请参阅 [OpenAI 提供商](../providers/openai.md)。

## 提示

-   当会话感觉陈旧或上下文臃肿时，使用 `/compact`。
-   大型工具输出已被截断；修剪可以进一步减少工具结果的积累。
-   如果您需要一个全新的开始，`/new` 或 `/reset` 会启动一个新的会话 ID。

[内存](./memory.md)[多智能体路由](./multi-agent.md)