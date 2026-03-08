

  Sesiones y memoria

  
# Gestión de Sesiones

OpenClaw trata **una sesión de chat directo por agente** como la principal. Los chats directos se colapsan a `agent::` (por defecto `main`), mientras que los chats de grupo/canal obtienen sus propias claves. Se respeta `session.mainKey`. Usa `session.dmScope` para controlar cómo se agrupan los **mensajes directos**:

-   `main` (por defecto): todos los DMs comparten la sesión principal para continuidad.
-   `per-peer`: aislar por id del remitente a través de canales.
-   `per-channel-peer`: aislar por canal + remitente (recomendado para bandejas de entrada multi-usuario).
-   `per-account-channel-peer`: aislar por cuenta + canal + remitente (recomendado para bandejas de entrada multi-cuenta). Usa `session.identityLinks` para mapear ids de pares con prefijo de proveedor a una identidad canónica para que la misma persona comparta una sesión de DM a través de canales cuando se usa `per-peer`, `per-channel-peer`, o `per-account-channel-peer`.

## Modo DM seguro (recomendado para configuraciones multi-usuario)

> **Advertencia de Seguridad:** Si tu agente puede recibir DMs de **múltiples personas**, deberías considerar seriamente habilitar el modo DM seguro. Sin él, todos los usuarios comparten el mismo contexto de conversación, lo que puede filtrar información privada entre usuarios.

**Ejemplo del problema con la configuración por defecto:**

-   Alice (`<SENDER_A>`) le envía un mensaje a tu agente sobre un tema privado (por ejemplo, una cita médica)
-   Bob (`<SENDER_B>`) le envía un mensaje a tu agente preguntando "¿De qué estábamos hablando?"
-   Debido a que ambos DMs comparten la misma sesión, el modelo podría responderle a Bob usando el contexto previo de Alice.

**La solución:** Configura `dmScope` para aislar sesiones por usuario:

```
// ~/.openclaw/openclaw.json
{
  session: {
    // Modo DM seguro: aislar contexto de DM por canal + remitente.
    dmScope: "per-channel-peer",
  },
}
```

**Cuándo habilitar esto:**

-   Tienes aprobaciones de emparejamiento para más de un remitente
-   Usas una lista de permitidos de DM con múltiples entradas
-   Configuras `dmPolicy: "open"`
-   Múltiples números de teléfono o cuentas pueden enviar mensajes a tu agente

Notas:

-   El valor por defecto es `dmScope: "main"` para continuidad (todos los DMs comparten la sesión principal). Esto está bien para configuraciones de un solo usuario.
-   La incorporación local de CLI escribe `session.dmScope: "per-channel-peer"` por defecto cuando no está configurado (se preservan los valores explícitos existentes).
-   Para bandejas de entrada multi-cuenta en el mismo canal, prefiere `per-account-channel-peer`.
-   Si la misma persona te contacta en múltiples canales, usa `session.identityLinks` para colapsar sus sesiones de DM en una identidad canónica.
-   Puedes verificar tu configuración de DM con `openclaw security audit` (ver [seguridad](../cli/security.md)).

## La puerta de enlace es la fuente de la verdad

Todo el estado de la sesión es **propiedad de la puerta de enlace** (el OpenClaw "maestro"). Los clientes de UI (aplicación macOS, WebChat, etc.) deben consultar a la puerta de enlace para listas de sesiones y conteos de tokens en lugar de leer archivos locales.

-   En **modo remoto**, el almacén de sesiones que te importa reside en el host de la puerta de enlace remota, no en tu Mac.
-   Los conteos de tokens mostrados en las UIs provienen de los campos de almacén de la puerta de enlace (`inputTokens`, `outputTokens`, `totalTokens`, `contextTokens`). Los clientes no analizan transcripciones JSONL para "corregir" los totales.

## Dónde reside el estado

-   En el **host de la puerta de enlace**:
    -   Archivo de almacén: `~/.openclaw/agents//sessions/sessions.json` (por agente).
-   Transcripciones: `~/.openclaw/agents//sessions/.jsonl` (las sesiones de temas de Telegram usan `.../-topic-.jsonl`).
-   El almacén es un mapa `sessionKey -> { sessionId, updatedAt, ... }`. Eliminar entradas es seguro; se recrean bajo demanda.
-   Las entradas de grupo pueden incluir `displayName`, `channel`, `subject`, `room`, y `space` para etiquetar sesiones en las UIs.
-   Las entradas de sesión incluyen metadatos de `origin` (etiqueta + pistas de enrutamiento) para que las UIs puedan explicar de dónde vino una sesión.
-   OpenClaw **no** lee carpetas de sesiones heredadas de Pi/Tau.

## Mantenimiento

OpenClaw aplica mantenimiento del almacén de sesiones para mantener `sessions.json` y los artefactos de transcripción acotados en el tiempo.

### Valores por defecto

-   `session.maintenance.mode`: `warn`
-   `session.maintenance.pruneAfter`: `30d`
-   `session.maintenance.maxEntries`: `500`
-   `session.maintenance.rotateBytes`: `10mb`
-   `session.maintenance.resetArchiveRetention`: por defecto `pruneAfter` (`30d`)
-   `session.maintenance.maxDiskBytes`: sin configurar (deshabilitado)
-   `session.maintenance.highWaterBytes`: por defecto `80%` de `maxDiskBytes` cuando la presupuestación está habilitada

### Cómo funciona

El mantenimiento se ejecuta durante las escrituras en el almacén de sesiones, y puedes activarlo bajo demanda con `openclaw sessions cleanup`.

-   `mode: "warn"`: reporta lo que sería eliminado pero no muta entradas/transcripciones.
-   `mode: "enforce"`: aplica limpieza en este orden:
    1.  podar entradas obsoletas más antiguas que `pruneAfter`
    2.  limitar el conteo de entradas a `maxEntries` (las más antiguas primero)
    3.  archivar archivos de transcripción para entradas eliminadas que ya no están referenciadas
    4.  purgar archivos antiguos `*.deleted.` y `*.reset.` por política de retención
    5.  rotar `sessions.json` cuando excede `rotateBytes`
    6.  si `maxDiskBytes` está configurado, hacer cumplir el presupuesto de disco hacia `highWaterBytes` (artefactos más antiguos primero, luego sesiones más antiguas)

### Advertencia de rendimiento para almacenes grandes

Los almacenes de sesiones grandes son comunes en configuraciones de alto volumen. El trabajo de mantenimiento es trabajo en la ruta de escritura, por lo que almacenes muy grandes pueden aumentar la latencia de escritura. Lo que más aumenta el costo:

-   valores muy altos de `session.maintenance.maxEntries`
-   ventanas largas de `pruneAfter` que mantienen entradas obsoletas
-   muchos artefactos de transcripción/archivo en `~/.openclaw/agents//sessions/`
-   habilitar presupuestos de disco (`maxDiskBytes`) sin límites razonables de poda/capacidad

Qué hacer:

-   usa `mode: "enforce"` en producción para que el crecimiento esté acotado automáticamente
-   configura límites de tiempo y de conteo (`pruneAfter` + `maxEntries`), no solo uno
-   configura `maxDiskBytes` + `highWaterBytes` para límites superiores estrictos en despliegues grandes
-   mantén `highWaterBytes` significativamente por debajo de `maxDiskBytes` (por defecto es 80%)
-   ejecuta `openclaw sessions cleanup --dry-run --json` después de cambios de configuración para verificar el impacto proyectado antes de hacer cumplir
-   para sesiones activas frecuentes, pasa `--active-key` al ejecutar limpieza manual

### Ejemplos de personalización

Usa una política de cumplimiento conservadora:

```json
{
  session: {
    maintenance: {
      mode: "enforce",
      pruneAfter: "45d",
      maxEntries: 800,
      rotateBytes: "20mb",
      resetArchiveRetention: "14d",
    },
  },
}
```

Habilita un presupuesto de disco estricto para el directorio de sesiones:

```json
{
  session: {
    maintenance: {
      mode: "enforce",
      maxDiskBytes: "1gb",
      highWaterBytes: "800mb",
    },
  },
}
```

Ajusta para instalaciones más grandes (ejemplo):

```json
{
  session: {
    maintenance: {
      mode: "enforce",
      pruneAfter: "14d",
      maxEntries: 2000,
      rotateBytes: "25mb",
      maxDiskBytes: "2gb",
      highWaterBytes: "1.6gb",
    },
  },
}
```

Vista previa o forzar mantenimiento desde CLI:

```bash
openclaw sessions cleanup --dry-run
openclaw sessions cleanup --enforce
```

## Poda de sesión

OpenClaw recorta **resultados antiguos de herramientas** del contexto en memoria justo antes de las llamadas al LLM por defecto. Esto **no** reescribe el historial JSONL. Ver [/concepts/session-pruning](./session-pruning.md).

## Vaciamiento de memoria pre-compactación

Cuando una sesión se acerca a la compactación automática, OpenClaw puede ejecutar un **turno de vaciamiento de memoria silencioso** que recuerda al modelo que escriba notas duraderas en disco. Esto solo se ejecuta cuando el espacio de trabajo es escribible. Ver [Memoria](./memory.md) y [Compactación](./compaction.md).

## Mapeo de transportes → claves de sesión

-   Los chats directos siguen `session.dmScope` (por defecto `main`).
    -   `main`: `agent::` (continuidad a través de dispositivos/canales).
        -   Múltiples números de teléfono y canales pueden mapearse a la misma clave principal del agente; actúan como transportes hacia una conversación.
    -   `per-peer`: `agent::dm:`.
    -   `per-channel-peer`: `agent:::dm:`.
    -   `per-account-channel-peer`: `agent::::dm:` (accountId por defecto es `default`).
    -   Si `session.identityLinks` coincide con un id de par con prefijo de proveedor (por ejemplo `telegram:123`), la clave canónica reemplaza `` para que la misma persona comparta una sesión a través de canales.
-   Los chats de grupo aíslan el estado: `agent:::group:` (las salas/canales usan `agent:::channel:`).
    -   Los temas de foro de Telegram agregan `:topic:` al id del grupo para aislamiento.
    -   Las claves heredadas `group:` aún se reconocen para migración.
-   Los contextos entrantes aún pueden usar `group:`; el canal se infiere de `Provider` y se normaliza a la forma canónica `agent:::group:`.
-   Otras fuentes:
    -   Trabajos cron: `cron:<job.id>`
    -   Webhooks: `hook:` (a menos que sea configurado explícitamente por el hook)
    -   Ejecuciones de nodo: `node-`

## Ciclo de vida

-   Política de reinicio: las sesiones se reutilizan hasta que expiran, y la expiración se evalúa en el próximo mensaje entrante.
-   Reinicio diario: por defecto a **las 4:00 AM hora local en el host de la puerta de enlace**. Una sesión está obsoleta una vez que su última actualización es anterior a la hora de reinicio diario más reciente.
-   Reinicio por inactividad (opcional): `idleMinutes` agrega una ventana de inactividad deslizante. Cuando se configuran tanto reinicio diario como por inactividad, **cualquiera que expire primero** fuerza una nueva sesión.
-   Solo inactividad heredado: si configuras `session.idleMinutes` sin ninguna configuración `session.reset`/`resetByType`, OpenClaw permanece en modo solo inactividad para compatibilidad hacia atrás.
-   Anulaciones por tipo (opcional): `resetByType` te permite anular la política para sesiones `direct`, `group`, y `thread` (thread = hilos de Slack/Discord, temas de Telegram, hilos de Matrix cuando los proporciona el conector).
-   Anulaciones por canal (opcional): `resetByChannel` anula la política de reinicio para un canal (se aplica a todos los tipos de sesión para ese canal y tiene precedencia sobre `reset`/`resetByType`).
-   Disparadores de reinicio: exactamente `/new` o `/reset` (más cualquier extra en `resetTriggers`) inicia un nuevo id de sesión y pasa el resto del mensaje. `/new ` acepta un alias de modelo, `provider/model`, o nombre de proveedor (coincidencia aproximada) para configurar el modelo de la nueva sesión. Si `/new` o `/reset` se envían solos, OpenClaw ejecuta un turno corto de saludo "hola" para confirmar el reinicio.
-   Reinicio manual: elimina claves específicas del almacén o elimina la transcripción JSONL; el próximo mensaje las recrea.
-   Los trabajos cron aislados siempre crean un `sessionId` nuevo por ejecución (sin reutilización por inactividad).

## Política de envío (opcional)

Bloquea la entrega para tipos de sesión específicos sin listar ids individuales.

```json
{
  session: {
    sendPolicy: {
      rules: [
        { action: "deny", match: { channel: "discord", chatType: "group" } },
        { action: "deny", match: { keyPrefix: "cron:" } },
        // Coincide con la clave de sesión cruda (incluyendo el prefijo `agent:<id>:`).
        { action: "deny", match: { rawKeyPrefix: "agent:main:discord:" } },
      ],
      default: "allow",
    },
  },
}
```

Anulación en tiempo de ejecución (solo propietario):

-   `/send on` → permitir para esta sesión
-   `/send off` → denegar para esta sesión
-   `/send inherit` → borrar anulación y usar reglas de configuración Envía estos como mensajes independientes para que se registren.

## Configuración (ejemplo de renombre opcional)

```
// ~/.openclaw/openclaw.json
{
  session: {
    scope: "per-sender", // mantener claves de grupo separadas
    dmScope: "main", // Continuidad de DM (configurar per-channel-peer/per-account-channel-peer para bandejas de entrada compartidas)
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"],
    },
    reset: {
      // Valores por defecto: mode=daily, atHour=4 (hora local del host de la puerta de enlace).
      // Si también configuras idleMinutes, gana el que expire primero.
      mode: "daily",
      atHour: 4,
      idleMinutes: 120,
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      direct: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 },
    },
    resetByChannel: {
      discord: { mode: "idle", idleMinutes: 10080 },
    },
    resetTriggers: ["/new", "/reset"],
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    mainKey: "main",
  },
}
```

## Inspección

-   `openclaw status` — muestra la ruta del almacén y sesiones recientes.
-   `openclaw sessions --json` — vuelca cada entrada (filtrar con `--active `).
-   `openclaw gateway call sessions.list --params '{}'` — obtén sesiones de la puerta de enlace en ejecución (usa `--url`/`--token` para acceso a puerta de enlace remota).
-   Envía `/status` como un mensaje independiente en el chat para ver si el agente es accesible, cuánto del contexto de sesión se usa, los toggles de pensamiento/verbose actuales, y cuándo se actualizaron tus credenciales web de WhatsApp por última vez (ayuda a detectar necesidades de reenlace).
-   Envía `/context list` o `/context detail` para ver qué hay en el prompt del sistema y los archivos del espacio de trabajo inyectados (y los mayores contribuyentes al contexto).
-   Envía `/stop` (o frases de abortar independientes como `stop`, `stop action`, `stop run`, `stop openclaw`) para abortar la ejecución actual, borrar seguimientos en cola para esa sesión y detener cualquier ejecución de sub-agente generada desde ella (la respuesta incluye el conteo detenido).
-   Envía `/compact` (instrucciones opcionales) como un mensaje independiente para resumir contexto más antiguo y liberar espacio en la ventana. Ver [/concepts/compaction](./compaction.md).
-   Las transcripciones JSONL se pueden abrir directamente para revisar turnos completos.

## Consejos

-   Mantén la clave principal dedicada al tráfico 1:1; deja que los grupos mantengan sus propias claves.
-   Al automatizar la limpieza, elimina claves individuales en lugar de todo el almacén para preservar el contexto en otros lugares.

## Metadatos de origen de sesión

Cada entrada de sesión registra de dónde vino (mejor esfuerzo) en `origin`:

-   `label`: etiqueta humana (resuelta a partir de la etiqueta de conversación + asunto/canal del grupo)
-   `provider`: id de canal normalizado (incluyendo extensiones)
-   `from`/`to`: ids de enrutamiento crudos del sobre entrante
-   `accountId`: id de cuenta del proveedor (cuando es multi-cuenta)
-   `threadId`: id de hilo/tema cuando el canal lo soporta Los campos de origen se completan para mensajes directos, canales y grupos. Si un conector solo actualiza el enrutamiento de entrega (por ejemplo, para mantener fresca una sesión principal de DM), aún debe proporcionar contexto entrante para que la sesión mantenga sus metadatos explicativos. Las extensiones pueden hacer esto enviando `ConversationLabel`, `GroupSubject`, `GroupChannel`, `GroupSpace`, y `SenderName` en el contexto entrante y llamando a `recordSessionMetaFromInbound` (o pasando el mismo contexto a `updateLastRoute`).

[Inicialización](../start/bootstrapping.md)[Poda de Sesión](./session-pruning.md)