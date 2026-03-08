

  提供商

  
# OpenAI

OpenAI 为 GPT 模型提供开发者 API。Codex 支持 **ChatGPT 登录**以获取订阅访问权限，或支持 **API 密钥**登录以获取按使用量计费的访问权限。Codex 云端服务需要 ChatGPT 登录。OpenAI 明确支持在 OpenClaw 等外部工具/工作流中使用订阅 OAuth。

## 选项 A：OpenAI API 密钥（OpenAI 平台）

**最适合：** 直接 API 访问和按使用量计费。从 OpenAI 仪表板获取您的 API 密钥。

### CLI 设置

```bash
openclaw onboard --auth-choice openai-api-key
# 或非交互式
openclaw onboard --openai-api-key "$OPENAI_API_KEY"
```

### 配置片段

```json
{
  env: { OPENAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "openai/gpt-5.4" } } },
}
```

OpenAI 当前的 API 模型文档列出了用于直接 OpenAI API 使用的 `gpt-5.4` 和 `gpt-5.4-pro`。OpenClaw 通过 `openai/*` Responses 路径转发两者。

## 选项 B：OpenAI Code (Codex) 订阅

**最适合：** 使用 ChatGPT/Codex 订阅访问权限而非 API 密钥。Codex 云端服务需要 ChatGPT 登录，而 Codex CLI 支持 ChatGPT 或 API 密钥登录。

### CLI 设置（Codex OAuth）

```bash
# 在向导中运行 Codex OAuth
openclaw onboard --auth-choice openai-codex

# 或直接运行 OAuth
openclaw models auth login --provider openai-codex
```

### 配置片段（Codex 订阅）

```json
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.4" } } },
}
```

OpenAI 当前的 Codex 文档将 `gpt-5.4` 列为当前的 Codex 模型。OpenClaw 将其映射为 `openai-codex/gpt-5.4` 以用于 ChatGPT/Codex OAuth 使用。

### 传输默认值

OpenClaw 使用 `pi-ai` 进行模型流式传输。对于 `openai/*` 和 `openai-codex/*`，默认传输是 `"auto"`（优先 WebSocket，然后 SSE 回退）。您可以设置 `agents.defaults.models.<provider/model>.params.transport`：

-   `"sse"`：强制使用 SSE
-   `"websocket"`：强制使用 WebSocket
-   `"auto"`：尝试 WebSocket，然后回退到 SSE

对于 `openai/*`（Responses API），当使用 WebSocket 传输时，OpenClaw 默认还启用了 WebSocket 预热（`openaiWsWarmup: true`）。相关的 OpenAI 文档：

-   [使用 WebSocket 的实时 API](https://platform.openai.com/docs/guides/realtime-websocket)
-   [流式 API 响应 (SSE)](https://platform.openai.com/docs/guides/streaming-responses)

```json
{
  agents: {
    defaults: {
      model: { primary: "openai-codex/gpt-5.4" },
      models: {
        "openai-codex/gpt-5.4": {
          params: {
            transport: "auto",
          },
        },
      },
    },
  },
}
```

### OpenAI WebSocket 预热

OpenAI 文档将预热描述为可选的。OpenClaw 默认对 `openai/*` 启用它，以减少使用 WebSocket 传输时的首轮延迟。

### 禁用预热

```json
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            openaiWsWarmup: false,
          },
        },
      },
    },
  },
}
```

### 显式启用预热

```json
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            openaiWsWarmup: true,
          },
        },
      },
    },
  },
}
```

### OpenAI 优先级处理

OpenAI 的 API 通过 `service_tier=priority` 公开优先级处理。在 OpenClaw 中，设置 `agents.defaults.models["openai/"].params.serviceTier` 以在直接的 `openai/*` Responses 请求中传递该字段。

```json
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            serviceTier: "priority",
          },
        },
      },
    },
  },
}
```

支持的值为 `auto`、`default`、`flex` 和 `priority`。

### OpenAI Responses 服务器端压缩

对于直接的 OpenAI Responses 模型（使用 `api: "openai-responses"` 且 `baseUrl` 为 `api.openai.com` 的 `openai/*`），OpenClaw 现在自动启用 OpenAI 服务器端压缩负载提示：

-   强制 `store: true`（除非模型兼容性设置 `supportsStore: false`）
-   注入 `context_management: [{ type: "compaction", compact_threshold: ... }]`

默认情况下，`compact_threshold` 是模型 `contextWindow` 的 `70%`（或不可用时为 `80000`）。

### 显式启用服务器端压缩

当您想要在兼容的 Responses 模型（例如 Azure OpenAI Responses）上强制注入 `context_management` 时，请使用此设置：

```json
{
  agents: {
    defaults: {
      models: {
        "azure-openai-responses/gpt-5.4": {
          params: {
            responsesServerCompaction: true,
          },
        },
      },
    },
  },
}
```

### 使用自定义阈值启用

```json
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            responsesServerCompaction: true,
            responsesCompactThreshold: 120000,
          },
        },
      },
    },
  },
}
```

### 禁用服务器端压缩

```json
{
  agents: {
    defaults: {
      models: {
        "openai/gpt-5.4": {
          params: {
            responsesServerCompaction: false,
          },
        },
      },
    },
  },
}
```

`responsesServerCompaction` 仅控制 `context_management` 注入。直接的 OpenAI Responses 模型仍然强制 `store: true`，除非兼容性设置 `supportsStore: false`。

## 注意事项

-   模型引用始终使用 `provider/model`（参见 [/concepts/models](../concepts/models.md)）。
-   身份验证详情和重用规则在 [/concepts/oauth](../concepts/oauth.md) 中。

[Ollama](./ollama.md)[OpenCode Zen](./opencode.md)