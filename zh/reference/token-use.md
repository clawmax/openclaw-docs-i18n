

  技术参考

  
# 令牌使用与成本

OpenClaw 跟踪的是**令牌**，而非字符。令牌是模型特定的，但大多数 OpenAI 风格的模型对于英文文本平均每个令牌约 4 个字符。

## 系统提示是如何构建的

OpenClaw 在每次运行时都会组装自己的系统提示。它包括：

-   工具列表 + 简短描述
-   技能列表（仅元数据；指令通过 `read` 按需加载）
-   自我更新指令
-   工作空间 + 引导文件（新建时的 `AGENTS.md`、`SOUL.md`、`TOOLS.md`、`IDENTITY.md`、`USER.md`、`HEARTBEAT.md`、`BOOTSTRAP.md`，以及存在时的 `MEMORY.md` 和/或 `memory.md`）。大文件会被 `agents.defaults.bootstrapMaxChars`（默认值：20000）截断，并且引导注入的总量受 `agents.defaults.bootstrapTotalMaxChars`（默认值：150000）限制。`memory/*.md` 文件通过记忆工具按需加载，不会自动注入。
-   时间（UTC + 用户时区）
-   回复标签 + 心跳行为
-   运行时元数据（主机/操作系统/模型/思考）

完整分解请参见[系统提示](../concepts/system-prompt.md)。

## 上下文窗口中包含哪些内容

模型接收到的所有内容都计入上下文限制：

-   系统提示（上述所有部分）
-   对话历史记录（用户 + 助手消息）
-   工具调用和工具结果
-   附件/转录内容（图像、音频、文件）
-   压缩摘要和修剪产物
-   提供商包装器或安全头（不可见，但仍计入）

对于图像，OpenClaw 在调用提供商之前会先对转录/工具图像载荷进行降采样。使用 `agents.defaults.imageMaxDimensionPx`（默认值：`1200`）来调整此设置：

-   较低的值通常会减少视觉令牌使用量和载荷大小。
-   较高的值会为 OCR/UI 密集的截图保留更多视觉细节。

要获取实际分解（按注入文件、工具、技能和系统提示大小），请使用 `/context list` 或 `/context detail`。参见[上下文](../concepts/context.md)。

## 如何查看当前令牌使用情况

在聊天中使用以下命令：

-   `/status` → 显示**包含丰富表情符号的状态卡片**，包含会话模型、上下文使用情况、上次响应的输入/输出令牌以及**估算成本**（仅限 API 密钥）。
-   `/usage off|tokens|full` → 为每条回复附加一个**每次响应的使用情况页脚**。
    -   每个会话持久保存（存储为 `responseUsage`）。
    -   OAuth 身份验证**隐藏成本**（仅显示令牌）。
-   `/usage cost` → 显示来自 OpenClaw 会话日志的本地成本摘要。

其他界面：

-   **TUI/Web TUI：** 支持 `/status` + `/usage`。
-   **CLI：** `openclaw status --usage` 和 `openclaw channels list` 显示提供商配额窗口（非每次响应成本）。

## 成本估算（何时显示）

成本根据您的模型定价配置估算：

```
models.providers.<provider>.models[].cost
```

这些是 `input`、`output`、`cacheRead` 和 `cacheWrite` 的**每 100 万令牌美元价格**。如果缺少定价信息，OpenClaw 仅显示令牌。OAuth 令牌从不显示美元成本。

## 缓存 TTL 和修剪的影响

提供商提示缓存仅在缓存 TTL 窗口内有效。OpenClaw 可以选择运行**缓存 TTL 修剪**：一旦缓存 TTL 过期，它会修剪会话，然后重置缓存窗口，以便后续请求可以重新使用新缓存的上下文，而不是重新缓存完整的历史记录。这可以在会话闲置超过 TTL 后保持较低的缓存写入成本。在[网关配置](../gateway/configuration.md)中配置此功能，并在[会话修剪](../concepts/session-pruning.md)中查看行为详情。心跳可以保持缓存在闲置间隙中处于**预热**状态。如果您的模型缓存 TTL 是 `1h`，将心跳间隔设置为略低于此值（例如 `55m`）可以避免重新缓存完整提示，从而降低缓存写入成本。在多代理设置中，您可以保留一个共享的模型配置，并通过 `agents.list[].params.cacheRetention` 为每个代理调整缓存行为。完整的逐项指南，请参见[提示缓存](./prompt-caching.md)。对于 Anthropic API 定价，缓存读取比输入令牌便宜得多，而缓存写入则按更高的乘数计费。有关最新费率和 TTL 乘数，请参阅 Anthropic 的提示缓存定价：[https://docs.anthropic.com/docs/build-with-claude/prompt-caching](https://docs.anthropic.com/docs/build-with-claude/prompt-caching)

### 示例：通过心跳保持 1 小时缓存预热

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-6"
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "long"
    heartbeat:
      every: "55m"
```

### 示例：混合流量下的按代理缓存策略

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-6"
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "long" # 大多数代理的默认基线
  list:
    - id: "research"
      default: true
      heartbeat:
        every: "55m" # 为深度会话保持长缓存预热
    - id: "alerts"
      params:
        cacheRetention: "none" # 避免为突发性通知写入缓存
```

`agents.list[].params` 会合并到所选模型的 `params` 之上，因此您可以仅覆盖 `cacheRetention` 并继承其他模型默认值不变。

### 示例：启用 Anthropic 100 万上下文 Beta 头

Anthropic 的 100 万上下文窗口目前处于 Beta 测试阶段。当您在支持的 Opus 或 Sonnet 模型上启用 `context1m` 时，OpenClaw 可以注入所需的 `anthropic-beta` 值。

```yaml
agents:
  defaults:
    models:
      "anthropic/claude-opus-4-6":
        params:
          context1m: true
```

这映射到 Anthropic 的 `context-1m-2025-08-07` beta 头。这仅在为该模型条目设置 `context1m: true` 时适用。要求：凭证必须符合长上下文使用资格（API 密钥计费，或启用了额外使用量的订阅）。否则，Anthropic 会响应 `HTTP 429: rate_limit_error: Extra usage is required for long context requests`。如果您使用 OAuth/订阅令牌（`sk-ant-oat-*`）进行 Anthropic 身份验证，OpenClaw 会跳过 `context-1m-*` beta 头，因为 Anthropic 目前会以 HTTP 401 拒绝该组合。

## 减少令牌压力的技巧

-   使用 `/compact` 来总结长会话。
-   在工作流中修剪大型工具输出。
-   对于截图密集的会话，降低 `agents.defaults.imageMaxDimensionPx`。
-   保持技能描述简短（技能列表会注入到提示中）。
-   对于冗长的探索性工作，优先使用较小的模型。

有关确切的技能列表开销公式，请参见[技能](../tools/skills.md)。

[向导参考](./wizard.md)[SecretRef 凭证界面](./secretref-credential-surface.md)