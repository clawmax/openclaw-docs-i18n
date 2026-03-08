

  配置

  
# 模型提供商

本页面涵盖 **LLM/模型提供商**（非 WhatsApp/Telegram 等聊天频道）。关于模型选择规则，请参阅 [/concepts/models](./models.md)。

## 快速规则

-   模型引用使用 `提供商/模型` 格式（例如：`opencode/claude-opus-4-6`）。
-   如果设置了 `agents.defaults.models`，它将变为允许列表。
-   CLI 辅助命令：`openclaw onboard`、`openclaw models list`、`openclaw models set <提供商/模型>`。

## API 密钥轮换

-   支持为选定的提供商进行通用提供商轮换。
-   通过以下方式配置多个密钥：
    -   `OPENCLAW_LIVE__KEY`（单个实时覆盖，最高优先级）
    -   `_API_KEYS`（逗号或分号分隔的列表）
    -   `_API_KEY`（主密钥）
    -   `_API_KEY_*`（编号列表，例如 `_API_KEY_1`）
-   对于 Google 提供商，`GOOGLE_API_KEY` 也作为后备密钥包含在内。
-   密钥选择顺序保持优先级并去重。
-   仅在收到速率限制响应（例如 `429`、`rate_limit`、`quota`、`resource exhausted`）时，才会使用下一个密钥重试请求。
-   非速率限制的失败会立即失败；不会尝试密钥轮换。
-   当所有候选密钥都失败时，将返回最后一次尝试的最终错误。

## 内置提供商（pi-ai 目录）

OpenClaw 附带 pi‑ai 目录。这些提供商**无需** `models.providers` 配置；只需设置身份验证并选择一个模型。

### OpenAI

-   提供商：`openai`
-   身份验证：`OPENAI_API_KEY`
-   可选轮换：`OPENAI_API_KEYS`、`OPENAI_API_KEY_1`、`OPENAI_API_KEY_2`，以及 `OPENCLAW_LIVE_OPENAI_KEY`（单个覆盖）
-   示例模型：`openai/gpt-5.1-codex`
-   CLI：`openclaw onboard --auth-choice openai-api-key`
-   默认传输方式为 `auto`（WebSocket 优先，SSE 后备）
-   可通过 `agents.defaults.models["openai/"].params.transport` 按模型覆盖（`"sse"`、`"websocket"` 或 `"auto"`）
-   OpenAI 响应 WebSocket 预热默认通过 `params.openaiWsWarmup` 启用（`true`/`false`）

```json
{
  agents: { defaults: { model: { primary: "openai/gpt-5.1-codex" } } },
}
```

### Anthropic

-   提供商：`anthropic`
-   身份验证：`ANTHROPIC_API_KEY` 或 `claude setup-token`
-   可选轮换：`ANTHROPIC_API_KEYS`、`ANTHROPIC_API_KEY_1`、`ANTHROPIC_API_KEY_2`，以及 `OPENCLAW_LIVE_ANTHROPIC_KEY`（单个覆盖）
-   示例模型：`anthropic/claude-opus-4-6`
-   CLI：`openclaw onboard --auth-choice token`（粘贴 setup-token）或 `openclaw models auth paste-token --provider anthropic`
-   政策说明：setup-token 支持是技术兼容性；Anthropic 过去曾阻止在 Claude Code 之外使用某些订阅。请核实当前 Anthropic 条款，并根据您的风险承受能力决定。
-   建议：Anthropic API 密钥身份验证是比订阅 setup-token 身份验证更安全、推荐的方式。

```json
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

### OpenAI Code (Codex)

-   提供商：`openai-codex`
-   身份验证：OAuth (ChatGPT)
-   示例模型：`openai-codex/gpt-5.3-codex`
-   CLI：`openclaw onboard --auth-choice openai-codex` 或 `openclaw models auth login --provider openai-codex`
-   默认传输方式为 `auto`（WebSocket 优先，SSE 后备）
-   可通过 `agents.defaults.models["openai-codex/"].params.transport` 按模型覆盖（`"sse"`、`"websocket"` 或 `"auto"`）
-   政策说明：OpenAI Codex OAuth 明确支持像 OpenClaw 这样的外部工具/工作流。

```json
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.3-codex" } } },
}
```

### OpenCode Zen

-   提供商：`opencode`
-   身份验证：`OPENCODE_API_KEY`（或 `OPENCODE_ZEN_API_KEY`）
-   示例模型：`opencode/claude-opus-4-6`
-   CLI：`openclaw onboard --auth-choice opencode-zen`

```json
{
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-6" } } },
}
```

### Google Gemini (API 密钥)

-   提供商：`google`
-   身份验证：`GEMINI_API_KEY`
-   可选轮换：`GEMINI_API_KEYS`、`GEMINI_API_KEY_1`、`GEMINI_API_KEY_2`、`GOOGLE_API_KEY` 后备，以及 `OPENCLAW_LIVE_GEMINI_KEY`（单个覆盖）
-   示例模型：`google/gemini-3-pro-preview`
-   CLI：`openclaw onboard --auth-choice gemini-api-key`

### Google Vertex、Antigravity 和 Gemini CLI

-   提供商：`google-vertex`、`google-antigravity`、`google-gemini-cli`
-   身份验证：Vertex 使用 gcloud ADC；Antigravity/Gemini CLI 使用各自的身份验证流程
-   注意：OpenClaw 中的 Antigravity 和 Gemini CLI OAuth 是非官方集成。一些用户报告在使用第三方客户端后 Google 账户受到限制。请查阅 Google 条款，如果选择继续，请使用非关键账户。
-   Antigravity OAuth 作为捆绑插件提供（`google-antigravity-auth`，默认禁用）。
    -   启用：`openclaw plugins enable google-antigravity-auth`
    -   登录：`openclaw models auth login --provider google-antigravity --set-default`
-   Gemini CLI OAuth 作为捆绑插件提供（`google-gemini-cli-auth`，默认禁用）。
    -   启用：`openclaw plugins enable google-gemini-cli-auth`
    -   登录：`openclaw models auth login --provider google-gemini-cli --set-default`
    -   注意：您**无需**将客户端 ID 或密钥粘贴到 `openclaw.json` 中。CLI 登录流程将令牌存储在网关主机上的身份验证配置文件中。

### Z.AI (GLM)

-   提供商：`zai`
-   身份验证：`ZAI_API_KEY`
-   示例模型：`zai/glm-5`
-   CLI：`openclaw onboard --auth-choice zai-api-key`
    -   别名：`z.ai/*` 和 `z-ai/*` 会标准化为 `zai/*`

### Vercel AI Gateway

-   提供商：`vercel-ai-gateway`
-   身份验证：`AI_GATEWAY_API_KEY`
-   示例模型：`vercel-ai-gateway/anthropic/claude-opus-4.6`
-   CLI：`openclaw onboard --auth-choice ai-gateway-api-key`

### Kilo Gateway

-   提供商：`kilocode`
-   身份验证：`KILOCODE_API_KEY`
-   示例模型：`kilocode/anthropic/claude-opus-4.6`
-   CLI：`openclaw onboard --kilocode-api-key `
-   基础 URL：`https://api.kilo.ai/api/gateway/`
-   扩展的内置目录包括 GLM-5 Free、MiniMax M2.5 Free、GPT-5.2、Gemini 3 Pro Preview、Gemini 3 Flash Preview、Grok Code Fast 1 和 Kimi K2.5。

有关设置详情，请参阅 [/providers/kilocode](../providers/kilocode.md)。

### 其他内置提供商

-   OpenRouter：`openrouter` (`OPENROUTER_API_KEY`)
-   示例模型：`openrouter/anthropic/claude-sonnet-4-5`
-   Kilo Gateway：`kilocode` (`KILOCODE_API_KEY`)
-   示例模型：`kilocode/anthropic/claude-opus-4.6`
-   xAI：`xai` (`XAI_API_KEY`)
-   Mistral：`mistral` (`MISTRAL_API_KEY`)
-   示例模型：`mistral/mistral-large-latest`
-   CLI：`openclaw onboard --auth-choice mistral-api-key`
-   Groq：`groq` (`GROQ_API_KEY`)
-   Cerebras：`cerebras` (`CEREBRAS_API_KEY`)
    -   Cerebras 上的 GLM 模型使用 ID `zai-glm-4.7` 和 `zai-glm-4.6`。
    -   OpenAI 兼容的基础 URL：`https://api.cerebras.ai/v1`。
-   GitHub Copilot：`github-copilot` (`COPILOT_GITHUB_TOKEN` / `GH_TOKEN` / `GITHUB_TOKEN`)
-   Hugging Face Inference：`huggingface` (`HUGGINGFACE_HUB_TOKEN` 或 `HF_TOKEN`) — OpenAI 兼容的路由器；示例模型：`huggingface/deepseek-ai/DeepSeek-R1`；CLI：`openclaw onboard --auth-choice huggingface-api-key`。请参阅 [Hugging Face (Inference)](../providers/huggingface.md)。

## 通过 models.providers 配置的提供商（自定义/基础 URL）

使用 `models.providers`（或 `models.json`）添加**自定义**提供商或 OpenAI/Anthropic 兼容的代理。

### Moonshot AI (Kimi)

Moonshot 使用 OpenAI 兼容的端点，因此将其配置为自定义提供商：

-   提供商：`moonshot`
-   身份验证：`MOONSHOT_API_KEY`
-   示例模型：`moonshot/kimi-k2.5`

Kimi K2 模型 ID：

-   `moonshot/kimi-k2.5`
-   `moonshot/kimi-k2-0905-preview`
-   `moonshot/kimi-k2-turbo-preview`
-   `moonshot/kimi-k2-thinking`
-   `moonshot/kimi-k2-thinking-turbo`

```json
{
  agents: {
    defaults: { model: { primary: "moonshot/kimi-k2.5" } },
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [{ id: "kimi-k2.5", name: "Kimi K2.5" }],
      },
    },
  },
}
```

### Kimi Coding

Kimi Coding 使用 Moonshot AI 的 Anthropic 兼容端点：

-   提供商：`kimi-coding`
-   身份验证：`KIMI_API_KEY`
-   示例模型：`kimi-coding/k2p5`

```json
{
  env: { KIMI_API_KEY: "sk-..." },
  agents: {
    defaults: { model: { primary: "kimi-coding/k2p5" } },
  },
}
```

### Qwen OAuth (免费层)

Qwen 通过设备代码流程提供对 Qwen Coder + Vision 的 OAuth 访问。启用捆绑插件，然后登录：

```bash
openclaw plugins enable qwen-portal-auth
openclaw models auth login --provider qwen-portal --set-default
```

模型引用：

-   `qwen-portal/coder-model`
-   `qwen-portal/vision-model`

有关设置详情和说明，请参阅 [/providers/qwen](../providers/qwen.md)。

### Volcano Engine (豆包)

Volcano Engine (火山引擎) 在中国提供对豆包和其他模型的访问。

-   提供商：`volcengine`（编程：`volcengine-plan`）
-   身份验证：`VOLCANO_ENGINE_API_KEY`
-   示例模型：`volcengine/doubao-seed-1-8-251228`
-   CLI：`openclaw onboard --auth-choice volcengine-api-key`

```json
{
  agents: {
    defaults: { model: { primary: "volcengine/doubao-seed-1-8-251228" } },
  },
}
```

可用模型：

-   `volcengine/doubao-seed-1-8-251228` (豆包 Seed 1.8)
-   `volcengine/doubao-seed-code-preview-251028`
-   `volcengine/kimi-k2-5-260127` (Kimi K2.5)
-   `volcengine/glm-4-7-251222` (GLM 4.7)
-   `volcengine/deepseek-v3-2-251201` (DeepSeek V3.2 128K)

编程模型 (`volcengine-plan`)：

-   `volcengine-plan/ark-code-latest`
-   `volcengine-plan/doubao-seed-code`
-   `volcengine-plan/kimi-k2.5`
-   `volcengine-plan/kimi-k2-thinking`
-   `volcengine-plan/glm-4.7`

### BytePlus (国际版)

BytePlus ARK 为国际用户提供与 Volcano Engine 相同的模型访问。

-   提供商：`byteplus`（编程：`byteplus-plan`）
-   身份验证：`BYTEPLUS_API_KEY`
-   示例模型：`byteplus/seed-1-8-251228`
-   CLI：`openclaw onboard --auth-choice byteplus-api-key`

```json
{
  agents: {
    defaults: { model: { primary: "byteplus/seed-1-8-251228" } },
  },
}
```

可用模型：

-   `byteplus/seed-1-8-251228` (Seed 1.8)
-   `byteplus/kimi-k2-5-260127` (Kimi K2.5)
-   `byteplus/glm-4-7-251222` (GLM 4.7)

编程模型 (`byteplus-plan`)：

-   `byteplus-plan/ark-code-latest`
-   `byteplus-plan/doubao-seed-code`
-   `byteplus-plan/kimi-k2.5`
-   `byteplus-plan/kimi-k2-thinking`
-   `byteplus-plan/glm-4.7`

### Synthetic

Synthetic 在 `synthetic` 提供商下提供 Anthropic 兼容的模型：

-   提供商：`synthetic`
-   身份验证：`SYNTHETIC_API_KEY`
-   示例模型：`synthetic/hf:MiniMaxAI/MiniMax-M2.5`
-   CLI：`openclaw onboard --auth-choice synthetic-api-key`

```json
{
  agents: {
    defaults: { model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.5" } },
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [{ id: "hf:MiniMaxAI/MiniMax-M2.5", name: "MiniMax M2.5" }],
      },
    },
  },
}
```

### MiniMax

MiniMax 通过 `models.providers` 配置，因为它使用自定义端点：

-   MiniMax (Anthropic 兼容)：`--auth-choice minimax-api`
-   身份验证：`MINIMAX_API_KEY`

有关设置详情、模型选项和配置片段，请参阅 [/providers/minimax](../providers/minimax.md)。

### Ollama

Ollama 是一个本地 LLM 运行时，提供 OpenAI 兼容的 API：

-   提供商：`ollama`
-   身份验证：无需（本地服务器）
-   示例模型：`ollama/llama3.3`
-   安装：[https://ollama.ai](https://ollama.ai)

```bash
# 安装 Ollama，然后拉取一个模型：
ollama pull llama3.3
```

```json
{
  agents: {
    defaults: { model: { primary: "ollama/llama3.3" } },
  },
}
```

当在本地 `http://127.0.0.1:11434/v1` 运行时，Ollama 会自动检测。有关模型推荐和自定义配置，请参阅 [/providers/ollama](../providers/ollama.md)。

### vLLM

vLLM 是一个本地（或自托管）的 OpenAI 兼容服务器：

-   提供商：`vllm`
-   身份验证：可选（取决于您的服务器）
-   默认基础 URL：`http://127.0.0.1:8000/v1`

要在本地选择自动发现（如果您的服务器不强制执行身份验证，任何值都有效）：

```bash
export VLLM_API_KEY="vllm-local"
```

然后设置一个模型（替换为 `/v1/models` 返回的 ID 之一）：

```json
{
  agents: {
    defaults: { model: { primary: "vllm/your-model-id" } },
  },
}
```

详情请参阅 [/providers/vllm](../providers/vllm.md)。

### 本地代理 (LM Studio, vLLM, LiteLLM 等)

示例 (OpenAI 兼容)：

```json
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.5-gs32" },
      models: { "lmstudio/minimax-m2.5-gs32": { alias: "Minimax" } },
    },
  },
  models: {
    providers: {
      lmstudio: {
        baseUrl: "http://localhost:1234/v1",
        apiKey: "LMSTUDIO_KEY",
        api: "openai-completions",
        models: [
          {
            id: "minimax-m2.5-gs32",
            name: "MiniMax M2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

注意：

-   对于自定义提供商，`reasoning`、`input`、`cost`、`contextWindow` 和 `maxTokens` 是可选的。当省略时，OpenClaw 默认为：
    -   `reasoning: false`
    -   `input: ["text"]`
    -   `cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 }`
    -   `contextWindow: 200000`
    -   `maxTokens: 8192`
-   建议：设置与您的代理/模型限制匹配的明确值。

## CLI 示例

```bash
openclaw onboard --auth-choice opencode-zen
openclaw models set opencode/claude-opus-4-6
openclaw models list
```

另请参阅：[/gateway/configuration](../gateway/configuration.md) 获取完整配置示例。

[模型 CLI](./models.md)[模型故障转移](./model-failover.md)