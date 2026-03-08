

  Overview

  
# Model Providers

OpenClaw can use many LLM providers. Pick a provider, authenticate, then set the default model as `provider/model`. Looking for chat channel docs (WhatsApp/Telegram/Discord/Slack/Mattermost (plugin)/etc.)? See [Channels](./channels.md).

## Quick start

1.  Authenticate with the provider (usually via `openclaw onboard`).
2.  Set the default model:

```json
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## Provider docs

-   [Amazon Bedrock](./providers/bedrock.md)
-   [Anthropic (API + Claude Code CLI)](./providers/anthropic.md)
-   [Cloudflare AI Gateway](./providers/cloudflare-ai-gateway.md)
-   [GLM models](./providers/glm.md)
-   [Hugging Face (Inference)](./providers/huggingface.md)
-   [Kilocode](./providers/kilocode.md)
-   [LiteLLM (unified gateway)](./providers/litellm.md)
-   [MiniMax](./providers/minimax.md)
-   [Mistral](./providers/mistral.md)
-   [Moonshot AI (Kimi + Kimi Coding)](./providers/moonshot.md)
-   [NVIDIA](./providers/nvidia.md)
-   [Ollama (local models)](./providers/ollama.md)
-   [OpenAI (API + Codex)](./providers/openai.md)
-   [OpenCode Zen](./providers/opencode.md)
-   [OpenRouter](./providers/openrouter.md)
-   [Qianfan](./providers/qianfan.md)
-   [Qwen (OAuth)](./providers/qwen.md)
-   [Together AI](./providers/together.md)
-   [Vercel AI Gateway](./providers/vercel-ai-gateway.md)
-   [Venice (Venice AI, privacy-focused)](./providers/venice.md)
-   [vLLM (local models)](./providers/vllm.md)
-   [Xiaomi](./providers/xiaomi.md)
-   [Z.AI](./providers/zai.md)

## Transcription providers

-   [Deepgram (audio transcription)](./providers/deepgram.md)

## Community tools

-   [Claude Max API Proxy](./providers/claude-max-api-proxy.md) - Community proxy for Claude subscription credentials (verify Anthropic policy/terms before use)

For the full provider catalog (xAI, Groq, Mistral, etc.) and advanced configuration, see [Model providers](./concepts/model-providers.md).

[Model Provider Quickstart](./providers/models.md)
