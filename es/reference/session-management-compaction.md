

  Internos de compactación

  
# Análisis Profundo de la Gestión de Sesiones

Este documento explica cómo OpenClaw gestiona las sesiones de principio a fin:

-   **Enrutamiento de sesiones** (cómo los mensajes entrantes se asignan a una `sessionKey`)
-   **Almacén de sesiones** (`sessions.json`) y qué rastrea
-   **Persistencia de transcripción** (`*.jsonl`) y su estructura
-   **Higiene de transcripción** (correcciones específicas del proveedor antes de las ejecuciones)
-   **Límites de contexto** (ventana de contexto vs tokens rastreados)
-   **Compactación** (manual + compactación automática) y dónde conectar el trabajo pre-compactación
-   **Mantenimiento silencioso** (ej. escrituras de memoria que no deben producir salida visible para el usuario)

Si quieres una descripción general de alto nivel primero, comienza con:

-   [/concepts/session](../concepts/session.md)
-   [/concepts/compaction](../concepts/compaction.md)
-   [/concepts/session-pruning](../concepts/session-pruning.md)
-   [/reference/transcript-hygiene](./transcript-hygiene.md)

* * *

## Fuente de verdad: el Gateway

OpenClaw está diseñado alrededor de un único **proceso Gateway** que posee el estado de la sesión.

-   Las UIs (aplicación macOS, web Control UI, TUI) deben consultar al Gateway para obtener listas de sesiones y recuentos de tokens.
-   En modo remoto, los archivos de sesión están en el host remoto; "revisar tus archivos locales de Mac" no reflejará lo que el Gateway está usando.

* * *

## Dos capas de persistencia

OpenClaw persiste las sesiones en dos capas:

1.  **Almacén de sesiones (`sessions.json`)**
    -   Mapa clave/valor: `sessionKey -> SessionEntry`
    -   Pequeño, mutable, seguro de editar (o eliminar entradas)
    -   Rastrea metadatos de la sesión (id de sesión actual, última actividad, interruptores, contadores de tokens, etc.)
2.  **Transcripción (`.jsonl`)**
    -   Transcripción de solo anexar con estructura de árbol (las entradas tienen `id` + `parentId`)
    -   Almacena la conversación real + llamadas a herramientas + resúmenes de compactación
    -   Se usa para reconstruir el contexto del modelo para turnos futuros

* * *

## Ubicaciones en disco

Por agente, en el host del Gateway:

-   Almacén: `~/.openclaw/agents//sessions/sessions.json`
-   Transcripciones: `~/.openclaw/agents//sessions/.jsonl`
    -   Sesiones de temas de Telegram: `.../-topic-.jsonl`

OpenClaw resuelve estas rutas a través de `src/config/sessions.ts`.

* * *

## Mantenimiento del almacén y controles de disco

La persistencia de sesiones tiene controles de mantenimiento automático (`session.maintenance`) para `sessions.json` y artefactos de transcripción:

-   `mode`: `warn` (predeterminado) o `enforce`
-   `pruneAfter`: límite de antigüedad para entradas obsoletas (predeterminado `30d`)
-   `maxEntries`: límite de entradas en `sessions.json` (predeterminado `500`)
-   `rotateBytes`: rotar `sessions.json` cuando excede el tamaño (predeterminado `10mb`)
-   `resetArchiveRetention`: retención para archivos de transcripción `*.reset.` (predeterminado: igual que `pruneAfter`; `false` desactiva la limpieza)
-   `maxDiskBytes`: presupuesto opcional para el directorio de sesiones
-   `highWaterBytes`: objetivo opcional después de la limpieza (predeterminado `80%` de `maxDiskBytes`)

Orden de aplicación para la limpieza del presupuesto de disco (`mode: "enforce"`):

1.  Eliminar primero los artefactos de transcripción archivados o huérfanos más antiguos.
2.  Si aún está por encima del objetivo, expulsar las entradas de sesión más antiguas y sus archivos de transcripción.
3.  Continuar hasta que el uso esté en o por debajo de `highWaterBytes`.

En `mode: "warn"`, OpenClaw reporta posibles expulsiones pero no muta el almacén/archivos. Ejecuta el mantenimiento bajo demanda:

```bash
openclaw sessions cleanup --dry-run
openclaw sessions cleanup --enforce
```

* * *

## Sesiones cron y registros de ejecución

Las ejecuciones cron aisladas también crean entradas de sesión/transcripciones, y tienen controles de retención dedicados:

-   `cron.sessionRetention` (predeterminado `24h`) poda las sesiones de ejecución cron aisladas antiguas del almacén de sesiones (`false` desactiva).
-   `cron.runLog.maxBytes` + `cron.runLog.keepLines` podan los archivos `~/.openclaw/cron/runs/.jsonl` (predeterminados: `2_000_000` bytes y `2000` líneas).

* * *

## Claves de sesión (sessionKey)

Una `sessionKey` identifica *en qué cubo de conversación* estás (enrutamiento + aislamiento). Patrones comunes:

-   Chat principal/directo (por agente): `agent::` (predeterminado `main`)
-   Grupo: `agent:::group:`
-   Sala/canal (Discord/Slack): `agent:::channel:` o `...:room:`
-   Cron: `cron:<job.id>`
-   Webhook: `hook:` (a menos que se sobrescriba)

Las reglas canónicas están documentadas en [/concepts/session](../concepts/session.md).

* * *

## Ids de sesión (sessionId)

Cada `sessionKey` apunta a un `sessionId` actual (el archivo de transcripción que continúa la conversación). Reglas generales:

-   **Reinicio** (`/new`, `/reset`) crea un nuevo `sessionId` para esa `sessionKey`.
-   **Reinicio diario** (predeterminado 4:00 AM hora local en el host del gateway) crea un nuevo `sessionId` en el siguiente mensaje después del límite de reinicio.
-   **Expiración por inactividad** (`session.reset.idleMinutes` o heredado `session.idleMinutes`) crea un nuevo `sessionId` cuando llega un mensaje después de la ventana de inactividad. Cuando están configurados tanto el diario como el de inactividad, gana el que expire primero.
-   **Guardia de bifurcación de hilo padre** (`session.parentForkMaxTokens`, predeterminado `100000`) omite la bifurcación de transcripción del padre cuando la sesión padre ya es demasiado grande; el nuevo hilo comienza desde cero. Establece `0` para desactivar.

Detalle de implementación: la decisión ocurre en `initSessionState()` en `src/auto-reply/reply/session.ts`.

* * *

## Esquema del almacén de sesiones (sessions.json)

El tipo de valor del almacén es `SessionEntry` en `src/config/sessions.ts`. Campos clave (no exhaustivo):

-   `sessionId`: id de transcripción actual (el nombre de archivo se deriva de esto a menos que se establezca `sessionFile`)
-   `updatedAt`: marca de tiempo de la última actividad
-   `sessionFile`: ruta de transcripción explícita opcional
-   `chatType`: `direct | group | room` (ayuda a las UIs y a la política de envío)
-   `provider`, `subject`, `room`, `space`, `displayName`: metadatos para etiquetado de grupo/canal
-   Interruptores:
    -   `thinkingLevel`, `verboseLevel`, `reasoningLevel`, `elevatedLevel`
    -   `sendPolicy` (sobrescritura por sesión)
-   Selección de modelo:
    -   `providerOverride`, `modelOverride`, `authProfileOverride`
-   Contadores de tokens (mejor esfuerzo / dependiente del proveedor):
    -   `inputTokens`, `outputTokens`, `totalTokens`, `contextTokens`
-   `compactionCount`: cuántas veces se completó la compactación automática para esta clave de sesión
-   `memoryFlushAt`: marca de tiempo para el último vaciado de memoria pre-compactación
-   `memoryFlushCompactionCount`: conteo de compactación cuando se ejecutó el último vaciado

El almacén es seguro de editar, pero el Gateway es la autoridad: puede reescribir o rehidratar entradas a medida que se ejecutan las sesiones.

* * *

## Estructura de transcripción (\*.jsonl)

Las transcripciones son gestionadas por `SessionManager` de `@mariozechner/pi-coding-agent`. El archivo es JSONL:

-   Primera línea: encabezado de sesión (`type: "session"`, incluye `id`, `cwd`, `timestamp`, opcional `parentSession`)
-   Luego: entradas de sesión con `id` + `parentId` (árbol)

Tipos de entrada notables:

-   `message`: mensajes de usuario/asistente/resultado de herramienta
-   `custom_message`: mensajes inyectados por extensiones que *sí* entran en el contexto del modelo (pueden estar ocultos en la UI)
-   `custom`: estado de extensión que *no* entra en el contexto del modelo
-   `compaction`: resumen de compactación persistido con `firstKeptEntryId` y `tokensBefore`
-   `branch_summary`: resumen persistido al navegar por una rama del árbol

OpenClaw intencionalmente **no** "corrige" las transcripciones; el Gateway usa `SessionManager` para leer/escribirlas.

* * *

## Ventanas de contexto vs tokens rastreados

Dos conceptos diferentes importan:

1.  **Ventana de contexto del modelo**: límite duro por modelo (tokens visibles para el modelo)
2.  **Contadores del almacén de sesiones**: estadísticas acumulativas escritas en `sessions.json` (usadas para /status y paneles)

Si estás ajustando límites:

-   La ventana de contexto proviene del catálogo de modelos (y puede ser sobrescrita vía configuración).
-   `contextTokens` en el almacén es un valor estimado/informativo en tiempo de ejecución; no lo trates como una garantía estricta.

Para más información, consulta [/token-use](./token-use.md).

* * *

## Compactación: qué es

La compactación resume la conversación anterior en una entrada `compaction` persistida en la transcripción y mantiene intactos los mensajes recientes. Después de la compactación, los turnos futuros ven:

-   El resumen de compactación
-   Los mensajes después de `firstKeptEntryId`

La compactación es **persistente** (a diferencia de la poda de sesiones). Consulta [/concepts/session-pruning](../concepts/session-pruning.md).

* * *

## Cuándo ocurre la compactación automática (tiempo de ejecución de Pi)

En el agente Pi embebido, la compactación automática se activa en dos casos:

1.  **Recuperación de desbordamiento**: el modelo devuelve un error de desbordamiento de contexto → compactar → reintentar.
2.  **Mantenimiento por umbral**: después de un turno exitoso, cuando:

`contextTokens > contextWindow - reserveTokens` Donde:

-   `contextWindow` es la ventana de contexto del modelo
-   `reserveTokens` es el margen reservado para prompts + la siguiente salida del modelo

Estas son semánticas del tiempo de ejecución de Pi (OpenClaw consume los eventos, pero Pi decide cuándo compactar).

* * *

## Configuración de compactación (reserveTokens, keepRecentTokens)

La configuración de compactación de Pi reside en la configuración de Pi:

```json
{
  compaction: {
    enabled: true,
    reserveTokens: 16384,
    keepRecentTokens: 20000,
  },
}
```

OpenClaw también aplica un piso de seguridad para ejecuciones embebidas:

-   Si `compaction.reserveTokens < reserveTokensFloor`, OpenClaw lo aumenta.
-   El piso predeterminado es `20000` tokens.
-   Establece `agents.defaults.compaction.reserveTokensFloor: 0` para desactivar el piso.
-   Si ya es más alto, OpenClaw lo deja como está.

Por qué: dejar suficiente margen para "mantenimiento" de múltiples turnos (como escrituras de memoria) antes de que la compactación sea inevitable. Implementación: `ensurePiCompactionReserveTokens()` en `src/agents/pi-settings.ts` (llamada desde `src/agents/pi-embedded-runner.ts`).

* * *

## Superficies visibles para el usuario

Puedes observar la compactación y el estado de la sesión a través de:

-   `/status` (en cualquier sesión de chat)
-   `openclaw status` (CLI)
-   `openclaw sessions` / `sessions --json`
-   Modo detallado: `🧹 Auto-compaction complete` + conteo de compactación

* * *

## Mantenimiento silencioso (NO\_REPLY)

OpenClaw admite turnos "silenciosos" para tareas en segundo plano donde el usuario no debe ver salida intermedia. Convención:

-   El asistente comienza su salida con `NO_REPLY` para indicar "no entregar una respuesta al usuario".
-   OpenClaw elimina/suprime esto en la capa de entrega.

A partir de `2026.1.10`, OpenClaw también suprime la **transmisión de borrador/escritura** cuando un fragmento parcial comienza con `NO_REPLY`, para que las operaciones silenciosas no filtren salida parcial a mitad del turno.

* * *

## "Vaciado de memoria" pre-compactación (implementado)

Objetivo: antes de que ocurra la compactación automática, ejecutar un turno agéntico silencioso que escriba estado duradero en disco (ej. `memory/YYYY-MM-DD.md` en el espacio de trabajo del agente) para que la compactación no pueda borrar contexto crítico. OpenClaw usa el enfoque de **vaciado pre-umbral**:

1.  Monitorear el uso del contexto de la sesión.
2.  Cuando cruza un "umbral suave" (por debajo del umbral de compactación de Pi), ejecutar una directiva silenciosa "escribir memoria ahora" al agente.
3.  Usar `NO_REPLY` para que el usuario no vea nada.

Configuración (`agents.defaults.compaction.memoryFlush`):

-   `enabled` (predeterminado: `true`)
-   `softThresholdTokens` (predeterminado: `4000`)
-   `prompt` (mensaje de usuario para el turno de vaciado)
-   `systemPrompt` (prompt de sistema extra añadido para el turno de vaciado)

Notas:

-   El prompt/prompt de sistema predeterminado incluyen una pista `NO_REPLY` para suprimir la entrega.
-   El vaciado se ejecuta una vez por ciclo de compactación (rastreado en `sessions.json`).
-   El vaciado se ejecuta solo para sesiones Pi embebidas (los backends CLI lo omiten).
-   El vaciado se omite cuando el espacio de trabajo de la sesión es de solo lectura (`workspaceAccess: "ro"` o `"none"`).
-   Consulta [Memoria](../concepts/memory.md) para el diseño del archivo del espacio de trabajo y patrones de escritura.

Pi también expone un gancho `session_before_compact` en la API de extensión, pero la lógica de vaciado de OpenClaw vive hoy en el lado del Gateway.

* * *

## Lista de verificación para solución de problemas

-   ¿Clave de sesión incorrecta? Comienza con [/concepts/session](../concepts/session.md) y confirma la `sessionKey` en `/status`.
-   ¿Desajuste entre almacén y transcripción? Confirma el host del Gateway y la ruta del almacén desde `openclaw status`.
-   ¿Compactación excesiva? Verifica:
    -   ventana de contexto del modelo (demasiado pequeña)
    -   configuración de compactación (`reserveTokens` demasiado alto para la ventana del modelo puede causar compactación temprana)
    -   inflación de resultados de herramientas: activa/ajusta la poda de sesiones
-   ¿Turnos silenciosos con fugas? Confirma que la respuesta comienza con `NO_REPLY` (token exacto) y que estás en una compilación que incluya la corrección de supresión de transmisión.

[Node.js](../install/node.md)[Configuración](../start/setup.md)