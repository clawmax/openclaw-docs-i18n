title: "Configuración del Proveedor OpenAI para Clave API de OpenClaw y Codex"
description: "Aprende a configurar claves API de OpenAI o OAuth de Codex en OpenClaw. Configura modelos GPT, gestiona el transporte y habilita la compactación del lado del servidor."
keywords: ["openai", "openclaw", "clave api", "codex", "gpt-5.4", "oauth", "websocket", "api de respuestas"]
---

  Proveedores

  
# OpenAI

OpenAI proporciona APIs para desarrolladores de modelos GPT. Codex admite **inicio de sesión con ChatGPT** para acceso por suscripción o **inicio de sesión con clave API** para acceso basado en uso. Codex cloud requiere inicio de sesión con ChatGPT. OpenAI admite explícitamente el uso de OAuth de suscripción en herramientas/flujos de trabajo externos como OpenClaw.

## Opción A: Clave API de OpenAI (Plataforma OpenAI)

**Ideal para:** acceso directo a la API y facturación basada en uso. Obtén tu clave API desde el panel de control de OpenAI.

### Configuración por CLI

```bash
openclaw onboard --auth-choice openai-api-key
# o no interactivo
openclaw onboard --openai-api-key "$OPENAI_API_KEY"
```

### Fragmento de configuración

```json
{
  env: { OPENAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "openai/gpt-5.4" } } },
}
```

La documentación actual de modelos de API de OpenAI lista `gpt-5.4` y `gpt-5.4-pro` para uso directo de la API de OpenAI. OpenClaw los reenvía a través de la ruta de Respuestas `openai/*`.

## Opción B: Suscripción OpenAI Code (Codex)

**Ideal para:** usar el acceso por suscripción a ChatGPT/Codex en lugar de una clave API. Codex cloud requiere inicio de sesión con ChatGPT, mientras que la CLI de Codex admite inicio de sesión con ChatGPT o clave API.

### Configuración por CLI (OAuth de Codex)

```bash
# Ejecuta OAuth de Codex en el asistente
openclaw onboard --auth-choice openai-codex

# O ejecuta OAuth directamente
openclaw models auth login --provider openai-codex
```

### Fragmento de configuración (suscripción a Codex)

```json
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.4" } } },
}
```

La documentación actual de Codex de OpenAI lista `gpt-5.4` como el modelo actual de Codex. OpenClaw lo asigna a `openai-codex/gpt-5.4` para el uso de OAuth de ChatGPT/Codex.

### Transporte por defecto

OpenClaw usa `pi-ai` para el streaming de modelos. Tanto para `openai/*` como para `openai-codex/*`, el transporte por defecto es `"auto"` (primero WebSocket, luego fallback a SSE). Puedes configurar `agents.defaults.models.<provider/model>.params.transport`:

-   `"sse"`: forzar SSE
-   `"websocket"`: forzar WebSocket
-   `"auto"`: intentar WebSocket, luego volver a SSE

Para `openai/*` (API de Respuestas), OpenClaw también habilita el calentamiento de WebSocket por defecto (`openaiWsWarmup: true`) cuando se usa transporte WebSocket. Documentación relacionada de OpenAI:

-   [API en tiempo real con WebSocket](https://platform.openai.com/docs/guides/realtime-websocket)
-   [Respuestas de API en streaming (SSE)](https://platform.openai.com/docs/guides/streaming-responses)

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

### Calentamiento de WebSocket de OpenAI

La documentación de OpenAI describe el calentamiento como opcional. OpenClaw lo habilita por defecto para `openai/*` para reducir la latencia del primer turno cuando se usa transporte WebSocket.

### Deshabilitar el calentamiento

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

### Habilitar el calentamiento explícitamente

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

### Procesamiento prioritario de OpenAI

La API de OpenAI expone el procesamiento prioritario a través de `service_tier=priority`. En OpenClaw, configura `agents.defaults.models["openai/"].params.serviceTier` para pasar ese campo en las solicitudes directas de Respuestas `openai/*`.

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

Los valores admitidos son `auto`, `default`, `flex` y `priority`.

### Compactación del lado del servidor de OpenAI Responses

Para los modelos directos de OpenAI Responses (`openai/*` usando `api: "openai-responses"` con `baseUrl` en `api.openai.com`), OpenClaw ahora habilita automáticamente las sugerencias de carga útil de compactación del lado del servidor de OpenAI:

-   Fuerza `store: true` (a menos que la compatibilidad del modelo establezca `supportsStore: false`)
-   Inyecta `context_management: [{ type: "compaction", compact_threshold: ... }]`

Por defecto, `compact_threshold` es el `70%` del `contextWindow` del modelo (o `80000` cuando no está disponible).

### Habilitar la compactación del lado del servidor explícitamente

Usa esto cuando quieras forzar la inyección de `context_management` en modelos de Respuestas compatibles (por ejemplo, Azure OpenAI Responses):

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

### Habilitar con un umbral personalizado

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

### Deshabilitar la compactación del lado del servidor

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

`responsesServerCompaction` solo controla la inyección de `context_management`. Los modelos directos de OpenAI Responses aún fuerzan `store: true` a menos que la compatibilidad establezca `supportsStore: false`.

## Notas

-   Las referencias de modelo siempre usan `proveedor/modelo` (ver [/concepts/models](../concepts/models.md)).
-   Los detalles de autenticación + reglas de reutilización están en [/concepts/oauth](../concepts/oauth.md).

[Ollama](./ollama.md)[OpenCode Zen](./opencode.md)