

  Providers

  
# Z.AI

Z.AI is the API platform for **GLM** models. It provides REST APIs for GLM and uses API keys for authentication. Create your API key in the Z.AI console. OpenClaw uses the `zai` provider with a Z.AI API key.

## CLI setup

```bash
openclaw onboard --auth-choice zai-api-key
# or non-interactive
openclaw onboard --zai-api-key "$ZAI_API_KEY"
```

## Config snippet

```json
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-5" } } },
}
```

## Notes

-   GLM models are available as `zai/` (example: `zai/glm-5`).
-   `tool_stream` is enabled by default for Z.AI tool-call streaming. Set `agents.defaults.models["zai/"].params.tool_stream` to `false` to disable it.
-   See [/providers/glm](./glm.md) for the model family overview.
-   Z.AI uses Bearer auth with your API key.

[Xiaomi MiMo](./xiaomi.md)
