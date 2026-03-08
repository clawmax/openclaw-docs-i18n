

  Overview

  
# Model Provider Quickstart

OpenClaw can use many LLM providers. Pick one, authenticate, then set the default model as `provider/model`.

## Quick start (two steps)

1.  Authenticate with the provider (usually via `openclaw onboard`).
2.  Set the default model:

```json
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## Supported providers (starter set)

-   [OpenAI (API + Codex)](./openai.md)
-   [Anthropic (API + Claude Code CLI)](./anthropic.md)
-   [OpenRouter](./openrouter.md)
-   [Vercel AI Gateway](./vercel-ai-gateway.md)
-   [Cloudflare AI Gateway](./cloudflare-ai-gateway.md)
-   [Moonshot AI (Kimi + Kimi Coding)](./moonshot.md)
-   [Mistral](./mistral.md)
-   [Synthetic](./synthetic.md)
-   [OpenCode Zen](./opencode.md)
-   [Z.AI](./zai.md)
-   [GLM models](./glm.md)
-   [MiniMax](./minimax.md)
-   [Venice (Venice AI)](./venice.md)
-   [Amazon Bedrock](./bedrock.md)
-   [Qianfan](./qianfan.md)

For the full provider catalog (xAI, Groq, Mistral, etc.) and advanced configuration, see [Model providers](../concepts/model-providers.md).

[Model Providers](../providers.md)[Models CLI](../concepts/models.md)
