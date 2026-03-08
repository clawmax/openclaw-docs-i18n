

  Providers

  
# GLM Models

GLM is a **model family** (not a company) available through the Z.AI platform. In OpenClaw, GLM models are accessed via the `zai` provider and model IDs like `zai/glm-5`.

## CLI setup

```bash
openclaw onboard --auth-choice zai-api-key
```

## Config snippet

```json
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-5" } } },
}
```

## Notes

-   GLM versions and availability can change; check Z.AI’s docs for the latest.
-   Example model IDs include `glm-5`, `glm-4.7`, and `glm-4.6`.
-   For provider details, see [/providers/zai](./zai.md).

[Litellm](./litellm.md)[MiniMax](./minimax.md)
