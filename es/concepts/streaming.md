

  Mensajes y entrega

  
# Streaming y Fragmentación

OpenClaw tiene dos capas de streaming separadas:

-   **Streaming por bloques (canales):** emite **bloques** completos mientras el asistente escribe. Estos son mensajes normales del canal (no deltas de tokens).
-   **Streaming de vista previa (Telegram/Discord/Slack):** actualiza un **mensaje de vista previa** temporal mientras se genera.

Hoy en día **no hay streaming verdadero de deltas de tokens** a los mensajes del canal. El streaming de vista previa está basado en mensajes (envío + ediciones/agregados).

## Streaming por bloques (mensajes del canal)

El streaming por bloques envía la salida del asistente en fragmentos gruesos a medida que está disponible.

```
Model output
  └─ text_delta/events
       ├─ (blockStreamingBreak=text_end)
       │    └─ chunker emits blocks as buffer grows
       └─ (blockStreamingBreak=message_end)
            └─ chunker flushes at message_end
                   └─ channel send (block replies)
```

Leyenda:

-   `text_delta/events`: eventos del flujo del modelo (pueden ser dispersos para modelos que no hacen streaming).
-   `chunker`: `EmbeddedBlockChunker` aplicando límites mín/máx + preferencia de ruptura.
-   `channel send`: mensajes salientes reales (respuestas de bloque).

**Controles:**

-   `agents.defaults.blockStreamingDefault`: `"on"`/`"off"` (por defecto off).
-   Anulaciones por canal: `*.blockStreaming` (y variantes por cuenta) para forzar `"on"`/`"off"` por canal.
-   `agents.defaults.blockStreamingBreak`: `"text_end"` o `"message_end"`.
-   `agents.defaults.blockStreamingChunk`: `{ minChars, maxChars, breakPreference? }`.
-   `agents.defaults.blockStreamingCoalesce`: `{ minChars?, maxChars?, idleMs? }` (fusiona bloques transmitidos antes del envío).
-   Límite máximo del canal: `*.textChunkLimit` (ej., `channels.whatsapp.textChunkLimit`).
-   Modo de fragmentación del canal: `*.chunkMode` (`length` por defecto, `newline` divide en líneas en blanco (límites de párrafo) antes de la fragmentación por longitud).
-   Límite flexible de Discord: `channels.discord.maxLinesPerMessage` (por defecto 17) divide respuestas largas para evitar recorte en la UI.

**Semántica de límites:**

-   `text_end`: transmite bloques tan pronto como el fragmentador los emite; vacía en cada `text_end`.
-   `message_end`: espera hasta que el mensaje del asistente termine, luego vacía la salida almacenada en búfer.

`message_end` aún usa el fragmentador si el texto en búfer excede `maxChars`, por lo que puede emitir múltiples fragmentos al final.

## Algoritmo de fragmentación (límites bajo/alto)

La fragmentación de bloques está implementada por `EmbeddedBlockChunker`:

-   **Límite bajo:** no emitir hasta que el búfer >= `minChars` (a menos que sea forzado).
-   **Límite alto:** preferir divisiones antes de `maxChars`; si es forzado, dividir en `maxChars`.
-   **Preferencia de ruptura:** `paragraph` → `newline` → `sentence` → `whitespace` → ruptura dura.
-   **Cercas de código:** nunca dividir dentro de cercas; cuando se fuerza en `maxChars`, cerrar + reabrir la cerca para mantener el Markdown válido.

`maxChars` está limitado por el `textChunkLimit` del canal, por lo que no puedes exceder los límites por canal.

## Coalescencia (fusionar bloques transmitidos)

Cuando el streaming por bloques está habilitado, OpenClaw puede **fusionar fragmentos de bloques consecutivos** antes de enviarlos. Esto reduce el "spam de una sola línea" mientras aún proporciona salida progresiva.

-   La coalescencia espera por **brechas de inactividad** (`idleMs`) antes de vaciar.
-   Los búferes están limitados por `maxChars` y se vaciarán si lo exceden.
-   `minChars` evita que se envíen fragmentos pequeños hasta que se acumule suficiente texto (el vaciado final siempre envía el texto restante).
-   El conector se deriva de `blockStreamingChunk.breakPreference` (`paragraph` → `\n\n`, `newline` → `\n`, `sentence` → espacio).
-   Las anulaciones por canal están disponibles mediante `*.blockStreamingCoalesce` (incluyendo configuraciones por cuenta).
-   El `minChars` de coalescencia por defecto se aumenta a 1500 para Signal/Slack/Discord a menos que se anule.

## Ritmo similar al humano entre bloques

Cuando el streaming por bloques está habilitado, puedes agregar una **pausa aleatorizada** entre las respuestas de bloque (después del primer bloque). Esto hace que las respuestas de múltiples burbujas se sientan más naturales.

-   Configuración: `agents.defaults.humanDelay` (anular por agente mediante `agents.list[].humanDelay`).
-   Modos: `off` (por defecto), `natural` (800–2500ms), `custom` (`minMs`/`maxMs`).
-   Se aplica solo a **respuestas de bloque**, no a respuestas finales o resúmenes de herramientas.

## "Transmitir fragmentos o todo"

Esto se traduce a:

-   **Transmitir fragmentos:** `blockStreamingDefault: "on"` + `blockStreamingBreak: "text_end"` (emitir sobre la marcha). Los canales que no son Telegram también necesitan `*.blockStreaming: true`.
-   **Transmitir todo al final:** `blockStreamingBreak: "message_end"` (vaciar una vez, posiblemente múltiples fragmentos si es muy largo).
-   **Sin streaming por bloques:** `blockStreamingDefault: "off"` (solo respuesta final).

**Nota del canal:** El streaming por bloques está **apagado a menos que** `*.blockStreaming` esté explícitamente configurado en `true`. Los canales pueden transmitir una vista previa en vivo (`channels..streaming`) sin respuestas de bloque. Recordatorio de ubicación de configuración: los valores por defecto de `blockStreaming*` viven bajo `agents.defaults`, no en la configuración raíz.

## Modos de streaming de vista previa

Clave canónica: `channels..streaming` Modos:

-   `off`: deshabilitar el streaming de vista previa.
-   `partial`: vista previa única que se reemplaza con el texto más reciente.
-   `block`: la vista previa se actualiza en pasos fragmentados/agregados.
-   `progress`: vista previa de progreso/estado durante la generación, respuesta final al completar.

### Mapeo por canal

| Canal | `off` | `partial` | `block` | `progress` |
| --- | --- | --- | --- | --- |
| Telegram | ✅ | ✅ | ✅ | se asigna a `partial` |
| Discord | ✅ | ✅ | ✅ | se asigna a `partial` |
| Slack | ✅ | ✅ | ✅ | ✅ |

Exclusivo de Slack:

-   `channels.slack.nativeStreaming` activa/desactiva las llamadas API nativas de streaming de Slack cuando `streaming=partial` (por defecto: `true`).

Migración de claves heredadas:

-   Telegram: `streamMode` + booleano `streaming` migran automáticamente al enum `streaming`.
-   Discord: `streamMode` + booleano `streaming` migran automáticamente al enum `streaming`.
-   Slack: `streamMode` migra automáticamente al enum `streaming`; el booleano `streaming` migra automáticamente a `nativeStreaming`.

### Comportamiento en tiempo de ejecución

Telegram:

-   Usa la API de Bot `sendMessageDraft` en MD cuando está disponible, y `sendMessage` + `editMessageText` para actualizaciones de vista previa en grupo/tema.
-   El streaming de vista previa se omite cuando el streaming por bloques de Telegram está explícitamente habilitado (para evitar doble streaming).
-   `/reasoning stream` puede escribir razonamiento en la vista previa.

Discord:

-   Usa envío + edición de mensajes de vista previa.
-   El modo `block` usa fragmentación de borrador (`draftChunk`).
-   El streaming de vista previa se omite cuando el streaming por bloques de Discord está explícitamente habilitado.

Slack:

-   `partial` puede usar el streaming nativo de Slack (`chat.startStream`/`append`/`stop`) cuando está disponible.
-   `block` usa vistas previas de borrador estilo agregado.
-   `progress` usa texto de vista previa de estado, luego respuesta final.

[Mensajes](./messages.md)[Política de Reintento](./retry.md)