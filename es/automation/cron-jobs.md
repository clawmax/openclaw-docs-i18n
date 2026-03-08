title: "Guía de Cron Jobs de OpenClaw: Programación y Automatización"
description: "Aprende a programar tareas automatizadas con cron jobs de OpenClaw. Configura recordatorios únicos, tareas recurrentes y elige entre modos de ejecución principal o aislada."
keywords: ["cron de openclaw", "programación de automatización", "trabajos cron", "programador de gateway", "turno de agente aislado", "eventos del sistema", "tareas recurrentes", "entrega de trabajos"]
---

  Automatización

  
# Cron Jobs

> **¿Cron vs Heartbeat?** Consulta [Cron vs Heartbeat](./cron-vs-heartbeat.md) para orientación sobre cuándo usar cada uno.

Cron es el programador integrado del Gateway. Persiste trabajos, despierta al agente en el momento adecuado y puede, opcionalmente, entregar la salida a un chat. Si quieres *"ejecutar esto cada mañana"* o *"avisar al agente en 20 minutos"*, cron es el mecanismo. Solución de problemas: [/automation/troubleshooting](./troubleshooting.md)

## TL;DR

-   Cron se ejecuta **dentro del Gateway** (no dentro del modelo).
-   Los trabajos persisten en `~/.openclaw/cron/` para que los reinicios no pierdan las programaciones.
-   Dos estilos de ejecución:
    -   **Sesión principal**: encola un evento del sistema, luego se ejecuta en el siguiente latido.
    -   **Aislada**: ejecuta un turno de agente dedicado en `cron:`, con entrega (anunciar por defecto o ninguna).
-   Los despertadores son de primera clase: un trabajo puede solicitar "despertar ahora" vs "siguiente latido".
-   La publicación de webhook es por trabajo mediante `delivery.mode = "webhook"` + `delivery.to = ""`.
-   El fallback heredado permanece para trabajos almacenados con `notify: true` cuando `cron.webhook` está configurado; migra esos trabajos al modo de entrega webhook.

## Inicio rápido (accionable)

Crea un recordatorio único, verifica que existe y ejecútalo inmediatamente:

```bash
openclaw cron add \
  --name "Reminder" \
  --at "2026-02-01T16:00:00Z" \
  --session main \
  --system-event "Reminder: check the cron docs draft" \
  --wake now \
  --delete-after-run

openclaw cron list
openclaw cron run <job-id>
openclaw cron runs --id <job-id>
```

Programa un trabajo aislado recurrente con entrega:

```bash
openclaw cron add \
  --name "Morning brief" \
  --cron "0 7 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize overnight updates." \
  --announce \
  --channel slack \
  --to "channel:C1234567890"
```

## Equivalentes de llamadas a herramientas (herramienta cron del Gateway)

Para las formas JSON canónicas y ejemplos, consulta [Esquema JSON para llamadas a herramientas](./cron-jobs.md#json-schema-for-tool-calls).

## Dónde se almacenan los cron jobs

Los cron jobs se persisten en el host del Gateway en `~/.openclaw/cron/jobs.json` por defecto. El Gateway carga el archivo en memoria y lo vuelve a escribir en cambios, por lo que las ediciones manuales solo son seguras cuando el Gateway está detenido. Prefiere `openclaw cron add/edit` o la API de llamadas a herramientas cron para cambios.

## Descripción general para principiantes

Piensa en un cron job como: **cuándo** ejecutar + **qué** hacer.

1.  **Elige una programación**
    -   Recordatorio único → `schedule.kind = "at"` (CLI: `--at`)
    -   Trabajo repetitivo → `schedule.kind = "every"` o `schedule.kind = "cron"`
    -   Si tu marca de tiempo ISO omite una zona horaria, se trata como **UTC**.
2.  **Elige dónde se ejecuta**
    -   `sessionTarget: "main"` → ejecutar durante el siguiente latido con el contexto principal.
    -   `sessionTarget: "isolated"` → ejecutar un turno de agente dedicado en `cron:`.
3.  **Elige la carga útil**
    -   Sesión principal → `payload.kind = "systemEvent"`
    -   Sesión aislada → `payload.kind = "agentTurn"`

Opcional: los trabajos únicos (`schedule.kind = "at"`) se eliminan después del éxito por defecto. Configura `deleteAfterRun: false` para mantenerlos (se deshabilitarán después del éxito).

## Conceptos

### Trabajos

Un cron job es un registro almacenado con:

-   una **programación** (cuándo debe ejecutarse),
-   una **carga útil** (qué debe hacer),
-   un **modo de entrega** opcional (`announce`, `webhook`, o `none`).
-   un **enlace de agente** opcional (`agentId`): ejecutar el trabajo bajo un agente específico; si falta o es desconocido, el gateway recurre al agente por defecto.

Los trabajos se identifican por un `jobId` estable (utilizado por la CLI/APIs del Gateway). En las llamadas a herramientas del agente, `jobId` es canónico; se acepta el `id` heredado por compatibilidad. Los trabajos únicos se autoeliminan después del éxito por defecto; configura `deleteAfterRun: false` para mantenerlos.

### Programaciones

Cron admite tres tipos de programación:

-   `at`: marca de tiempo única mediante `schedule.at` (ISO 8601).
-   `every`: intervalo fijo (ms).
-   `cron`: expresión cron de 5 campos (o 6 campos con segundos) con zona horaria IANA opcional.

Las expresiones cron usan `croner`. Si se omite una zona horaria, se usa la zona horaria local del host del Gateway. Para reducir los picos de carga en la hora en punto entre muchos gateways, OpenClaw aplica una ventana de desfase determinista por trabajo de hasta 5 minutos para expresiones recurrentes en la hora en punto (por ejemplo `0 * * * *`, `0 */2 * * *`). Las expresiones de hora fija como `0 7 * * *` permanecen exactas. Para cualquier programación cron, puedes configurar una ventana de desfase explícita con `schedule.staggerMs` (`0` mantiene el tiempo exacto). Accesos directos de CLI:

-   `--stagger 30s` (o `1m`, `5m`) para configurar una ventana de desfase explícita.
-   `--exact` para forzar `staggerMs = 0`.

### Ejecución principal vs aislada

#### Trabajos de sesión principal (eventos del sistema)

Los trabajos principales encolan un evento del sistema y, opcionalmente, despiertan al ejecutor de latidos. Deben usar `payload.kind = "systemEvent"`.

-   `wakeMode: "now"` (por defecto): el evento desencadena una ejecución de latido inmediata.
-   `wakeMode: "next-heartbeat"`: el evento espera el siguiente latido programado.

Esta es la mejor opción cuando quieres el prompt de latido normal + el contexto de la sesión principal. Consulta [Heartbeat](../gateway/heartbeat.md).

#### Trabajos aislados (sesiones cron dedicadas)

Los trabajos aislados ejecutan un turno de agente dedicado en la sesión `cron:`. Comportamientos clave:

-   El prompt tiene el prefijo `[cron: ]` para trazabilidad.
-   Cada ejecución comienza con un **id de sesión nuevo** (sin arrastre de conversación previa).
-   Comportamiento por defecto: si se omite `delivery`, los trabajos aislados anuncian un resumen (`delivery.mode = "announce"`).
-   `delivery.mode` elige qué sucede:
    -   `announce`: entregar un resumen al canal objetivo y publicar un breve resumen en la sesión principal.
    -   `webhook`: hacer POST de la carga útil del evento finalizado a `delivery.to` cuando el evento finalizado incluya un resumen.
    -   `none`: solo interno (sin entrega, sin resumen en la sesión principal).
-   `wakeMode` controla cuándo se publica el resumen en la sesión principal:
    -   `now`: latido inmediato.
    -   `next-heartbeat`: espera el siguiente latido programado.

Usa trabajos aislados para tareas ruidosas, frecuentes o "tareas de fondo" que no deberían saturar tu historial de chat principal.

### Formas de carga útil (qué se ejecuta)

Se admiten dos tipos de carga útil:

-   `systemEvent`: solo sesión principal, se enruta a través del prompt de latido.
-   `agentTurn`: solo sesión aislada, ejecuta un turno de agente dedicado.

Campos comunes de `agentTurn`:

-   `message`: prompt de texto obligatorio.
-   `model` / `thinking`: anulaciones opcionales (ver más abajo).
-   `timeoutSeconds`: anulación de tiempo de espera opcional.
-   `lightContext`: modo de arranque ligero opcional para trabajos que no necesitan inyección de archivos de arranque del espacio de trabajo.

Configuración de entrega:

-   `delivery.mode`: `none` | `announce` | `webhook`.
-   `delivery.channel`: `last` o un canal específico.
-   `delivery.to`: objetivo específico del canal (anunciar) o URL de webhook (modo webhook).
-   `delivery.bestEffort`: evitar que el trabajo falle si falla la entrega del anuncio.

La entrega por anuncio suprime los envíos de herramientas de mensajería para la ejecución; usa `delivery.channel`/`delivery.to` para dirigirse al chat. Cuando `delivery.mode = "none"`, no se publica ningún resumen en la sesión principal. Si se omite `delivery` para trabajos aislados, OpenClaw usa `announce` por defecto.

#### Flujo de entrega por anuncio

Cuando `delivery.mode = "announce"`, cron entrega directamente a través de los adaptadores de canal saliente. El agente principal no se activa para crear o reenviar el mensaje. Detalles del comportamiento:

-   Contenido: la entrega usa las cargas útiles salientes de la ejecución aislada (texto/media) con fragmentación y formato de canal normales.
-   Las respuestas solo de latido (`HEARTBEAT_OK` sin contenido real) no se entregan.
-   Si la ejecución aislada ya envió un mensaje al mismo objetivo a través de la herramienta de mensaje, se omite la entrega para evitar duplicados.
-   Los objetivos de entrega faltantes o inválidos hacen fallar el trabajo a menos que `delivery.bestEffort = true`.
-   Se publica un breve resumen en la sesión principal solo cuando `delivery.mode = "announce"`.
-   El resumen de la sesión principal respeta `wakeMode`: `now` desencadena un latido inmediato y `next-heartbeat` espera el siguiente latido programado.

#### Flujo de entrega por webhook

Cuando `delivery.mode = "webhook"`, cron hace POST de la carga útil del evento finalizado a `delivery.to` cuando el evento finalizado incluye un resumen. Detalles del comportamiento:

-   El endpoint debe ser una URL HTTP(S) válida.
-   No se intenta la entrega al canal en modo webhook.
-   No se publica ningún resumen en la sesión principal en modo webhook.
-   Si `cron.webhookToken` está configurado, el encabezado de autenticación es `Authorization: Bearer <cron.webhookToken>`.
-   Fallback obsoleto: los trabajos heredados almacenados con `notify: true` aún publican en `cron.webhook` (si está configurado), con una advertencia para que puedas migrar a `delivery.mode = "webhook"`.

### Anulaciones de modelo y pensamiento

Los trabajos aislados (`agentTurn`) pueden anular el modelo y el nivel de pensamiento:

-   `model`: Cadena de proveedor/modelo (ej., `anthropic/claude-sonnet-4-20250514`) o alias (ej., `opus`)
-   `thinking`: Nivel de pensamiento (`off`, `minimal`, `low`, `medium`, `high`, `xhigh`; solo modelos GPT-5.2 + Codex)

Nota: Puedes configurar `model` también en trabajos de sesión principal, pero cambia el modelo de la sesión principal compartida. Recomendamos anulaciones de modelo solo para trabajos aislados para evitar cambios de contexto inesperados. Prioridad de resolución:

1.  Anulación de carga útil del trabajo (más alta)
2.  Valores por defecto específicos del hook (ej., `hooks.gmail.model`)
3.  Configuración por defecto del agente

### Contexto de arranque ligero

Los trabajos aislados (`agentTurn`) pueden configurar `lightContext: true` para ejecutarse con contexto de arranque ligero.

-   Usa esto para tareas programadas que no necesitan inyección de archivos de arranque del espacio de trabajo.
-   En la práctica, el tiempo de ejecución integrado se ejecuta con `bootstrapContextMode: "lightweight"`, que mantiene el contexto de arranque cron vacío a propósito.
-   Equivalentes CLI: `openclaw cron add --light-context ...` y `openclaw cron edit --light-context`.

### Entrega (canal + objetivo)

Los trabajos aislados pueden entregar la salida a un canal a través de la configuración de nivel superior `delivery`:

-   `delivery.mode`: `announce` (entrega al canal), `webhook` (HTTP POST), o `none`.
-   `delivery.channel`: `whatsapp` / `telegram` / `discord` / `slack` / `mattermost` (plugin) / `signal` / `imessage` / `last`.
-   `delivery.to`: objetivo del destinatario específico del canal.

La entrega `announce` solo es válida para trabajos aislados (`sessionTarget: "isolated"`). La entrega `webhook` es válida tanto para trabajos principales como aislados. Si se omite `delivery.channel` o `delivery.to`, cron puede recurrir a la "última ruta" de la sesión principal (el último lugar donde respondió el agente). Recordatorios de formato de objetivo:

-   Los objetivos de Slack/Discord/Mattermost (plugin) deben usar prefijos explícitos (ej. `channel:`, `user:`) para evitar ambigüedad.
-   Los temas de Telegram deben usar la forma `:topic:` (ver más abajo).

#### Objetivos de entrega de Telegram (temas / hilos de foro)

Telegram admite temas de foro a través de `message_thread_id`. Para la entrega cron, puedes codificar el tema/hilo en el campo `to`:

-   `-1001234567890` (solo id de chat)
-   `-1001234567890:topic:123` (preferido: marcador de tema explícito)
-   `-1001234567890:123` (abreviado: sufijo numérico)

También se aceptan objetivos con prefijo como `telegram:...` / `telegram:group:...`:

-   `telegram:group:-1001234567890:topic:123`

## Esquema JSON para llamadas a herramientas

Usa estas formas al llamar directamente a las herramientas `cron.*` del Gateway (llamadas a herramientas del agente o RPC). Los flags de CLI aceptan duraciones humanas como `20m`, pero las llamadas a herramientas deben usar una cadena ISO 8601 para `schedule.at` y milisegundos para `schedule.everyMs`.

### Parámetros de cron.add

Trabajo único, sesión principal (evento del sistema):

```json
{
  "name": "Reminder",
  "schedule": { "kind": "at", "at": "2026-02-01T16:00:00Z" },
  "sessionTarget": "main",
  "wakeMode": "now",
  "payload": { "kind": "systemEvent", "text": "Reminder text" },
  "deleteAfterRun": true
}
```

Trabajo recurrente, aislado con entrega:

```json
{
  "name": "Morning brief",
  "schedule": { "kind": "cron", "expr": "0 7 * * *", "tz": "America/Los_Angeles" },
  "sessionTarget": "isolated",
  "wakeMode": "next-heartbeat",
  "payload": {
    "kind": "agentTurn",
    "message": "Summarize overnight updates.",
    "lightContext": true
  },
  "delivery": {
    "mode": "announce",
    "channel": "slack",
    "to": "channel:C1234567890",
    "bestEffort": true
  }
}
```

Notas:

-   `schedule.kind`: `at` (`at`), `every` (`everyMs`), o `cron` (`expr`, `tz` opcional).
-   `schedule.at` acepta ISO 8601 (zona horaria opcional; tratada como UTC cuando se omite).
-   `everyMs` son milisegundos.
-   `sessionTarget` debe ser `"main"` o `"isolated"` y debe coincidir con `payload.kind`.
-   Campos opcionales: `agentId`, `description`, `enabled`, `deleteAfterRun` (por defecto true para `at`), `delivery`.
-   `wakeMode` por defecto es `"now"` cuando se omite.

### Parámetros de cron.update

```json
{
  "jobId": "job-123",
  "patch": {
    "enabled": false,
    "schedule": { "kind": "every", "everyMs": 3600000 }
  }
}
```

Notas:

-   `jobId` es canónico; se acepta `id` por compatibilidad.
-   Usa `agentId: null` en el parche para borrar un enlace de agente.

### Parámetros de cron.run y cron.remove

```json
{ "jobId": "job-123", "mode": "force" }
```

```json
{ "jobId": "job-123" }
```

## Almacenamiento e historial

-   Almacén de trabajos: `~/.openclaw/cron/jobs.json` (JSON gestionado por el Gateway).
-   Historial de ejecuciones: `~/.openclaw/cron/runs/.jsonl` (JSONL, podado automáticamente por tamaño y número de líneas).
-   Las sesiones de ejecución cron aisladas en `sessions.json` se podan por `cron.sessionRetention` (por defecto `24h`; configura `false` para deshabilitar).
-   Anular ruta de almacenamiento: `cron.store` en la configuración.

## Política de reintento

Cuando un trabajo falla, OpenClaw clasifica los errores como **transitorios** (reintentables) o **permanentes** (deshabilitar inmediatamente).

### Errores transitorios (reintentados)

-   Límite de tasa (429, demasiadas solicitudes, recursos agotados)
-   Sobrecarga del proveedor (por ejemplo, Anthropic `529 overloaded_error`, resúmenes de fallback por sobrecarga)
-   Errores de red (tiempo de espera, ECONNRESET, fetch fallido, socket)
-   Errores del servidor (5xx)
-   Errores relacionados con Cloudflare

### Errores permanentes (sin reintento)

-   Fallos de autenticación (clave API inválida, no autorizado)
-   Errores de configuración o validación
-   Otros errores no transitorios

### Comportamiento por defecto (sin configuración)

**Trabajos únicos (`schedule.kind: "at"`):**

-   En error transitorio: reintentar hasta 3 veces con retroceso exponencial (30s → 1m → 5m).
-   En error permanente: deshabilitar inmediatamente.
-   En éxito u omisión: deshabilitar (o eliminar si `deleteAfterRun: true`).

**Trabajos recurrentes (`cron` / `every`):**

-   En cualquier error: aplicar retroceso exponencial (30s → 1m → 5m → 15m → 60m) antes de la siguiente ejecución programada.
-   El trabajo permanece habilitado; el retroceso se reinicia después de la siguiente ejecución exitosa.

Configura `cron.retry` para anular estos valores por defecto (ver [Configuración](./cron-jobs.md#configuration)).

## Configuración

```json
{
  cron: {
    enabled: true, // por defecto true
    store: "~/.openclaw/cron/jobs.json",
    maxConcurrentRuns: 1, // por defecto 1
    // Opcional: anular política de reintento para trabajos únicos
    retry: {
      maxAttempts: 3,
      backoffMs: [60000, 120000, 300000],
      retryOn: ["rate_limit", "overloaded", "network", "server_error"],
    },
    webhook: "https://example.invalid/legacy", // fallback obsoleto para trabajos almacenados con notify:true
    webhookToken: "replace-with-dedicated-webhook-token", // token bearer opcional para modo webhook
    sessionRetention: "24h", // cadena de duración o false
    runLog: {
      maxBytes: "2mb", // por defecto 2_000_000 bytes
      keepLines: 2000, // por defecto 2000
    },
  },
}
```

Comportamiento de poda de registros de ejecución:

-   `cron.runLog.maxBytes`: tamaño máximo del archivo de registro de ejecución antes de podar.
-   `cron.runLog.keepLines`: al podar, mantener solo las N líneas más nuevas.
-   Ambos se aplican a los archivos `cron/runs/.jsonl`.

Comportamiento de webhook:

-   Preferido: configura `delivery.mode: "webhook"` con `delivery.to: "https://..."` por trabajo.
-   Las URLs de webhook deben ser URLs `http://` o `https://` válidas.
-   Al publicar, la carga útil es el JSON del evento cron finalizado.
-   Si `cron.webhookToken` está configurado, el encabezado de autenticación es `Authorization: Bearer <cron.webhookToken>`.
-   Si `cron.webhookToken` no está configurado, no se envía el encabezado `Authorization`.
-   Fallback obsoleto: los trabajos heredados almacenados con `notify: true` aún usan `cron.webhook` cuando está presente.

Deshabilitar cron completamente:

-   `cron.enabled: false` (configuración)
-   `OPENCLAW_SKIP_CRON=1` (env)

## Mantenimiento

Cron tiene dos rutas de mantenimiento integradas: retención de sesiones de ejecución aisladas y poda de registros de ejecución.

### Valores por defecto

-   `cron.sessionRetention`: `24h` (configura `false` para deshabilitar la poda de sesiones de ejecución)
-   `cron.runLog.maxBytes`: `2_000_000` bytes
-   `cron.runLog.keepLines`: `2000`

### Cómo funciona

-   Las ejecuciones aisladas crean entradas de sesión (`...:cron::run:`) y archivos de transcripción.
-   El recolector elimina las entradas de sesión de ejecución caducadas más antiguas que `cron.sessionRetention`.
-   Para las sesiones de ejecución eliminadas que ya no están referenciadas por el almacén de sesiones, OpenClaw archiva los archivos de transcripción y purga los archivos antiguos eliminados en la misma ventana de retención.
-   Después de cada anexión de ejecución, se verifica el tamaño de `cron/runs/.jsonl`:
    -   si el tamaño del archivo excede `runLog.maxBytes`, se recorta a las `runLog.keepLines` líneas más nuevas.

### Advertencia de rendimiento para programadores de alto volumen

Las configuraciones cron de alta frecuencia pueden generar grandes huellas de sesiones de ejecución y registros de ejecución. El mantenimiento está integrado, pero límites laxos aún pueden crear trabajo de E/S y limpieza evitable. Qué vigilar:

-   ventanas largas de `cron.sessionRetention` con muchas ejecuciones aisladas
-   alto `cron.runLog.keepLines` combinado con `runLog.maxBytes` grande
-   muchos trabajos recurrentes ruidosos escribiendo en el mismo `cron/runs/.jsonl`

Qué hacer:

-   mantén `cron.sessionRetention` tan corta como lo permitan tus necesidades de depuraci��n/auditoría
-   mantén los registros de ejecución acotados con `runLog.maxBytes` y `runLog.keepLines` moderados
-   mueve trabajos de fondo ruidosos a modo aislado con reglas de entrega que eviten charla innecesaria
-   revisa el crecimiento periódicamente con `openclaw cron runs` y ajusta la retención antes de que los registros se vuelvan grandes

### Ejemplos de personalización

Mantener sesiones de ejecución por una semana y permitir registros de ejecución más grandes:

```json
{
  cron: {
    sessionRetention: "7d",
    runLog: {
      maxBytes: "10mb",
      keepLines: 5000,
    },
  },
}
```

Deshabilitar la poda de sesiones de ejecución aisladas pero mantener la poda de registros de ejecución:

```json
{
  cron: {
    sessionRetention: false,
    runLog: {
      maxBytes: "5mb",
      keepLines: 3000,
    },
  },
}
```

Ajustar para uso cron de alto volumen (ejemplo):

```json
{
  cron: {
    sessionRetention: "12h",
    runLog: {
      maxBytes: "3mb",
      keepLines: 1500,
    },
  },
}
```

## Inicio rápido de CLI

Recordatorio único (UTC ISO, autoeliminar después del éxito):

```bash
openclaw cron add \
  --name "Send reminder" \
  --at "2026-01-12T18:00:00Z" \
  --session main \
  --system-event "Reminder: submit expense report." \
  --wake now \
  --delete-after-run
```

Recordatorio único (sesión principal, despertar inmediatamente):

```bash
openclaw cron add \
  --name "Calendar check" \
  --at "20m" \
  --session main \
  --system-event "Next heartbeat: check calendar." \
  --wake now
```

Trabajo aislado recurrente (anunciar a WhatsApp):

```bash
openclaw cron add \
  --name "Morning status" \
  --cron "0 7 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize inbox + calendar for today." \
  --announce \
  --channel whatsapp \
  --to "+15551234567"
```

Trabajo cron recurrente con desfase explícito de 30 segundos:

```bash
openclaw cron add \
  --name "Minute watcher" \
  --cron "0 * * * * *" \
  --tz "UTC" \
  --stagger 30s \
  --session isolated \
  --message "Run minute watcher checks." \
  --announce
```

Trabajo aislado recurrente (entregar a un tema de Telegram):

```bash
openclaw cron add \
  --name "Nightly summary (topic)" \
  --cron "0 22 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize today; send to the nightly topic." \
  --announce \
  --channel telegram \
  --to "-1001234567890:topic:123"
```

Trabajo aislado con anulación de modelo y pensamiento:

```bash
openclaw cron add \
  --name "Deep analysis" \
  --cron "0 6 * * 1" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Weekly deep analysis of project progress." \
  --model "opus" \
  --thinking high \
  --announce \
  --channel whatsapp \
  --to "+15551234567"
```

Selección de agente (configuraciones multiagente):

```bash
# Fijar un trabajo al agente "ops" (recurre al por defecto si ese agente falta)
openclaw cron add --name "Ops sweep" --cron "0 6 * * *" --session isolated --message "Check ops queue" --agent ops

# Cambiar o borrar el agente en un trabajo existente
openclaw cron edit <jobId> --agent ops
openclaw cron edit <jobId> --clear-agent
```

Ejecución manual (force es el valor por defecto, usa `--due` para ejecutar solo cuando corresponda):

```bash
openclaw cron run <jobId>
openclaw cron run <jobId> --due
```

Editar un trabajo existente (parchear campos):

```bash
openclaw cron edit <jobId> \
  --message "Updated prompt" \
  --model "opus" \
  --thinking low
```

Forzar un trabajo cron existente para que se ejecute exactamente según la programación (sin desfase):

```bash
openclaw cron edit <jobId> --exact
```

Historial de ejecuciones:

```bash
openclaw cron runs --id <jobId> --limit 50
```

Evento del sistema inmediato sin crear un trabajo:

```bash
openclaw system event --mode now --text "Next heartbeat: check battery."
```

## Superficie de API del Gateway

-   `cron.list`, `cron.status`, `cron.add`, `cron.update`, `cron.remove`
-   `cron.run` (forzar o cuando corresponda), `cron.runs` Para eventos del sistema inmediatos sin un trabajo, usa [`openclaw system event`](../cli/system.md).

## Solución de problemas

### "No se ejecuta nada"

-   Verifica que cron esté habilitado: `cron.enabled` y `OPENCLAW_SKIP_CRON`.
-   Verifica que el Gateway se esté ejecutando continuamente (cron se ejecuta dentro del proceso del Gateway).
-   Para programaciones `cron`: confirma la zona horaria (`--tz`) vs la zona horaria del host.

### Un trabajo recurrente sigue retrasándose después de fallos

-   OpenClaw aplica retroceso exponencial de reintento para trabajos recurrentes después de errores consecutivos: 30s, 1m, 5m, 15m, luego 60m entre reintentos.
-   El retroceso se reinicia automáticamente después de la siguiente ejecución exitosa.
-   Los trabajos únicos (`at`) reintentan errores transitorios (límite de tasa, sobrecargado, red, server\_error) hasta 3 veces con retroceso; los errores permanentes deshabilitan inmediatamente. Consulta [Política de reintento](./cron-jobs.md#retry-policy).

### Telegram entrega en el lugar equivocado

-   Para temas de foro, usa `-100…:topic:` para que sea explícito y no ambiguo.
-   Si ves prefijos `telegram:...` en registros o objetivos de "última ruta" almacenados, es normal; la entrega cron los acepta y aún analiza los IDs de tema correctamente.

### Reintentos de entrega de anuncio de subagente

-   Cuando una ejecución de subagente se completa, el gateway anuncia el resultado a la sesión solicitante.
-   Si el flujo de anuncio devuelve `false` (ej. la sesión solicitante está ocupada), el gateway reintenta hasta 3 veces con seguimiento a través de `announceRetryCount`.
-   Los anuncios más antiguos de 5 minutos después de `endedAt` se expiran forzosamente para evitar que entradas obsoletas se repitan indefinidamente.
-   Si ves entregas de anuncio repetidas en los registros, revisa el registro de subagentes en busca de entradas con valores altos de `announceRetryCount`.

[Hooks](./hooks.md)[Cron vs Heartbeat](./cron-vs-heartbeat.md)