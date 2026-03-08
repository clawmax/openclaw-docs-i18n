

  Habilidades

  
# Comandos de Barra

Los comandos son manejados por el Gateway. La mayoría de los comandos deben enviarse como un mensaje **independiente** que comience con `/`. El comando exclusivo del host en el chat bash usa `! ` (con `/bash ` como alias). Hay dos sistemas relacionados:

-   **Comandos**: mensajes independientes `/...`.
-   **Directivas**: `/think`, `/verbose`, `/reasoning`, `/elevated`, `/exec`, `/model`, `/queue`.
    -   Las directivas se eliminan del mensaje antes de que el modelo lo vea.
    -   En mensajes de chat normales (no solo de directivas), se tratan como "sugerencias en línea" y **no** persisten en la configuración de la sesión.
    -   En mensajes solo de directivas (el mensaje contiene solo directivas), persisten en la sesión y responden con un acuse de recibo.
    -   Las directivas solo se aplican para **remitentes autorizados**. Si `commands.allowFrom` está configurado, es la única lista de permitidos utilizada; de lo contrario, la autorización proviene de las listas de permitidos/emparejamiento del canal más `commands.useAccessGroups`. Los remitentes no autorizados ven las directivas tratadas como texto plano.

También hay algunos **atajos en línea** (solo para remitentes autorizados/en lista de permitidos): `/help`, `/commands`, `/status`, `/whoami` (`/id`). Se ejecutan inmediatamente, se eliminan antes de que el modelo vea el mensaje, y el texto restante continúa a través del flujo normal.

## Config

```json
{
  commands: {
    native: "auto",
    nativeSkills: "auto",
    text: true,
    bash: false,
    bashForegroundMs: 2000,
    config: false,
    debug: false,
    restart: false,
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

-   `commands.text` (por defecto `true`) habilita el análisis de `/...` en mensajes de chat.
    -   En superficies sin comandos nativos (WhatsApp/WebChat/Signal/iMessage/Google Chat/MS Teams), los comandos de texto aún funcionan incluso si configuras esto en `false`.
-   `commands.native` (por defecto `"auto"`) registra comandos nativos.
    -   Auto: activado para Discord/Telegram; desactivado para Slack (hasta que agregues comandos de barra); ignorado para proveedores sin soporte nativo.
    -   Configura `channels.discord.commands.native`, `channels.telegram.commands.native`, o `channels.slack.commands.native` para anular por proveedor (bool o `"auto"`).
    -   `false` borra los comandos registrados previamente en Discord/Telegram al inicio. Los comandos de Slack se gestionan en la aplicación de Slack y no se eliminan automáticamente.
-   `commands.nativeSkills` (por defecto `"auto"`) registra comandos de **habilidad** de forma nativa cuando es compatible.
    -   Auto: activado para Discord/Telegram; desactivado para Slack (Slack requiere crear un comando de barra por habilidad).
    -   Configura `channels.discord.commands.nativeSkills`, `channels.telegram.commands.nativeSkills`, o `channels.slack.commands.nativeSkills` para anular por proveedor (bool o `"auto"`).
-   `commands.bash` (por defecto `false`) habilita `! ` para ejecutar comandos de shell del host (`/bash ` es un alias; requiere listas de permitidos `tools.elevated`).
-   `commands.bashForegroundMs` (por defecto `2000`) controla cuánto tiempo espera bash antes de cambiar al modo en segundo plano (`0` pasa a segundo plano inmediatamente).
-   `commands.config` (por defecto `false`) habilita `/config` (lee/escribe `openclaw.json`).
-   `commands.debug` (por defecto `false`) habilita `/debug` (anulaciones solo en tiempo de ejecución).
-   `commands.allowFrom` (opcional) establece una lista de permitidos por proveedor para la autorización de comandos. Cuando está configurada, es la única fuente de autorización para comandos y directivas (se ignoran las listas de permitidos/emparejamiento del canal y `commands.useAccessGroups`). Usa `"*"` para un valor predeterminado global; las claves específicas del proveedor lo anulan.
-   `commands.useAccessGroups` (por defecto `true`) aplica listas de permitidos/políticas para comandos cuando `commands.allowFrom` no está configurado.

## Lista de comandos

Texto + nativo (cuando está habilitado):

-   `/help`
-   `/commands`
-   `/skill  [entrada]` (ejecuta una habilidad por nombre)
-   `/status` (muestra el estado actual; incluye uso/cuota del proveedor para el proveedor de modelo actual cuando está disponible)
-   `/allowlist` (lista/agrega/elimina entradas de la lista de permitidos)
-   `/approve  allow-once|allow-always|deny` (resuelve solicitudes de aprobación de exec)
-   `/context [list|detail|json]` (explica el "contexto"; `detail` muestra tamaño por archivo + por herramienta + por habilidad + prompt del sistema)
-   `/export-session [ruta]` (alias: `/export`) (exporta la sesión actual a HTML con el prompt del sistema completo)
-   `/whoami` (muestra tu id de remitente; alias: `/id`)
-   `/session idle <duración|off>` (gestiona la auto-desactivación por inactividad para enlaces de hilos enfocados)
-   `/session max-age <duración|off>` (gestiona la auto-desactivación por edad máxima para enlaces de hilos enfocados)
-   `/subagents list|kill|log|info|send|steer|spawn` (inspecciona, controla o genera ejecuciones de sub-agentes para la sesión actual)
-   `/acp spawn|cancel|steer|close|status|set-mode|set|cwd|permissions|timeout|model|reset-options|doctor|install|sessions` (inspecciona y controla sesiones de tiempo de ejecución de ACP)
-   `/agents` (lista agentes vinculados a hilos para esta sesión)
-   `/focus ` (Discord: vincula este hilo, o un nuevo hilo, a un objetivo de sesión/subagente)
-   `/unfocus` (Discord: elimina el enlace de hilo actual)
-   `/kill <id|#|all>` (aborta inmediatamente uno o todos los sub-agentes en ejecución para esta sesión; sin mensaje de confirmación)
-   `/steer <id|#> ` (dirige un sub-agente en ejecución inmediatamente: en ejecución cuando es posible, de lo contrario aborta el trabajo actual y reinicia con el mensaje de dirección)
-   `/tell <id|#> ` (alias para `/steer`)
-   `/config show|get|set|unset` (persiste la configuración en disco, solo para el propietario; requiere `commands.config: true`)
-   `/debug show|set|unset|reset` (anulaciones en tiempo de ejecución, solo para el propietario; requiere `commands.debug: true`)
-   `/usage off|tokens|full|cost` (pie de página de uso por respuesta o resumen de costos local)
-   `/tts off|always|inbound|tagged|status|provider|limit|summary|audio` (controla TTS; ver [/tts](../tts.md))
    -   Discord: el comando nativo es `/voice` (Discord reserva `/tts`); el texto `/tts` aún funciona.
-   `/stop`
-   `/restart`
-   `/dock-telegram` (alias: `/dock_telegram`) (cambia las respuestas a Telegram)
-   `/dock-discord` (alias: `/dock_discord`) (cambia las respuestas a Discord)
-   `/dock-slack` (alias: `/dock_slack`) (cambia las respuestas a Slack)
-   `/activation mention|always` (solo grupos)
-   `/send on|off|inherit` (solo para el propietario)
-   `/reset` o `/new [modelo]` (sugerencia de modelo opcional; el resto se pasa)
-   `/think <off|minimal|low|medium|high|xhigh>` (elecciones dinámicas por modelo/proveedor; alias: `/thinking`, `/t`)
-   `/verbose on|full|off` (alias: `/v`)
-   `/reasoning on|off|stream` (alias: `/reason`; cuando está activado, envía un mensaje separado con prefijo `Reasoning:`; `stream` = solo borrador de Telegram)
-   `/elevated on|off|ask|full` (alias: `/elev`; `full` omite aprobaciones de exec)
-   `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=` (envía `/exec` para mostrar el actual)
-   `/model ` (alias: `/models`; o `/` de `agents.defaults.models.*.alias`)
-   `/queue ` (más opciones como `debounce:2s cap:25 drop:summarize`; envía `/queue` para ver la configuración actual)
-   `/bash ` (solo host; alias para `! `; requiere `commands.bash: true` + listas de permitidos `tools.elevated`)

Solo texto:

-   `/compact [instrucciones]` (ver [/concepts/compaction](../concepts/compaction.md))
-   `! ` (solo host; uno a la vez; usa `!poll` + `!stop` para trabajos de larga duración)
-   `!poll` (verifica la salida / estado; acepta `sessionId` opcional; `/bash poll` también funciona)
-   `!stop` (detiene el trabajo bash en ejecución; acepta `sessionId` opcional; `/bash stop` también funciona)

Notas:

-   Los comandos aceptan un `:` opcional entre el comando y los argumentos (ej. `/think: high`, `/send: on`, `/help:`).
-   `/new ` acepta un alias de modelo, `proveedor/modelo`, o un nombre de proveedor (coincidencia aproximada); si no hay coincidencia, el texto se trata como el cuerpo del mensaje.
-   Para un desglose completo del uso del proveedor, usa `openclaw status --usage`.
-   `/allowlist add|remove` requiere `commands.config=true` y respeta `configWrites` del canal.
-   `/usage` controla el pie de página de uso por respuesta; `/usage cost` imprime un resumen de costos local desde los registros de sesión de OpenClaw.
-   `/restart` está habilitado por defecto; configura `commands.restart: false` para deshabilitarlo.
-   Comando nativo solo para Discord: `/vc join|leave|status` controla los canales de voz (requiere `channels.discord.voice` y comandos nativos; no disponible como texto).
-   Los comandos de enlace de hilos de Discord (`/focus`, `/unfocus`, `/agents`, `/session idle`, `/session max-age`) requieren que los enlaces de hilos efectivos estén habilitados (`session.threadBindings.enabled` y/o `channels.discord.threadBindings.enabled`).
-   Referencia de comandos ACP y comportamiento en tiempo de ejecución: [Agentes ACP](./acp-agents.md).
-   `/verbose` está destinado a depuración y visibilidad extra; mantenlo **desactivado** en uso normal.
-   Los resúmenes de fallos de herramientas aún se muestran cuando son relevantes, pero el texto detallado del fallo solo se incluye cuando `/verbose` está `on` o `full`.
-   `/reasoning` (y `/verbose`) son riesgosos en configuraciones de grupo: pueden revelar razonamientos internos o salidas de herramientas que no pretendías exponer. Prefiere dejarlos desactivados, especialmente en chats grupales.
-   **Ruta rápida:** los mensajes solo de comandos de remitentes en lista de permitidos se manejan inmediatamente (omitir cola + modelo).
-   **Puerta de mención de grupo:** los mensajes solo de comandos de remitentes en lista de permitidos omiten los requisitos de mención.
-   **Atajos en línea (solo remitentes en lista de permitidos):** ciertos comandos también funcionan cuando están incrustados en un mensaje normal y se eliminan antes de que el modelo vea el texto restante.
    -   Ejemplo: `hey /status` desencadena una respuesta de estado, y el texto restante continúa a través del flujo normal.
-   Actualmente: `/help`, `/commands`, `/status`, `/whoami` (`/id`).
-   Los mensajes solo de comandos no autorizados se ignoran silenciosamente, y los tokens `/...` en línea se tratan como texto plano.
-   **Comandos de habilidad:** las habilidades `user-invocable` se exponen como comandos de barra. Los nombres se sanitizan a `a-z0-9_` (máx. 32 caracteres); las colisiones obtienen sufijos numéricos (ej. `_2`).
    -   `/skill  [entrada]` ejecuta una habilidad por nombre (útil cuando los límites de comandos nativos impiden comandos por habilidad).
    -   Por defecto, los comandos de habilidad se reenvían al modelo como una solicitud normal.
    -   Las habilidades pueden declarar opcionalmente `command-dispatch: tool` para enrutar el comando directamente a una herramienta (determinista, sin modelo).
    -   Ejemplo: `/prose` (plugin OpenProse) — ver [OpenProse](../prose.md).
-   **Argumentos de comandos nativos:** Discord usa autocompletar para opciones dinámicas (y menús de botones cuando omites argumentos requeridos). Telegram y Slack muestran un menú de botones cuando un comando admite opciones y omites el argumento.

## Superficies de uso (qué se muestra dónde)

-   **Uso/cuota del proveedor** (ejemplo: "Claude 80% restante") aparece en `/status` para el proveedor de modelo actual cuando el seguimiento de uso está habilitado.
-   **Tokens/costo por respuesta** es controlado por `/usage off|tokens|full` (agregado a las respuestas normales).
-   `/model status` trata sobre **modelos/auth/endpoints**, no sobre uso.

## Selección de modelo (/model)

`/model` se implementa como una directiva. Ejemplos:

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model opus@anthropic:default
/model status
```

Notas:

-   `/model` y `/model list` muestran un selector compacto y numerado (familia de modelo + proveedores disponibles).
-   En Discord, `/model` y `/models` abren un selector interactivo con desplegables de proveedor y modelo más un paso de Enviar.
-   `/model <#>` selecciona de ese selector (y prefiere el proveedor actual cuando es posible).
-   `/model status` muestra la vista detallada, incluyendo el endpoint del proveedor configurado (`baseUrl`) y el modo API (`api`) cuando está disponible.

## Anulaciones de depuración

`/debug` te permite configurar anulaciones de configuración **solo en tiempo de ejecución** (memoria, no disco). Solo para el propietario. Deshabilitado por defecto; habilita con `commands.debug: true`. Ejemplos:

```bash
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug set channels.whatsapp.allowFrom=["+1555","+4477"]
/debug unset messages.responsePrefix
/debug reset
```

Notas:

-   Las anulaciones se aplican inmediatamente a nuevas lecturas de configuración, pero **no** escriben en `openclaw.json`.
-   Usa `/debug reset` para borrar todas las anulaciones y volver a la configuración en disco.

## Actualizaciones de configuración

`/config` escribe en tu configuración en disco (`openclaw.json`). Solo para el propietario. Deshabilitado por defecto; habilita con `commands.config: true`. Ejemplos:

```bash
/config show
/config show messages.responsePrefix
/config get messages.responsePrefix
/config set messages.responsePrefix="[openclaw]"
/config unset messages.responsePrefix
```

Notas:

-   La configuración se valida antes de escribir; los cambios inválidos son rechazados.
-   Las actualizaciones de `/config` persisten a través de reinicios.

## Notas de superficie

-   **Comandos de texto** se ejecutan en la sesión de chat normal (los MD comparten `main`, los grupos tienen su propia sesión).
-   **Comandos nativos** usan sesiones aisladas:
    -   Discord: `agent::discord:slash:`
    -   Slack: `agent::slack:slash:` (prefijo configurable via `channels.slack.slashCommand.sessionPrefix`)
    -   Telegram: `telegram:slash:` (apunta a la sesión de chat via `CommandTargetSessionKey`)
-   **`/stop`** apunta a la sesión de chat activa para que pueda abortar la ejecución actual.
-   **Slack:** `channels.slack.slashCommand` aún es compatible para un solo comando estilo `/openclaw`. Si habilitas `commands.native`, debes crear un comando de barra de Slack por comando incorporado (mismos nombres que `/help`). Los menús de argumentos de comandos para Slack se entregan como botones efímeros de Block Kit.
    -   Excepción nativa de Slack: registra `/agentstatus` (no `/status`) porque Slack reserva `/status`. El texto `/status` aún funciona en mensajes de Slack.

[Creando Habilidades](./creating-skills.md)[Habilidades](./skills.md)