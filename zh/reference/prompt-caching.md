

  技术参考

  
# 提示词缓存

提示词缓存意味着模型提供商可以跨对话轮次复用未更改的提示词前缀（通常是系统/开发者指令和其他稳定上下文），而不是每次都重新处理它们。首次匹配的请求会写入缓存令牌（`cacheWrite`），后续匹配的请求可以读取它们（`cacheRead`）。这很重要：降低令牌成本、加速响应，并为长时间运行的会话提供更可预测的性能。如果没有缓存，即使大部分输入没有变化，重复的提示词也会在每一轮支付完整的提示词成本。本页涵盖所有影响提示词复用和令牌成本的缓存相关配置项。有关 Anthropic 定价详情，请参阅：[https://docs.anthropic.com/docs/build-with-claude/prompt-caching](https://docs.anthropic.com/docs/build-with-claude/prompt-caching)

## 主要配置项

### cacheRetention（模型级和每个智能体）

在模型参数上设置缓存保留：

```yaml
agents:
  defaults:
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "short" # none | short | long
```

每个智能体的覆盖设置：

```yaml
agents:
  list:
    - id: "alerts"
      params:
        cacheRetention: "none"
```

配置合并顺序：

1.  `agents.defaults.models["provider/model"].params`
2.  `agents.list[].params`（匹配智能体 id；按键覆盖）

### 旧版 cacheControlTtl

旧版值仍被接受并映射：

-   `5m` -> `short`
-   `1h` -> `long`

新配置建议使用 `cacheRetention`。

### contextPruning.mode: "cache-ttl"

在缓存 TTL 窗口后修剪旧的工具结果上下文，以便空闲后的请求不会重新缓存过大的历史记录。

```yaml
agents:
  defaults:
    contextPruning:
      mode: "cache-ttl"
      ttl: "1h"
```

完整行为请参阅[会话修剪](../concepts/session-pruning.md)。

### 心跳保活

心跳可以保持缓存窗口处于活跃状态，并减少空闲间隔后的重复缓存写入。

```yaml
agents:
  defaults:
    heartbeat:
      every: "55m"
```

每个智能体的心跳支持在 `agents.list[].heartbeat` 中设置。

## 提供商行为

### Anthropic（直接 API）

-   支持 `cacheRetention`。
-   对于使用 Anthropic API 密钥认证的配置文件，当未设置时，OpenClaw 会为 Anthropic 模型引用注入 `cacheRetention: "short"`。

### Amazon Bedrock

-   Anthropic Claude 模型引用（`amazon-bedrock/*anthropic.claude*`）支持显式的 `cacheRetention` 透传。
-   非 Anthropic 的 Bedrock 模型在运行时会被强制设置为 `cacheRetention: "none"`。

### OpenRouter Anthropic 模型

对于 `openrouter/anthropic/*` 模型引用，OpenClaw 会在系统/开发者提示词块中注入 Anthropic 的 `cache_control`，以提高提示词缓存的复用率。

### 其他提供商

如果提供商不支持此缓存模式，则 `cacheRetention` 无效。

## 调优模式

### 混合流量（推荐默认值）

在主智能体上保持长期存活的基线，在突发性通知智能体上禁用缓存：

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-6"
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "long"
  list:
    - id: "research"
      default: true
      heartbeat:
        every: "55m"
    - id: "alerts"
      params:
        cacheRetention: "none"
```

### 成本优先基线

-   设置基线 `cacheRetention: "short"`。
-   启用 `contextPruning.mode: "cache-ttl"`。
-   仅对受益于热缓存的智能体，将心跳保持在低于您的 TTL。

## 缓存诊断

OpenClaw 为嵌入式智能体运行提供专用的缓存跟踪诊断。

### diagnostics.cacheTrace 配置

```yaml
diagnostics:
  cacheTrace:
    enabled: true
    filePath: "~/.openclaw/logs/cache-trace.jsonl" # 可选
    includeMessages: false # 默认 true
    includePrompt: false # 默认 true
    includeSystem: false # 默认 true
```

默认值：

-   `filePath`: `$OPENCLAW_STATE_DIR/logs/cache-trace.jsonl`
-   `includeMessages`: `true`
-   `includePrompt`: `true`
-   `includeSystem`: `true`

### 环境变量开关（一次性调试）

-   `OPENCLAW_CACHE_TRACE=1` 启用缓存跟踪。
-   `OPENCLAW_CACHE_TRACE_FILE=/path/to/cache-trace.jsonl` 覆盖输出路径。
-   `OPENCLAW_CACHE_TRACE_MESSAGES=0|1` 切换完整消息负载捕获。
-   `OPENCLAW_CACHE_TRACE_PROMPT=0|1` 切换提示词文本捕获。
-   `OPENCLAW_CACHE_TRACE_SYSTEM=0|1` 切换系统提示词捕获。

### 检查内容

-   缓存跟踪事件为 JSONL 格式，包含分阶段快照，如 `session:loaded`、`prompt:before`、`stream:context` 和 `session:after`。
-   每轮对话的缓存令牌影响可通过 `cacheRead` 和 `cacheWrite` 在正常使用界面中查看（例如 `/usage full` 和会话使用摘要）。

## 快速故障排除

-   大多数轮次出现高 `cacheWrite`：检查易变的系统提示词输入，并验证模型/提供商支持您的缓存设置。
-   `cacheRetention` 无效：确认模型键匹配 `agents.defaults.models["provider/model"]`。
-   带有缓存设置的 Bedrock Nova/Mistral 请求：预期在运行时强制设置为 `none`。

相关文档：

-   [Anthropic](../providers/anthropic.md)
-   [令牌使用与成本](./token-use.md)
-   [会话修剪](../concepts/session-pruning.md)
-   [网关配置参考](../gateway/configuration-reference.md)

[SecretRef 凭证界面](./secretref-credential-surface.md)[API 使用与成本](./api-usage-costs.md)