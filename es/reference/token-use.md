

  Referencia técnica

  
# Uso y costos de tokens

OpenClaw rastrea **tokens**, no caracteres. Los tokens son específicos del modelo, pero la mayoría de los modelos estilo OpenAI promedian ~4 caracteres por token para texto en inglés.

## Cómo se construye el prompt del sistema

OpenClaw ensambla su propio prompt del sistema en cada ejecución. Incluye:

-   Lista de herramientas + descripciones cortas
-   Lista de habilidades (solo metadatos; las instrucciones se cargan bajo demanda con `read`)
-   Instrucciones de auto-actualización
-   Espacio de trabajo + archivos de arranque (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md` cuando son nuevos, más `MEMORY.md` y/o `memory.md` cuando están presentes). Los archivos grandes se truncan por `agents.defaults.bootstrapMaxChars` (por defecto: 20000), y la inyección total de arranque está limitada por `agents.defaults.bootstrapTotalMaxChars` (por defecto: 150000). Los archivos `memory/*.md` son bajo demanda a través de herramientas de memoria y no se inyectan automáticamente.
-   Hora (UTC + zona horaria del usuario)
-   Etiquetas de respuesta + comportamiento de latido
-   Metadatos de tiempo de ejecución (host/SO/modelo/pensamiento)

Consulta el desglose completo en [Prompt del Sistema](../concepts/system-prompt.md).

## Qué cuenta en la ventana de contexto

Todo lo que el modelo recibe cuenta para el límite de contexto:

-   Prompt del sistema (todas las secciones listadas arriba)
-   Historial de conversación (mensajes de usuario + asistente)
-   Llamadas a herramientas y resultados de herramientas
-   Adjuntos/transcripciones (imágenes, audio, archivos)
-   Resúmenes de compactación y artefactos de poda
-   Envolturas del proveedor o encabezados de seguridad (no visibles, pero aún contados)

Para imágenes, OpenClaw reduce la escala de las cargas útiles de imagen de transcripción/herramienta antes de las llamadas al proveedor. Usa `agents.defaults.imageMaxDimensionPx` (por defecto: `1200`) para ajustar esto:

-   Valores más bajos generalmente reducen el uso de tokens de visión y el tamaño de la carga útil.
-   Valores más altos preservan más detalle visual para capturas de pantalla con mucho OCR/UI.

Para un desglose práctico (por archivo inyectado, herramientas, habilidades y tamaño del prompt del sistema), usa `/context list` o `/context detail`. Consulta [Contexto](../concepts/context.md).

## Cómo ver el uso actual de tokens

Usa estos comandos en el chat:

-   `/status` → **tarjeta de estado rica en emojis** con el modelo de la sesión, uso del contexto, tokens de entrada/salida de la última respuesta y **costo estimado** (solo clave API).
-   `/usage off|tokens|full` → agrega un **pie de página de uso por respuesta** a cada réplica.
    -   Persiste por sesión (almacenado como `responseUsage`).
    -   La autenticación OAuth **oculta el costo** (solo tokens).
-   `/usage cost` → muestra un resumen de costos local desde los registros de sesión de OpenClaw.

Otras superficies:

-   **TUI/Web TUI:** `/status` + `/usage` son compatibles.
-   **CLI:** `openclaw status --usage` y `openclaw channels list` muestran ventanas de cuota del proveedor (no costos por respuesta).

## Estimación de costos (cuando se muestra)

Los costos se estiman desde tu configuración de precios del modelo:

```
models.providers.<provider>.models[].cost
```

Estos son **USD por 1M de tokens** para `input`, `output`, `cacheRead` y `cacheWrite`. Si faltan los precios, OpenClaw solo muestra tokens. Los tokens OAuth nunca muestran costo en dólares.

## TTL de caché e impacto de la poda

El almacenamiento en caché de prompts del proveedor solo se aplica dentro de la ventana de TTL de caché. OpenClaw puede ejecutar opcionalmente **poda por TTL de caché**: poda la sesión una vez que ha expirado el TTL de caché, luego reinicia la ventana de caché para que las solicitudes posteriores puedan reutilizar el contexto recién almacenado en caché en lugar de volver a almacenar en caché el historial completo. Esto mantiene los costos de escritura de caché más bajos cuando una sesión permanece inactiva más allá del TTL. Configúralo en [Configuración de Gateway](../gateway/configuration.md) y consulta los detalles del comportamiento en [Poda de sesión](../concepts/session-pruning.md). El latido puede mantener la caché **caliente** a través de intervalos de inactividad. Si el TTL de caché de tu modelo es `1h`, establecer el intervalo de latido justo por debajo de eso (por ejemplo, `55m`) puede evitar volver a almacenar en caché el prompt completo, reduciendo los costos de escritura de caché. En configuraciones multi-agente, puedes mantener una configuración de modelo compartida y ajustar el comportamiento de caché por agente con `agents.list[].params.cacheRetention`. Para una guía completa parámetro por parámetro, consulta [Almacenamiento en caché de prompts](./prompt-caching.md). Para los precios de la API de Anthropic, las lecturas de caché son significativamente más baratas que los tokens de entrada, mientras que las escrituras de caché se facturan con un multiplicador más alto. Consulta los precios de almacenamiento en caché de prompts de Anthropic para conocer las tarifas más recientes y los multiplicadores de TTL: [https://docs.anthropic.com/docs/build-with-claude/prompt-caching](https://docs.anthropic.com/docs/build-with-claude/prompt-caching)

### Ejemplo: mantener caché de 1h caliente con latido

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-6"
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "long"
    heartbeat:
      every: "55m"
```

### Ejemplo: tráfico mixto con estrategia de caché por agente

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-6"
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "long" # línea base por defecto para la mayoría de agentes
  list:
    - id: "research"
      default: true
      heartbeat:
        every: "55m" # mantener caché larga caliente para sesiones profundas
    - id: "alerts"
      params:
        cacheRetention: "none" # evitar escrituras de caché para notificaciones explosivas
```

`agents.list[].params` se fusiona sobre los `params` del modelo seleccionado, por lo que puedes anular solo `cacheRetention` y heredar otros valores por defecto del modelo sin cambios.

### Ejemplo: habilitar encabezado beta de contexto 1M de Anthropic

La ventana de contexto de 1M de Anthropic está actualmente en fase beta. OpenClaw puede inyectar el valor requerido `anthropic-beta` cuando habilitas `context1m` en modelos Opus o Sonnet compatibles.

```yaml
agents:
  defaults:
    models:
      "anthropic/claude-opus-4-6":
        params:
          context1m: true
```

Esto se asigna al encabezado beta `context-1m-2025-08-07` de Anthropic. Esto solo se aplica cuando `context1m: true` está configurado en esa entrada del modelo. Requisito: la credencial debe ser elegible para uso de contexto largo (facturación por clave API, o suscripción con Uso Extra habilitado). Si no, Anthropic responde con `HTTP 429: rate_limit_error: Extra usage is required for long context requests`. Si autenticas Anthropic con tokens OAuth/suscripción (`sk-ant-oat-*`), OpenClaw omite el encabezado beta `context-1m-*` porque Anthropic actualmente rechaza esa combinación con HTTP 401.

## Consejos para reducir la presión de tokens

-   Usa `/compact` para resumir sesiones largas.
-   Recorta salidas grandes de herramientas en tus flujos de trabajo.
-   Reduce `agents.defaults.imageMaxDimensionPx` para sesiones con muchas capturas de pantalla.
-   Mantén las descripciones de habilidades cortas (la lista de habilidades se inyecta en el prompt).
-   Prefiere modelos más pequeños para trabajos exploratorios y verbosos.

Consulta [Habilidades](../tools/skills.md) para la fórmula exacta de sobrecarga de la lista de habilidades.

[Referencia del Asistente](./wizard.md)[Superficie de Credencial SecretRef](./secretref-credential-surface.md)