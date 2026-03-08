

  Herramientas integradas

  
# Tarea LLM

`llm-task` es una **herramienta de plugin opcional** que ejecuta una tarea LLM que solo devuelve JSON y produce una salida estructurada (opcionalmente validada contra un JSON Schema). Esto es ideal para motores de flujo de trabajo como Lobster: puedes agregar un solo paso LLM sin escribir código OpenClaw personalizado para cada flujo.

## Habilitar el plugin

1.  Habilita el plugin:

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  }
}
```

2.  Agrega la herramienta a la lista de permitidas (se registra con `optional: true`):

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": { "allow": ["llm-task"] }
      }
    ]
  }
}
```

## Configuración (opcional)

```json
{
  "plugins": {
    "entries": {
      "llm-task": {
        "enabled": true,
        "config": {
          "defaultProvider": "openai-codex",
          "defaultModel": "gpt-5.4",
          "defaultAuthProfileId": "main",
          "allowedModels": ["openai-codex/gpt-5.4"],
          "maxTokens": 800,
          "timeoutMs": 30000
        }
      }
    }
  }
}
```

`allowedModels` es una lista de permitidos de cadenas `proveedor/modelo`. Si se establece, cualquier solicitud fuera de la lista será rechazada.

## Parámetros de la herramienta

-   `prompt` (cadena, requerido)
-   `input` (cualquiera, opcional)
-   `schema` (objeto, JSON Schema opcional)
-   `provider` (cadena, opcional)
-   `model` (cadena, opcional)
-   `authProfileId` (cadena, opcional)
-   `temperature` (número, opcional)
-   `maxTokens` (número, opcional)
-   `timeoutMs` (número, opcional)

## Salida

Devuelve `details.json` que contiene el JSON analizado (y valida contra `schema` cuando se proporciona).

## Ejemplo: Paso de flujo de trabajo Lobster

```
openclaw.invoke --tool llm-task --action json --args-json '{
  "prompt": "Given the input email, return intent and draft.",
  "input": {
    "subject": "Hello",
    "body": "Can you help?"
  },
  "schema": {
    "type": "object",
    "properties": {
      "intent": { "type": "string" },
      "draft": { "type": "string" }
    },
    "required": ["intent", "draft"],
    "additionalProperties": false
  }
}'
```

## Notas de seguridad

-   La herramienta es **solo para JSON** e instruye al modelo para que solo genere JSON (sin bloques de código, sin comentarios).
-   No se exponen herramientas al modelo para esta ejecución.
-   Trata la salida como no confiable a menos que la valides con `schema`.
-   Coloca aprobaciones antes de cualquier paso con efectos secundarios (enviar, publicar, ejecutar).

[Firecrawl](./firecrawl.md)[Lobster](./lobster.md)

---