

  Proveedores

  
# OpenRouter

OpenRouter proporciona una **API unificada** que enruta las solicitudes a muchos modelos detrás de un único endpoint y clave API. Es compatible con OpenAI, por lo que la mayoría de los SDKs de OpenAI funcionan cambiando la URL base.

## Configuración por CLI

```bash
openclaw onboard --auth-choice apiKey --token-provider openrouter --token "$OPENROUTER_API_KEY"
```

## Fragmento de configuración

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

## Notas

-   Las referencias de modelo son `openrouter//`.
-   Para más opciones de modelos/proveedores, consulta [/concepts/model-providers](../concepts/model-providers.md).
-   OpenRouter utiliza un token Bearer con tu clave API internamente.

[OpenCode Zen](./opencode.md)[Qianfan](./qianfan.md)

---