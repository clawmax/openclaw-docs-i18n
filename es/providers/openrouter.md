title: "Configuración y Configuración del Proveedor OpenRouter para OpenClaw AI"
description: "Aprende a configurar el proveedor OpenRouter en OpenClaw AI. Usa una API unificada para acceder a múltiples modelos de IA con una sola clave API y un endpoint compatible con OpenAI."
keywords: ["openrouter", "openclaw ai", "proveedor api", "api unificada", "enrutamiento de modelos", "compatible con openai", "claude sonnet", "configuración ia"]
---

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