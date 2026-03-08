

  Referencia técnica

  
# Almacenamiento en caché de prompts

El almacenamiento en caché de prompts significa que el proveedor del modelo puede reutilizar prefijos de prompts sin cambios (generalmente instrucciones del sistema/desarrollador y otro contexto estable) entre turnos en lugar de reprocesarlos cada vez. La primera solicitud coincidente escribe tokens en la caché (`cacheWrite`), y las solicitudes coincidentes posteriores pueden leerlos (`cacheRead`). Por qué es importante: menor coste de tokens, respuestas más rápidas y un rendimiento más predecible para sesiones de larga duración. Sin caché, los prompts repetidos pagan el coste completo del prompt en cada turno incluso cuando la mayor parte de la entrada no cambió. Esta página cubre todos los controles relacionados con la caché que afectan la reutilización de prompts y el coste de tokens. Para detalles de precios de Anthropic, consulta: [https://docs.anthropic.com/docs/build-with-claude/prompt-caching](https://docs.anthropic.com/docs/build-with-claude/prompt-caching)

## Controles principales

### cacheRetention (modelo y por agente)

Establece la retención de caché en los parámetros del modelo:

```yaml
agents:
  defaults:
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "short" # none | short | long
```

Anulación por agente:

```yaml
agents:
  list:
    - id: "alerts"
      params:
        cacheRetention: "none"
```

Orden de fusión de configuración:

1.  `agents.defaults.models["provider/model"].params`
2.  `agents.list[].params` (coincide con el id del agente; anula por clave)

### cacheControlTtl heredado

Los valores heredados aún se aceptan y se mapean:

-   `5m` -> `short`
-   `1h` -> `long`

Prefiere `cacheRetention` para configuraciones nuevas.

### contextPruning.mode: "cache-ttl"

Poda el contexto de resultados de herramientas antiguos después de los períodos de vida útil (TTL) de la caché, para que las solicitudes posteriores a periodos de inactividad no vuelvan a almacenar en caché un historial de gran tamaño.

```yaml
agents:
  defaults:
    contextPruning:
      mode: "cache-ttl"
      ttl: "1h"
```

Consulta [Poda de sesión](../concepts/session-pruning.md) para el comportamiento completo.

### Latido para mantener caliente

El latido puede mantener las ventanas de caché activas y reducir las escrituras repetidas en caché después de brechas de inactividad.

```yaml
agents:
  defaults:
    heartbeat:
      every: "55m"
```

El latido por agente es compatible en `agents.list[].heartbeat`.

## Comportamiento del proveedor

### Anthropic (API directa)

-   `cacheRetention` es compatible.
-   Con perfiles de autenticación de clave API de Anthropic, OpenClaw establece `cacheRetention: "short"` para las referencias de modelos de Anthropic cuando no está configurado.

### Amazon Bedrock

-   Las referencias de modelos Anthropic Claude (`amazon-bedrock/*anthropic.claude*`) admiten el paso explícito de `cacheRetention`.
-   Los modelos Bedrock no-Anthropic se fuerzan a `cacheRetention: "none"` en tiempo de ejecución.

### Modelos Anthropic de OpenRouter

Para las referencias de modelos `openrouter/anthropic/*`, OpenClaw inyecta `cache_control` de Anthropic en los bloques de prompts del sistema/desarrollador para mejorar la reutilización de la caché de prompts.

### Otros proveedores

Si el proveedor no admite este modo de caché, `cacheRetention` no tiene efecto.

## Patrones de ajuste

### Tráfico mixto (valor predeterminado recomendado)

Mantén una línea base de larga duración en tu agente principal, desactiva el almacenamiento en caché en agentes notificadores con ráfagas:

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-6"
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "long"
  list:
    - id: "research"
      default: true
      heartbeat:
        every: "55m"
    - id: "alerts"
      params:
        cacheRetention: "none"
```

### Línea base priorizando costes

-   Establece la línea base `cacheRetention: "short"`.
-   Habilita `contextPruning.mode: "cache-ttl"`.
-   Mantén el latido por debajo de tu TTL solo para los agentes que se beneficien de cachés activas.

## Diagnóstico de caché

OpenClaw expone diagnósticos de seguimiento de caché dedicados para ejecuciones de agentes integrados.

### Configuración de diagnostics.cacheTrace

```yaml
diagnostics:
  cacheTrace:
    enabled: true
    filePath: "~/.openclaw/logs/cache-trace.jsonl" # opcional
    includeMessages: false # predeterminado true
    includePrompt: false # predeterminado true
    includeSystem: false # predeterminado true
```

Valores predeterminados:

-   `filePath`: `$OPENCLAW_STATE_DIR/logs/cache-trace.jsonl`
-   `includeMessages`: `true`
-   `includePrompt`: `true`
-   `includeSystem`: `true`

### Alternancias de entorno (depuración puntual)

-   `OPENCLAW_CACHE_TRACE=1` habilita el seguimiento de caché.
-   `OPENCLAW_CACHE_TRACE_FILE=/path/to/cache-trace.jsonl` anula la ruta de salida.
-   `OPENCLAW_CACHE_TRACE_MESSAGES=0|1` alterna la captura completa de la carga útil de mensajes.
-   `OPENCLAW_CACHE_TRACE_PROMPT=0|1` alterna la captura del texto del prompt.
-   `OPENCLAW_CACHE_TRACE_SYSTEM=0|1` alterna la captura del prompt del sistema.

### Qué inspeccionar

-   Los eventos de seguimiento de caché están en JSONL e incluyen instantáneas por etapas como `session:loaded`, `prompt:before`, `stream:context` y `session:after`.
-   El impacto de tokens de caché por turno es visible en las superficies de uso normal a través de `cacheRead` y `cacheWrite` (por ejemplo, `/usage full` y resúmenes de uso de sesión).

## Solución rápida de problemas

-   Alto `cacheWrite` en la mayoría de los turnos: verifica las entradas volátiles del prompt del sistema y confirma que el modelo/proveedor admite tus ajustes de caché.
-   Sin efecto de `cacheRetention`: confirma que la clave del modelo coincide con `agents.defaults.models["provider/model"]`.
-   Solicitudes de Bedrock Nova/Mistral con ajustes de caché: se espera que en tiempo de ejecución se fuerce a `none`.

Documentación relacionada:

-   [Anthropic](../providers/anthropic.md)
-   [Uso y costes de tokens](./token-use.md)
-   [Poda de sesión](../concepts/session-pruning.md)
-   [Referencia de configuración de Gateway](../gateway/configuration-reference.md)

[Superficie de credenciales SecretRef](./secretref-credential-surface.md)[Uso y costes de la API](./api-usage-costs.md)