

  Comandos CLI

  
# Referencia de la CLI

Esta página describe el comportamiento actual de la CLI. Si los comandos cambian, actualiza este documento.

## Páginas de comandos

-   [`setup`](./cli/setup.md)
-   [`onboard`](./cli/onboard.md)
-   [`configure`](./cli/configure.md)
-   [`config`](./cli/config.md)
-   [`completion`](./cli/completion.md)
-   [`doctor`](./cli/doctor.md)
-   [`dashboard`](./cli/dashboard.md)
-   [`reset`](./cli/reset.md)
-   [`uninstall`](./cli/uninstall.md)
-   [`update`](./cli/update.md)
-   [`message`](./cli/message.md)
-   [`agent`](./cli/agent.md)
-   [`agents`](./cli/agents.md)
-   [`acp`](./cli/acp.md)
-   [`status`](./cli/status.md)
-   [`health`](./cli/health.md)
-   [`sessions`](./cli/sessions.md)
-   [`gateway`](./cli/gateway.md)
-   [`logs`](./cli/logs.md)
-   [`system`](./cli/system.md)
-   [`models`](./cli/models.md)
-   [`memory`](./cli/memory.md)
-   [`directory`](./cli/directory.md)
-   [`nodes`](./cli/nodes.md)
-   [`devices`](./cli/devices.md)
-   [`node`](./cli/node.md)
-   [`approvals`](./cli/approvals.md)
-   [`sandbox`](./cli/sandbox.md)
-   [`tui`](./cli/tui.md)
-   [`browser`](./cli/browser.md)
-   [`cron`](./cli/cron.md)
-   [`dns`](./cli/dns.md)
-   [`docs`](./cli/docs.md)
-   [`hooks`](./cli/hooks.md)
-   [`webhooks`](./cli/webhooks.md)
-   [`pairing`](./cli/pairing.md)
-   [`qr`](./cli/qr.md)
-   [`plugins`](./cli/plugins.md) (comandos de plugin)
-   [`channels`](./cli/channels.md)
-   [`security`](./cli/security.md)
-   [`secrets`](./cli/secrets.md)
-   [`skills`](./cli/skills.md)
-   [`daemon`](./cli/daemon.md) (alias heredado para comandos del servicio gateway)
-   [`clawbot`](./cli/clawbot.md) (espacio de nombres de alias heredado)
-   [`voicecall`](./cli/voicecall.md) (plugin; si está instalado)

## Banderas globales

-   `--dev`: aísla el estado bajo `~/.openclaw-dev` y cambia los puertos predeterminados.
-   `--profile `: aísla el estado bajo `~/.openclaw-`.
-   `--no-color`: deshabilita colores ANSI.
-   `--update`: abreviatura para `openclaw update` (solo instalaciones desde fuente).
-   `-V`, `--version`, `-v`: imprime la versión y sale.

## Estilo de salida

-   Los colores ANSI e indicadores de progreso solo se renderizan en sesiones TTY.
-   Los hipervínculos OSC-8 se renderizan como enlaces clicables en terminales compatibles; de lo contrario, mostramos URLs simples.
-   `--json` (y `--plain` donde se admita) desactiva el estilo para una salida limpia.
-   `--no-color` desactiva el estilo ANSI; también se respeta `NO_COLOR=1`.
-   Los comandos de larga duración muestran un indicador de progreso (OSC 9;4 cuando se admite).

## Paleta de colores

OpenClaw usa una paleta de colores "lobster" para la salida de la CLI.

-   `accent` (#FF5A2D): encabezados, etiquetas, resaltados primarios.
-   `accentBright` (#FF7A3D): nombres de comandos, énfasis.
-   `accentDim` (#D14A22): texto de resaltado secundario.
-   `info` (#FF8A5B): valores informativos.
-   `success` (#2FBF71): estados de éxito.
-   `warn` (#FFB020): advertencias, alternativas, atención.
-   `error` (#E23D2D): errores, fallos.
-   `muted` (#8B7F77): desénfasis, metadatos.

Fuente de verdad de la paleta: `src/terminal/palette.ts` (también conocida como "lobster seam").

## Árbol de comandos

```bash
openclaw [--dev] [--profile <name>] <command>
  setup
  onboard
  configure
  config
    get
    set
    unset
  completion
  doctor
  dashboard
  security
    audit
  secrets
    reload
    migrate
  reset
  uninstall
  update
  channels
    list
    status
    logs
    add
    remove
    login
    logout
  directory
  skills
    list
    info
    check
  plugins
    list
    info
    install
    enable
    disable
    doctor
  memory
    status
    index
    search
  message
  agent
  agents
    list
    add
    delete
  acp
  status
  health
  sessions
  gateway
    call
    health
    status
    probe
    discover
    install
    uninstall
    start
    stop
    restart
    run
  daemon
    status
    install
    uninstall
    start
    stop
    restart
  logs
  system
    event
    heartbeat last|enable|disable
    presence
  models
    list
    status
    set
    set-image
    aliases list|add|remove
    fallbacks list|add|remove|clear
    image-fallbacks list|add|remove|clear
    scan
    auth add|setup-token|paste-token
    auth order get|set|clear
  sandbox
    list
    recreate
    explain
  cron
    status
    list
    add
    edit
    rm
    enable
    disable
    runs
    run
  nodes
  devices
  node
    run
    status
    install
    uninstall
    start
    stop
    restart
  approvals
    get
    set
    allowlist add|remove
  browser
    status
    start
    stop
    reset-profile
    tabs
    open
    focus
    close
    profiles
    create-profile
    delete-profile
    screenshot
    snapshot
    navigate
    resize
    click
    type
    press
    hover
    drag
    select
    upload
    fill
    dialog
    wait
    evaluate
    console
    pdf
  hooks
    list
    info
    check
    enable
    disable
    install
    update
  webhooks
    gmail setup|run
  pairing
    list
    approve
  qr
  clawbot
    qr
  docs
  dns
    setup
  tui
```

Nota: los plugins pueden agregar comandos de nivel superior adicionales (por ejemplo `openclaw voicecall`).

## Seguridad

-   `openclaw security audit` — audita la configuración + el estado local en busca de errores de seguridad comunes.
-   `openclaw security audit --deep` — sondeo en vivo del Gateway con el mejor esfuerzo.
-   `openclaw security audit --fix` — ajusta los valores predeterminados seguros y cambia permisos (chmod) del estado/configuración.

## Secretos

-   `openclaw secrets reload` — vuelve a resolver las referencias y cambia atómicamente la instantánea del entorno de ejecución.
-   `openclaw secrets audit` — escanea en busca de residuos de texto plano, referencias no resueltas y desviación de precedencia.
-   `openclaw secrets configure` — asistente interactivo para configuración del proveedor + mapeo de SecretRef + preflight/aplicación.
-   `openclaw secrets apply --from <plan.json>` — aplica un plan generado previamente (se admite `--dry-run`).

## Plugins

Gestiona extensiones y su configuración:

-   `openclaw plugins list` — descubre plugins (usa `--json` para salida de máquina).
-   `openclaw plugins info ` — muestra detalles de un plugin.
-   `openclaw plugins install <path|.tgz|npm-spec>` — instala un plugin (o agrega una ruta de plugin a `plugins.load.paths`).
-   `openclaw plugins enable ` / `disable ` — activa/desactiva `plugins.entries..enabled`.
-   `openclaw plugins doctor` — reporta errores de carga de plugins.

La mayoría de los cambios en los plugins requieren un reinicio del gateway. Ver [/plugin](./tools/plugin.md).

## Memoria

Búsqueda vectorial sobre `MEMORY.md` + `memory/*.md`:

-   `openclaw memory status` — muestra estadísticas del índice.
-   `openclaw memory index` — reindexa los archivos de memoria.
-   `openclaw memory search ""` (o `--query ""`) — búsqueda semántica sobre la memoria.

## Comandos de barra diagonal (slash) en el chat

Los mensajes de chat admiten comandos `/...` (texto y nativos). Ver [/tools/slash-commands](./tools/slash-commands.md). Destacados:

-   `/status` para diagnósticos rápidos.
-   `/config` para cambios de configuración persistente.
-   `/debug` para anulaciones de configuración solo en tiempo de ejecución (en memoria, no en disco; requiere `commands.debug: true`).

## Configuración + incorporación

### setup

Inicializa la configuración + el espacio de trabajo. Opciones:

-   `--workspace `: ruta del espacio de trabajo del agente (predeterminado `~/.openclaw/workspace`).
-   `--wizard`: ejecuta el asistente de incorporación.
-   `--non-interactive`: ejecuta el asistente sin preguntas.
-   `--mode <local|remote>`: modo del asistente.
-   `--remote-url `: URL del Gateway remoto.
-   `--remote-token `: token del Gateway remoto.

El asistente se ejecuta automáticamente cuando hay presentes banderas del asistente (`--non-interactive`, `--mode`, `--remote-url`, `--remote-token`).

### onboard

Asistente interactivo para configurar el gateway, espacio de trabajo y habilidades. Opciones:

-   `--workspace `
-   `--reset` (restablece configuración + credenciales + sesiones antes del asistente)
-   `--reset-scope <config|config+creds+sessions|full>` (predeterminado `config+creds+sessions`; usa `full` para también eliminar el espacio de trabajo)
-   `--non-interactive`
-   `--mode <local|remote>`
-   `--flow <quickstart|advanced|manual>` (manual es un alias para advanced)
-   `--auth-choice <setup-token|token|chutes|openai-codex|openai-api-key|openrouter-api-key|ai-gateway-api-key|moonshot-api-key|moonshot-api-key-cn|kimi-code-api-key|synthetic-api-key|venice-api-key|gemini-api-key|zai-api-key|mistral-api-key|apiKey|minimax-api|minimax-api-lightning|opencode-zen|custom-api-key|skip>`
-   `--token-provider ` (no interactivo; usado con `--auth-choice token`)
-   `--token ` (no interactivo; usado con `--auth-choice token`)
-   `--token-profile-id ` (no interactivo; predeterminado: `:manual`)
-   `--token-expires-in <duración>` (no interactivo; ej. `365d`, `12h`)
-   `--secret-input-mode <plaintext|ref>` (predeterminado `plaintext`; usa `ref` para almacenar referencias de entorno predeterminadas del proveedor en lugar de claves en texto plano)
-   `--anthropic-api-key `
-   `--openai-api-key `
-   `--mistral-api-key `
-   `--openrouter-api-key `
-   `--ai-gateway-api-key `
-   `--moonshot-api-key `
-   `--kimi-code-api-key `
-   `--gemini-api-key `
-   `--zai-api-key `
-   `--minimax-api-key `
-   `--opencode-zen-api-key `
-   `--custom-base-url ` (no interactivo; usado con `--auth-choice custom-api-key`)
-   `--custom-model-id ` (no interactivo; usado con `--auth-choice custom-api-key`)
-   `--custom-api-key ` (no interactivo; opcional; usado con `--auth-choice custom-api-key`; recurre a `CUSTOM_API_KEY` cuando se omite)
-   `--custom-provider-id ` (no interactivo; id de proveedor personalizado opcional)
-   `--custom-compatibility <openai|anthropic>` (no interactivo; opcional; predeterminado `openai`)
-   `--gateway-port `
-   `--gateway-bind <loopback|lan|tailnet|auto|custom>`
-   `--gateway-auth <token|password>`
-   `--gateway-token `
-   `--gateway-token-ref-env ` (no interactivo; almacena `gateway.auth.token` como un SecretRef de entorno; requiere que la variable de entorno esté configurada; no se puede combinar con `--gateway-token`)
-   `--gateway-password <contraseña>`
-   `--remote-url `
-   `--remote-token `
-   `--tailscale <off|serve|funnel>`
-   `--tailscale-reset-on-exit`
-   `--install-daemon`
-   `--no-install-daemon` (alias: `--skip-daemon`)
-   `--daemon-runtime <node|bun>`
-   `--skip-channels`
-   `--skip-skills`
-   `--skip-health`
-   `--skip-ui`
-   `--node-manager <npm|pnpm|bun>` (se recomienda pnpm; bun no se recomienda para el entorno de ejecución del Gateway)
-   `--json`

### configure

Asistente de configuración interactivo (modelos, canales, habilidades, gateway).

### config

Asistentes de configuración no interactivos (get/set/unset/file/validate). Ejecutar `openclaw config` sin subcomando inicia el asistente. Subcomandos:

-   `config get `: imprime un valor de configuración (ruta con puntos/corchetes).
-   `config set  `: establece un valor (JSON5 o cadena cruda).
-   `config unset `: elimina un valor.
-   `config file`: imprime la ruta del archivo de configuración activo.
-   `config validate`: valida la configuración actual contra el esquema sin iniciar el gateway.
-   `config validate --json`: emite salida JSON legible por máquina.

### doctor

Comprobaciones de salud + correcciones rápidas (configuración + gateway + servicios heredados). Opciones:

-   `--no-workspace-suggestions`: deshabilita sugerencias de memoria del espacio de trabajo.
-   `--yes`: acepta los valores predeterminados sin preguntar (sin interfaz).
-   `--non-interactive`: omite preguntas; aplica solo migraciones seguras.
-   `--deep`: escanea servicios del sistema en busca de instalaciones adicionales del gateway.

## Asistentes de canales

### channels

Gestiona cuentas de canales de chat (WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (plugin)/Signal/iMessage/MS Teams). Subcomandos:

-   `channels list`: muestra los canales configurados y los perfiles de autenticación.
-   `channels status`: verifica la accesibilidad del gateway y la salud del canal (`--probe` ejecuta comprobaciones adicionales; usa `openclaw health` o `openclaw status --deep` para sondeos de salud del gateway).
-   Consejo: `channels status` imprime advertencias con sugerencias de corrección cuando puede detectar configuraciones erróneas comunes (luego te dirige a `openclaw doctor`).
-   `channels logs`: muestra registros recientes del canal desde el archivo de registro del gateway.
-   `channels add`: configuración estilo asistente cuando no se pasan banderas; las banderas cambian al modo no interactivo.
    -   Al agregar una cuenta no predeterminada a un canal que aún usa la configuración de nivel superior de cuenta única, OpenClaw mueve los valores con alcance de cuenta a `channels..accounts.default` antes de escribir la nueva cuenta.
    -   `channels add` no interactivo no crea/actualiza automáticamente los enlaces; los enlaces solo de canal continúan coincidiendo con la cuenta predeterminada.
-   `channels remove`: deshabilita por defecto; pasa `--delete` para eliminar entradas de configuración sin preguntas.
-   `channels login`: inicio de sesión interactivo en el canal (solo WhatsApp Web).
-   `channels logout`: cierra la sesión de un canal (si se admite).

Opciones comunes:

-   `--channel `: `whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams`
-   `--account `: id de cuenta del canal (predeterminado `default`)
-   `--name `: nombre para mostrar de la cuenta

Opciones de `channels login`:

-   `--channel ` (predeterminado `whatsapp`; admite `whatsapp`/`web`)
-   `--account `
-   `--verbose`

Opciones de `channels logout`:

-   `--channel ` (predeterminado `whatsapp`)
-   `--account `

Opciones de `channels list`:

-   `--no-usage`: omite instantáneas de uso/cuota del proveedor de modelos (solo respaldado por OAuth/API).
-   `--json`: salida JSON (incluye uso a menos que se establezca `--no-usage`).

Opciones de `channels logs`:

-   `--channel <nombre|all>` (predeterminado `all`)
-   `--lines ` (predeterminado `200`)
-   `--json`

Más detalles: [/concepts/oauth](./concepts/oauth.md) Ejemplos:

```bash
openclaw channels add --channel telegram --account alerts --name "Alerts Bot" --token $TELEGRAM_BOT_TOKEN
openclaw channels add --channel discord --account work --name "Work Bot" --token $DISCORD_BOT_TOKEN
openclaw channels remove --channel discord --account work --delete
openclaw channels status --probe
openclaw status --deep
```

### skills

Lista e inspecciona las habilidades disponibles más información de preparación. Subcomandos:

-   `skills list`: lista habilidades (predeterminado cuando no hay subcomando).
-   `skills info `: muestra detalles de una habilidad.
-   `skills check`: resumen de habilidades listas vs requisitos faltantes.

Opciones:

-   `--eligible`: muestra solo habilidades listas.
-   `--json`: salida JSON (sin estilo).
-   `-v`, `--verbose`: incluye detalles de requisitos faltantes.

Consejo: usa `npx clawhub` para buscar, instalar y sincronizar habilidades.

### pairing

Aprueba solicitudes de emparejamiento por DM en todos los canales. Subcomandos:

-   `pairing list [channel] [--channel ] [--account ] [--json]`
-   `pairing approve  <código> [--account ] [--notify]`
-   `pairing approve --channel  [--account ] <código> [--notify]`

### devices

Gestiona entradas de emparejamiento de dispositivos del gateway y tokens de dispositivo por rol. Subcomandos:

-   `devices list [--json]`
-   `devices approve [requestId] [--latest]`
-   `devices reject `
-   `devices remove `
-   `devices clear --yes [--pending]`
-   `devices rotate --device  --role  [--scope <alcance...>]`
-   `devices revoke --device  --role `

### webhooks gmail

Configuración + ejecutor de hook Gmail Pub/Sub. Ver [/automation/gmail-pubsub](./automation/gmail-pubsub.md). Subcomandos:

-   `webhooks gmail setup` (requiere `--account `; admite `--project`, `--topic`, `--subscription`, `--label`, `--hook-url`, `--hook-token`, `--push-token`, `--bind`, `--port`, `--path`, `--include-body`, `--max-bytes`, `--renew-minutes`, `--tailscale`, `--tailscale-path`, `--tailscale-target`, `--push-endpoint`, `--json`)
-   `webhooks gmail run` (anulaciones en tiempo de ejecución para las mismas banderas)

### dns setup

Asistente DNS de descubrimiento de área amplia (CoreDNS + Tailscale). Ver [/gateway/discovery](./gateway/discovery.md). Opciones:

-   `--apply`: instala/actualiza la configuración de CoreDNS (requiere sudo; solo macOS).

## Mensajería + agente

### message

Mensajería saliente unificada + acciones de canal. Ver: [/cli/message](./cli/message.md) Subcomandos:

-   `message send|poll|react|reactions|read|edit|delete|pin|unpin|pins|permissions|search|timeout|kick|ban`
-   `message thread <create|list|reply>`
-   `message emoji <list|upload>`
-   `message sticker <send|upload>`
-   `message role <info|add|remove>`
-   `message channel <info|list>`
-   `message member info`
-   `message voice status`
-   `message event <list|create>`

Ejemplos:

-   `openclaw message send --target +15555550123 --message "Hola"`
-   `openclaw message poll --channel discord --target channel:123 --poll-question "¿Snack?" --poll-option Pizza --poll-option Sushi`

### agent

Ejecuta un turno de agente a través del Gateway (o `--local` integrado). Requerido:

-   `--message `

Opciones:

-   `--to ` (para clave de sesión y entrega opcional)
-   `--session-id `
-   `--thinking <off|minimal|low|medium|high|xhigh>` (solo modelos GPT-5.2 + Codex)
-   `--verbose <on|full|off>`
-   `--channel <whatsapp|telegram|discord|slack|mattermost|signal|imessage|msteams>`
-   `--local`
-   `--deliver`
-   `--json`
-   `--timeout `

### agents

Gestiona agentes aislados (espacios de trabajo + autenticación + enrutamiento).

#### agents list

Lista los agentes configurados. Opciones:

-   `--json`
-   `--bindings`

#### agents add \[nombre\]

Agrega un nuevo agente aislado. Ejecuta el asistente guiado a menos que se pasen banderas (o `--non-interactive`); `--workspace` es obligatorio en modo no interactivo. Opciones:

-   `--workspace `
-   `--model `
-   `--agent-dir `
-   `--bind <canal[:accountId]>` (repetible)
-   `--non-interactive`
-   `--json`

Las especificaciones de enlace usan `canal[:accountId]`. Cuando se omite `accountId`, OpenClaw puede resolver el alcance de la cuenta a través de valores predeterminados del canal/ganchos del plugin; de lo contrario, es un enlace de canal sin alcance de cuenta explícito.

#### agents bindings

Lista enlaces de enrutamiento. Opciones:

-   `--agent `
-   `--json`

#### agents bind

Agrega enlaces de enrutamiento para un agente. Opciones:

-   `--agent `
-   `--bind <canal[:accountId]>` (repetible)
-   `--json`

#### agents unbind

Elimina enlaces de enrutamiento para un agente. Opciones:

-   `--agent `
-   `--bind <canal[:accountId]>` (repetible)
-   `--all`
-   `--json`

#### agents delete &lt;id&gt;

Elimina un agente y poda su espacio de trabajo + estado. Opciones:

-   `--force`
-   `--json`

### acp

Ejecuta el puente ACP que conecta IDEs al Gateway. Ver [`acp`](./cli/acp.md) para opciones completas y ejemplos.

### status

Muestra la salud de la sesión vinculada y los destinatarios recientes. Opciones:

-   `--json`
-   `--all` (diagnóstico completo; solo lectura, se puede pegar)
-   `--deep` (sondea canales)
-   `--usage` (muestra uso/cuota del proveedor de modelos)
-   `--timeout `
-   `--verbose`
-   `--debug` (alias para `--verbose`)

Notas:

-   La descripción general incluye el estado del Gateway + servicio del host del nodo cuando está disponible.

### Seguimiento de uso

OpenClaw puede mostrar el uso/cuota del proveedor cuando hay credenciales OAuth/API disponibles. Muestra:

-   `/status` (agrega una línea corta de uso del proveedor cuando está disponible)
-   `openclaw status --usage` (imprime el desglose completo del proveedor)
-   Barra de menú de macOS (sección Uso bajo Contexto)

Notas:

-   Los datos provienen directamente de los endpoints de uso del proveedor (sin estimaciones).
-   Proveedores: Anthropic, GitHub Copilot, OpenAI Codex OAuth, más Gemini CLI/Antigravity cuando esos plugins de proveedor están habilitados.
-   Si no existen credenciales coincidentes, el uso está oculto.
-   Detalles: ver [Seguimiento de uso](./concepts/usage-tracking.md).

### health

Obtiene el estado de salud del Gateway en ejecución. Opciones:

-   `--json`
-   `--timeout `
-   `--verbose`

### sessions

Lista las sesiones de conversación almacenadas. Opciones:

-   `--json`
-   `--verbose`
-   `--store `
-   `--active `

## Restablecer / Desinstalar

### reset

Restablece la configuración/estado local (mantiene la CLI instalada). Opciones:

-   `--scope <config|config+creds+sessions|full>`
-   `--yes`
-   `--non-interactive`
-   `--dry-run`

Notas:

-   `--non-interactive` requiere `--scope` y `--yes`.

### uninstall

Desinstala el servicio del gateway + datos locales (la CLI permanece). Opciones:

-   `--service`
-   `--state`
-   `--workspace`
-   `--app`
-   `--all`
-   `--yes`
-   `--non-interactive`
-   `--dry-run`

Notas:

-   `--non-interactive` requiere `--yes` y alcances explícitos (o `--all`).

## Gateway

### gateway

Ejecuta el Gateway WebSocket. Opciones:

-   `--port `
-   `--bind <loopback|tailnet|lan|auto|custom>`
-   `--token `
-   `--auth <token|password>`
-   `--password <contraseña>`
-   `--tailscale <off|serve|funnel>`
-   `--tailscale-reset-on-exit`
-   `--allow-unconfigured`
-   `--dev`
-   `--reset` (restablece configuración de desarrollo + credenciales + sesiones + espacio de trabajo)
-   `--force` (mata el listener existente en el puerto)
-   `--verbose`
-   `--claude-cli-logs`
-   `--ws-log <auto|full|compact>`
-   `--compact` (alias para `--ws-log compact`)
-   `--raw-stream`
-   `--raw-stream-path `

### gateway service

Gestiona el servicio del Gateway (launchd/systemd/schtasks). Subcomandos:

-   `gateway status` (sondea el RPC del Gateway por defecto)
-   `gateway install` (instalación del servicio)
-   `gateway uninstall`
-   `gateway start`
-   `gateway stop`
-   `gateway restart`

Notas:

-   `gateway status` sondea el RPC del Gateway por defecto usando el puerto/configuración resuelto del servicio (anular con `--url/--token/--password`).
-   `gateway status` admite `--no-probe`, `--deep` y `--json` para scripting.
-   `gateway status` también muestra servicios de gateway heredados o adicionales cuando puede detectarlos (`--deep` agrega escaneos a nivel del sistema). Los servicios OpenClaw con nombre de perfil se tratan como de primera clase y no se marcan como "extra".
-   `gateway status` imprime qué ruta de configuración usa la CLI vs qué configuración probablemente usa el servicio (entorno del servicio), más la URL objetivo de sondeo resuelta.
-   `gateway install|uninstall|start|stop|restart` admite `--json` para scripting (la salida predeterminada sigue siendo amigable para humanos).
-   `gateway install` usa por defecto el entorno de ejecución Node; bun **no se recomienda** (errores de WhatsApp/Telegram).
-   Opciones de `gateway install`: `--port`, `--runtime`, `--token`, `--force`, `--json`.

### logs

Sigue los registros de archivo del Gateway a través de RPC. Notas:

-   Las sesiones TTY renderizan una vista estructurada y coloreada; las no TTY recurren a texto plano.
-   `--json` emite JSON delimitado por líneas (un evento de registro por línea).

Ejemplos:

```bash
openclaw logs --follow
openclaw logs --limit 200
openclaw logs --plain
openclaw logs --json
openclaw logs --no-color
```

### gateway &lt;subcomando&gt;

Asistentes CLI del Gateway (usa `--url`, `--token`, `--password`, `--timeout`, `--expect-final` para subcomandos RPC). Cuando pasas `--url`, la CLI no aplica automáticamente la configuración o credenciales del entorno. Incluye `--token` o `--password` explícitamente. La falta de credenciales explícitas es un error. Subcomandos:

-   `gateway call <método> [--params ]`
-   `gateway health`
-   `gateway status`
-   `gateway probe`
-   `gateway discover`
-   `gateway install|uninstall|start|stop|restart`
-   `gateway run`

RPCs comunes:

-   `config.apply` (valida + escribe configuración + reinicia + despierta)
-   `config.patch` (fusiona una