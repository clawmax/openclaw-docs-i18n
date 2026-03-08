title: "Configuración y configuración del proveedor Xiaomi MiMo para OpenClaw AI"
description: "Aprende a configurar el proveedor Xiaomi MiMo en OpenClaw AI. Usa el modelo mimo-v2-flash con APIs compatibles con Anthropic mediante autenticación por clave API."
keywords: ["xiaomi mimo", "proveedor openclaw", "mimo-v2-flash", "api anthropic", "autenticación por clave api", "configuración de modelo", "configuración de openclaw", "api xiaomi"]
---

  Proveedores

  
# Xiaomi MiMo

Xiaomi MiMo es la plataforma API para los modelos **MiMo**. Proporciona APIs REST compatibles con los formatos de OpenAI y Anthropic y utiliza claves API para la autenticación. Crea tu clave API en la [consola de Xiaomi MiMo](https://platform.xiaomimimo.com/#/console/api-keys). OpenClaw utiliza el proveedor `xiaomi` con una clave API de Xiaomi MiMo.

## Resumen del modelo

-   **mimo-v2-flash**: Ventana de contexto de 262144 tokens, compatible con la API de Mensajes de Anthropic.
-   URL base: `https://api.xiaomimimo.com/anthropic`
-   Autorización: `Bearer $XIAOMI_API_KEY`

## Configuración por CLI

```bash
openclaw onboard --auth-choice xiaomi-api-key
# o no interactivo
openclaw onboard --auth-choice xiaomi-api-key --xiaomi-api-key "$XIAOMI_API_KEY"
```

## Fragmento de configuración

```json
{
  env: { XIAOMI_API_KEY: "your-key" },
  agents: { defaults: { model: { primary: "xiaomi/mimo-v2-flash" } } },
  models: {
    mode: "merge",
    providers: {
      xiaomi: {
        baseUrl: "https://api.xiaomimimo.com/anthropic",
        api: "anthropic-messages",
        apiKey: "XIAOMI_API_KEY",
        models: [
          {
            id: "mimo-v2-flash",
            name: "Xiaomi MiMo V2 Flash",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 262144,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## Notas

-   Referencia del modelo: `xiaomi/mimo-v2-flash`.
-   El proveedor se inyecta automáticamente cuando `XIAOMI_API_KEY` está configurada (o existe un perfil de autenticación).
-   Consulta [/concepts/model-providers](../concepts/model-providers.md) para las reglas de los proveedores.

[vLLM](./vllm.md)[Z.AI](./zai.md)