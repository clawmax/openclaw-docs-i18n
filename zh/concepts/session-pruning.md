

  会话与内存

  
# 会话修剪

会话修剪会在每次LLM调用前，从内存上下文中**移除旧的工具结果**。它**不会**重写磁盘上的会话历史记录（`*.jsonl`）。

## 何时运行

-   当启用 `mode: "cache-ttl"` 且该会话最后一次Anthropic调用时间早于 `ttl` 时。
-   仅影响发送给模型用于该请求的消息。
-   仅对Anthropic API调用（以及OpenRouter的Anthropic模型）生效。
-   为获得最佳效果，请将 `ttl` 与您的模型 `cacheRetention` 策略匹配（`short` = 5分钟，`long` = 1小时）。
-   修剪后，TTL窗口会重置，因此后续请求将继续使用缓存，直到 `ttl` 再次过期。

## 智能默认值（Anthropic）

-   **OAuth 或 setup-token** 配置文件：启用 `cache-ttl` 修剪，并将心跳设置为 `1h`。
-   **API key** 配置文件：启用 `cache-ttl` 修剪，将心跳设置为 `30m`，并在Anthropic模型上默认设置 `cacheRetention: "short"`。
-   如果您显式设置了这些值中的任何一个，OpenClaw**不会**覆盖它们。

## 改进之处（成本 + 缓存行为）

-   **为何修剪：** Anthropic提示缓存仅在TTL内有效。如果会话闲置时间超过TTL，除非您先进行修剪，否则下一个请求将重新缓存整个提示。
-   **降低哪些成本：** 修剪减少了TTL过期后第一个请求的**cacheWrite**大小。
-   **TTL重置为何重要：** 一旦修剪运行，缓存窗口就会重置，因此后续请求可以重用新缓存的提示，而不是再次重新缓存整个历史记录。
-   **它不做什么：** 修剪不会增加令牌或产生“双重”成本；它只改变TTL过期后第一个请求中缓存的内容。

## 可修剪的内容

-   仅限 `toolResult` 消息。
-   用户消息和助手消息**永不**被修改。
-   最后 `keepLastAssistants` 条助手消息受到保护；该截止点之后的工具结果不会被修剪。
-   如果没有足够的助手消息来确定截止点，则跳过修剪。
-   包含**图像块**的工具结果会被跳过（永不修剪/清除）。

## 上下文窗口估算

修剪使用估算的上下文窗口（字符数 ≈ 令牌数 × 4）。基础窗口按以下顺序解析：

1.  `models.providers.*.models[].contextWindow` 覆盖设置。
2.  模型定义中的 `contextWindow`（来自模型注册表）。
3.  默认值 `200000` 令牌。

如果设置了 `agents.defaults.contextTokens`，它将被视为对已解析窗口的上限（最小值）。

## 模式

### cache-ttl

-   仅当最后一次Anthropic调用时间早于 `ttl`（默认 `5m`）时，修剪才会运行。
-   运行时行为：与之前相同的软修剪 + 硬清除行为。

## 软修剪与硬修剪

-   **软修剪**：仅针对过大的工具结果。
    -   保留头部和尾部，插入 `...`，并附加一条包含原始大小的说明。
    -   跳过包含图像块的结果。
-   **硬清除**：用 `hardClear.placeholder` 替换整个工具结果。

## 工具选择

-   `tools.allow` / `tools.deny` 支持 `*` 通配符。
-   Deny 规则优先。
-   匹配不区分大小写。
-   空的允许列表 => 允许所有工具。

## 与其他限制的交互

-   内置工具已经会截断自己的输出；会话修剪是额外的一层，防止长时间运行的聊天在模型上下文中积累过多的工具输出。
-   压缩是独立的：压缩会进行总结并持久化，而修剪是每个请求的临时操作。请参阅 [/concepts/compaction](./compaction.md)。

## 默认值（启用时）

-   `ttl`: `"5m"`
-   `keepLastAssistants`: `3`
-   `softTrimRatio`: `0.3`
-   `hardClearRatio`: `0.5`
-   `minPrunableToolChars`: `50000`
-   `softTrim`: `{ maxChars: 4000, headChars: 1500, tailChars: 1500 }`
-   `hardClear`: `{ enabled: true, placeholder: "[旧工具结果内容已清除]" }`

## 示例

默认（关闭）：

```json
{
  agents: { defaults: { contextPruning: { mode: "off" } } },
}
```

启用基于TTL的修剪：

```json
{
  agents: { defaults: { contextPruning: { mode: "cache-ttl", ttl: "5m" } } },
}
```

将修剪限制在特定工具：

```json
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "cache-ttl",
        tools: { allow: ["exec", "read"], deny: ["*image*"] },
      },
    },
  },
}
```

查看配置参考：[网关配置](../gateway/configuration.md)

[会话管理](./session.md)[会话工具](./session-tool.md)