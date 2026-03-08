

  Guías

  
# Referencia de incorporación CLI

Esta página es la referencia completa para `openclaw onboard`. Para la guía breve, consulta [Asistente de incorporación (CLI)](./wizard.md).

## Qué hace el asistente

El modo local (predeterminado) te guía a través de:

-   Configuración de modelo y autenticación (OAuth de suscripción OpenAI Code, clave API de Anthropic o token de configuración, más opciones de MiniMax, GLM, Moonshot y AI Gateway)
-   Ubicación del espacio de trabajo y archivos de arranque
-   Configuración de la puerta de enlace (puerto, enlace, autenticación, tailscale)
-   Canales y proveedores (Telegram, WhatsApp, Discord, Google Chat, complemento Mattermost, Signal)
-   Instalación del daemon (LaunchAgent o unidad de usuario systemd)
-   Comprobación de estado
-   Configuración de habilidades

El modo remoto configura esta máquina para conectarse a una puerta de enlace en otro lugar. No instala ni modifica nada en el host remoto.

## Detalles del flujo local

### Paso 1: Detección de configuración existente

-   Si existe `~/.openclaw/openclaw.json`, elige Conservar, Modificar o Restablecer.
-   Volver a ejecutar el asistente no borra nada a menos que elijas explícitamente Restablecer (o pases `--reset`).
-   La CLI `--reset` tiene como valor predeterminado `config+creds+sessions`; usa `--reset-scope full` para también eliminar el espacio de trabajo.
-   Si la configuración no es válida o contiene claves heredadas, el asistente se detiene y te pide que ejecutes `openclaw doctor` antes de continuar.
-   Restablecer usa `trash` y ofrece alcances:
    -   Solo configuración
    -   Configuración + credenciales + sesiones
    -   Restablecimiento completo (también elimina el espacio de trabajo)

### Paso 2: Modelo y autenticación

-   La matriz completa de opciones está en [Opciones de autenticación y modelo](#auth-and-model-options).

### Paso 3: Espacio de trabajo

-   Predeterminado `~/.openclaw/workspace` (configurable).
-   Siembra los archivos del espacio de trabajo necesarios para el ritual de arranque de la primera ejecución.
-   Diseño del espacio de trabajo: [Espacio de trabajo del agente](../concepts/agent-workspace.md).

### Paso 4: Puerta de enlace

-   Solicita puerto, enlace, modo de autenticación y exposición de tailscale.
-   Recomendado: mantener la autenticación por token habilitada incluso para loopback, para que los clientes WS locales deban autenticarse.
-   En modo token, la incorporación interactiva ofrece:
    -   **Generar/almacenar token en texto plano** (predeterminado)
    -   **Usar SecretRef** (opt-in)
-   En modo contraseña, la incorporación interactiva también admite almacenamiento en texto plano o SecretRef.
-   Ruta de SecretRef de token no interactivo: `--gateway-token-ref-env <ENV_VAR>`.
    -   Requiere una variable de entorno no vacía en el entorno del proceso de incorporación.
    -   No se puede combinar con `--gateway-token`.
-   Deshabilita la autenticación solo si confías plenamente en cada proceso local.
-   Los enlaces que no son loopback aún requieren autenticación.

### Paso 5: Canales

-   [WhatsApp](../channels/whatsapp.md): inicio de sesión QR opcional
-   [Telegram](../channels/telegram.md): token del bot
-   [Discord](../channels/discord.md): token del bot
-   [Google Chat](../channels/googlechat.md): JSON de cuenta de servicio + audiencia del webhook
-   Complemento [Mattermost](../channels/mattermost.md): token del bot + URL base
-   [Signal](../channels/signal.md): instalación opcional de `signal-cli` + configuración de cuenta
-   [BlueBubbles](../channels/bluebubbles.md): recomendado para iMessage; URL del servidor + contraseña + webhook
-   [iMessage](../channels/imessage.md): ruta CLI heredada `imsg` + acceso a la base de datos
-   Seguridad de DM: el valor predeterminado es emparejamiento. El primer DM envía un código; aprueba mediante `openclaw pairing approve  ` o usa listas de permitidos.

### Paso 6: Instalación del daemon

-   macOS: LaunchAgent
    -   Requiere sesión de usuario iniciada; para headless, usa un LaunchDaemon personalizado (no incluido).
-   Linux y Windows a través de WSL2: unidad de usuario systemd
    -   El asistente intenta `loginctl enable-linger ` para que la puerta de enlace permanezca activa después del cierre de sesión.
    -   Puede solicitar sudo (escribe `/var/lib/systemd/linger`); primero intenta sin sudo.
-   Selección del entorno de ejecución: Node (recomendado; requerido para WhatsApp y Telegram). Bun no es recomendado.

### Paso 7: Comprobación de estado

-   Inicia la puerta de enlace (si es necesario) y ejecuta `openclaw health`.
-   `openclaw status --deep` agrega sondeos de estado de la puerta de enlace a la salida de estado.

### Paso 8: Habilidades

-   Lee las habilidades disponibles y verifica los requisitos.
-   Te permite elegir el administrador de nodos: npm o pnpm (bun no recomendado).
-   Instala dependencias opcionales (algunas usan Homebrew en macOS).

### Paso 9: Finalizar

-   Resumen y próximos pasos, incluidas opciones de aplicaciones iOS, Android y macOS.

 

> **ℹ️** Si no se detecta una GUI, el asistente imprime instrucciones de reenvío de puertos SSH para la UI de Control en lugar de abrir un navegador. Si faltan los recursos de la UI de Control, el asistente intenta construirlos; el respaldo es `pnpm ui:build` (instala automáticamente las dependencias de la UI).

## Detalles del modo remoto

El modo remoto configura esta máquina para conectarse a una puerta de enlace en otro lugar.

> **ℹ️** El modo remoto no instala ni modifica nada en el host remoto.

 Lo que configuras:

-   URL de la puerta de enlace remota (`ws://...`)
-   Token si la autenticación de la puerta de enlace remota es requerida (recomendado)

> **ℹ️** -   Si la puerta de enlace es solo loopback, usa túnel SSH o una tailnet.
> -   Pistas de descubrimiento:
>     -   macOS: Bonjour (`dns-sd`)
>     -   Linux: Avahi (`avahi-browse`)

## Opciones de autenticación y modelo

Usa `ANTHROPIC_API_KEY` si está presente o solicita una clave, luego la guarda para uso del daemon.

-   macOS: verifica el elemento del llavero “Claude Code-credentials”
-   Linux y Windows: reutiliza `~/.claude/.credentials.json` si está presente

En macOS, elige “Permitir siempre” para que los inicios de launchd no se bloqueen.

Ejecuta `claude setup-token` en cualquier máquina, luego pega el token. Puedes nombrarlo; en blanco usa el predeterminado.

Si existe `~/.codex/auth.json`, el asistente puede reutilizarlo.

Flujo del navegador; pega `code#state`.Establece `agents.defaults.model` en `openai-codex/gpt-5.4` cuando el modelo no está configurado o es `openai/*`.

Usa `OPENAI_API_KEY` si está presente o solicita una clave, luego almacena la credencial en perfiles de autenticación.Establece `agents.defaults.model` en `openai/gpt-5.1-codex` cuando el modelo no está configurado, es `openai/*` o `openai-codex/*`.

Solicita `XAI_API_KEY` y configura xAI como proveedor de modelos.

Solicita `OPENCODE_API_KEY` (o `OPENCODE_ZEN_API_KEY`). URL de configuración: [opencode.ai/auth](https://opencode.ai/auth).

Almacena la clave por ti.

Solicita `AI_GATEWAY_API_KEY`. Más detalles: [Vercel AI Gateway](../providers/vercel-ai-gateway.md).

Solicita ID de cuenta, ID de puerta de enlace y `CLOUDFLARE_AI_GATEWAY_API_KEY`. Más detalles: [Cloudflare AI Gateway](../providers/cloudflare-ai-gateway.md).

La configuración se escribe automáticamente. Más detalles: [MiniMax](../providers/minimax.md).

Solicita `SYNTHETIC_API_KEY`. Más detalles: [Synthetic](../providers/synthetic.md).

Las configuraciones de Moonshot (Kimi K2) y Kimi Coding se escriben automáticamente. Más detalles: [Moonshot AI (Kimi + Kimi Coding)](../providers/moonshot.md).

Funciona con endpoints compatibles con OpenAI y compatibles con Anthropic.La incorporación interactiva admite las mismas opciones de almacenamiento de claves API que otros flujos de claves API de proveedores:

-   **Pegar clave API ahora** (texto plano)
-   **Usar referencia secreta** (referencia de entorno o referencia de proveedor configurado, con validación previa)

Banderas no interactivas:

-   `--auth-choice custom-api-key`
-   `--custom-base-url`
-   `--custom-model-id`
-   `--custom-api-key` (opcional; recurre a `CUSTOM_API_KEY`)
-   `--custom-provider-id` (opcional)
-   `--custom-compatibility <openai|anthropic>` (opcional; predeterminado `openai`)

Deja la autenticación sin configurar.

 Comportamiento del modelo:

-   Elige el modelo predeterminado de las opciones detectadas, o ingresa el proveedor y el modelo manualmente.
-   El asistente ejecuta una verificación del modelo y advierte si el modelo configurado es desconocido o le falta autenticación.

Rutas de credenciales y perfiles:

-   Credenciales OAuth: `~/.openclaw/credentials/oauth.json`
-   Perfiles de autenticación (claves API + OAuth): `~/.openclaw/agents//agent/auth-profiles.json`

Modo de almacenamiento de credenciales:

-   El comportamiento predeterminado de incorporación persiste las claves API como valores de texto plano en los perfiles de autenticación.
-   `--secret-input-mode ref` habilita el modo de referencia en lugar del almacenamiento de claves en texto plano. En la incorporación interactiva, puedes elegir cualquiera:
    -   referencia de variable de entorno (por ejemplo `keyRef: { source: "env", provider: "default", id: "OPENAI_API_KEY" }`)
    -   referencia de proveedor configurado (`file` o `exec`) con alias de proveedor + id
-   El modo de referencia interactivo ejecuta una validación previa rápida antes de guardar.
    -   Referencias de entorno: valida el nombre de la variable + valor no vacío en el entorno actual de incorporación.
    -   Referencias de proveedor: valida la configuración del proveedor y resuelve el id solicitado.
    -   Si la validación previa falla, la incorporación muestra el error y te permite reintentar.
-   En modo no interactivo, `--secret-input-mode ref` solo es respaldado por entorno.
    -   Establece la variable de entorno del proveedor en el entorno del proceso de incorporación.
    -   Las banderas de clave en línea (por ejemplo `--openai-api-key`) requieren que esa variable de entorno esté configurada; de lo contrario, la incorporación falla rápidamente.
    -   Para proveedores personalizados, el modo `ref` no interactivo almacena `models.providers..apiKey` como `{ source: "env", provider: "default", id: "CUSTOM_API_KEY" }`.
    -   En ese caso de proveedor personalizado, `--custom-api-key` requiere que `CUSTOM_API_KEY` esté configurada; de lo contrario, la incorporación falla rápidamente.
-   Las credenciales de autenticación de la puerta de enlace admiten opciones de texto plano y SecretRef en la incorporación interactiva:
    -   Modo token: **Generar/almacenar token en texto plano** (predeterminado) o **Usar SecretRef**.
    -   Modo contraseña: texto plano o SecretRef.
-   Ruta de SecretRef de token no interactivo: `--gateway-token-ref-env <ENV_VAR>`.
-   Las configuraciones existentes en texto plano continúan funcionando sin cambios.

> **ℹ️** Consejo para servidores y headless: completa OAuth en una máquina con navegador, luego copia `~/.openclaw/credentials/oauth.json` (o `$OPENCLAW_STATE_DIR/credentials/oauth.json`) al host de la puerta de enlace.

## Salidas e internos

Campos típicos en `~/.openclaw/openclaw.json`:

-   `agents.defaults.workspace`
-   `agents.defaults.model` / `models.providers` (si se elige Minimax)
-   `tools.profile` (la incorporación local establece `"coding"` como predeterminado cuando no está configurado; los valores explícitos existentes se conservan)
-   `gateway.*` (modo, enlace, autenticación, tailscale)
-   `session.dmScope` (la incorporación local establece esto como `per-channel-peer` cuando no está configurado; los valores explícitos existentes se conservan)
-   `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
-   Listas de permitidos de canales (Slack, Discord, Matrix, Microsoft Teams) cuando optas por ellas durante las solicitudes (los nombres se resuelven a IDs cuando es posible)
-   `skills.install.nodeManager`
-   `wizard.lastRunAt`
-   `wizard.lastRunVersion`
-   `wizard.lastRunCommit`
-   `wizard.lastRunCommand`
-   `wizard.lastRunMode`

`openclaw agents add` escribe `agents.list[]` y `bindings` opcional. Las credenciales de WhatsApp van bajo `~/.openclaw/credentials/whatsapp//`. Las sesiones se almacenan bajo `~/.openclaw/agents//sessions/`.

> **ℹ️** Algunos canales se entregan como complementos. Cuando se seleccionan durante la incorporación, el asistente solicita instalar el complemento (npm o ruta local) antes de la configuración del canal.

 RPC del asistente de puerta de enlace:

-   `wizard.start`
-   `wizard.next`
-   `wizard.cancel`
-   `wizard.status`

Los clientes (aplicación macOS y UI de Control) pueden renderizar pasos sin reimplementar la lógica de incorporación. Comportamiento de configuración de Signal:

-   Descarga el recurso de lanzamiento apropiado
-   Lo almacena bajo `~/.openclaw/tools/signal-cli//`
-   Escribe `channels.signal.cliPath` en la configuración
-   Las compilaciones JVM requieren Java 21
-   Las compilaciones nativas se usan cuando están disponibles
-   Windows usa WSL2 y sigue el flujo signal-cli de Linux dentro de WSL

## Documentación relacionada

-   Centro de incorporación: [Asistente de incorporación (CLI)](./wizard.md)
-   Automatización y scripts: [Automatización CLI](./wizard-cli-automation.md)
-   Referencia de comandos: [`openclaw onboard`](../cli/onboard.md)

[Configuración de Asistente Personal](./openclaw.md)[Automatización CLI](./wizard-cli-automation.md)