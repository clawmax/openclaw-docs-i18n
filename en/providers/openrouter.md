

  Providers

  
# OpenRouter

OpenRouter provides a **unified API** that routes requests to many models behind a single endpoint and API key. It is OpenAI-compatible, so most OpenAI SDKs work by switching the base URL.

## CLI setup

```bash
openclaw onboard --auth-choice apiKey --token-provider openrouter --token "$OPENROUTER_API_KEY"
```

## Config snippet

```json
{
  env: { OPENROUTER_API_KEY: "sk-or-..." },
  agents: {
    defaults: {
      model: { primary: "openrouter/anthropic/claude-sonnet-4-5" },
    },
  },
}
```

## Notes

-   Model refs are `openrouter//`.
-   For more model/provider options, see [/concepts/model-providers](../concepts/model-providers.md).
-   OpenRouter uses a Bearer token with your API key under the hood.

[OpenCode Zen](./opencode.md)[Qianfan](./qianfan.md)
