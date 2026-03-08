

  Sesiones y memoria

  
# Poda de Sesiones

La poda de sesiones recorta **resultados antiguos de herramientas** del contexto en memoria justo antes de cada llamada al LLM. **No** reescribe el historial de sesiones en disco (`*.jsonl`).

## Cuándo se ejecuta

-   Cuando `mode: "cache-ttl"` está habilitado y la última llamada a Anthropic para la sesión es más antigua que `ttl`.
-   Solo afecta a los mensajes enviados al modelo para esa solicitud.
-   Solo está activa para llamadas a la API de Anthropic (y modelos Anthropic de OpenRouter).
-   Para mejores resultados, iguala `ttl` a tu política `cacheRetention` del modelo (`short` = 5m, `long` = 1h).
-   Después de una poda, la ventana TTL se reinicia, por lo que las solicitudes posteriores mantienen el caché hasta que `ttl` expire nuevamente.

## Valores predeterminados inteligentes (Anthropic)

-   Perfiles con **OAuth o setup-token**: habilitan la poda `cache-ttl` y configuran el heartbeat a `1h`.
-   Perfiles con **clave API**: habilitan la poda `cache-ttl`, configuran el heartbeat a `30m` y `cacheRetention: "short"` por defecto en modelos Anthropic.
-   Si configuras cualquiera de estos valores explícitamente, OpenClaw **no** los sobrescribe.

## Qué mejora esto (costo + comportamiento del caché)

-   **Por qué podar:** El caché de prompts de Anthropic solo se aplica dentro del TTL. Si una sesión queda inactiva más allá del TTL, la siguiente solicitud vuelve a cachear el prompt completo a menos que lo recortes primero.
-   **Qué se abarata:** la poda reduce el tamaño de **cacheWrite** para esa primera solicitud después de que expire el TTL.
-   **Por qué importa el reinicio del TTL:** una vez que se ejecuta la poda, la ventana de caché se reinicia, por lo que las solicitudes de seguimiento pueden reutilizar el prompt recién cacheado en lugar de volver a cachear todo el historial.
-   **Qué no hace:** la poda no añade tokens ni "duplica" costos; solo cambia lo que se cachea en esa primera solicitud posterior al TTL.

## Qué se puede podar

-   Solo mensajes de tipo `toolResult`.
-   Los mensajes de usuario y asistente **nunca** se modifican.
-   Los últimos `keepLastAssistants` mensajes del asistente están protegidos; los resultados de herramientas posteriores a ese límite no se podan.
-   Si no hay suficientes mensajes del asistente para establecer el límite, se omite la poda.
-   Los resultados de herramientas que contienen **bloques de imagen** se omiten (nunca se recortan/borran).

## Estimación de la ventana de contexto

La poda utiliza una ventana de contexto estimada (caracteres ≈ tokens × 4). La ventana base se resuelve en este orden:

1.  Anulación `models.providers.*.models[].contextWindow`.
2.  `contextWindow` de la definición del modelo (del registro de modelos).
3.  Valor predeterminado de `200000` tokens.

Si `agents.defaults.contextTokens` está configurado, se trata como un límite (mínimo) en la ventana resuelta.

## Modo

### cache-ttl

-   La poda solo se ejecuta si la última llamada a Anthropic es más antigua que `ttl` (predeterminado `5m`).
-   Cuando se ejecuta: mismo comportamiento de recorte suave + borrado duro que antes.

## Poda suave vs dura

-   **Recorte suave**: solo para resultados de herramientas de gran tamaño.
    -   Mantiene el principio y el final, inserta `...`, y añade una nota con el tamaño original.
    -   Omite resultados con bloques de imagen.
-   **Borrado duro**: reemplaza todo el resultado de la herramienta con `hardClear.placeholder`.

## Selección de herramientas

-   `tools.allow` / `tools.deny` admiten comodines `*`.
-   Deny tiene prioridad.
-   La coincidencia no distingue entre mayúsculas y minúsculas.
-   Lista de allow vacía => todas las herramientas permitidas.

## Interacción con otros límites

-   Las herramientas integradas ya truncan su propia salida; la poda de sesiones es una capa adicional que evita que los chats de larga duración acumulen demasiada salida de herramientas en el contexto del modelo.
-   La compactación es independiente: la compactación resume y persiste, la poda es transitoria por solicitud. Ver [/concepts/compaction](./compaction.md).

## Valores predeterminados (cuando está habilitado)

-   `ttl`: `"5m"`
-   `keepLastAssistants`: `3`
-   `softTrimRatio`: `0.3`
-   `hardClearRatio`: `0.5`
-   `minPrunableToolChars`: `50000`
-   `softTrim`: `{ maxChars: 4000, headChars: 1500, tailChars: 1500 }`
-   `hardClear`: `{ enabled: true, placeholder: "[Contenido antiguo de resultado de herramienta borrado]" }`

## Ejemplos

Predeterminado (desactivado):

```json
{
  agents: { defaults: { contextPruning: { mode: "off" } } },
}
```

Habilitar poda consciente del TTL:

```json
{
  agents: { defaults: { contextPruning: { mode: "cache-ttl", ttl: "5m" } } },
}
```

Restringir la poda a herramientas específicas:

```json
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "cache-ttl",
        tools: { allow: ["exec", "read"], deny: ["*image*"] },
      },
    },
  },
}
```

Ver referencia de configuración: [Configuración de Gateway](../gateway/configuration.md)

[Gestión de Sesiones](./session.md)[Herramientas de Sesión](./session-tool.md)