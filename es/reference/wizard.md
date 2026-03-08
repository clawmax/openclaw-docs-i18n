

  Referencia técnica

  
# Referencia del Asistente de Incorporación

Esta es la referencia completa para el asistente CLI `openclaw onboard`. Para una visión general de alto nivel, consulta [Asistente de Incorporación](../start/wizard.md).

## Detalles del flujo (modo local)

### Paso 1: Detección de configuración existente

-   Si existe `~/.openclaw/openclaw.json`, elige **Conservar / Modificar / Restablecer**.
-   Volver a ejecutar el asistente **no** borra nada a menos que elijas explícitamente **Restablecer** (o pases `--reset`).
-   La CLI `--reset` por defecto es `config+creds+sessions`; usa `--reset-scope full` para también eliminar el espacio de trabajo.
-   Si la configuración es inválida o contiene claves heredadas, el asistente se detiene y te pide que ejecutes `openclaw doctor` antes de continuar.
-   Restablecer usa `trash` (nunca `rm`) y ofrece alcances:
    -   Solo configuración
    -   Configuración + credenciales + sesiones
    -   Restablecimiento completo (también elimina el espacio de trabajo)

### Paso 2: Modelo/Autenticación

-   **Clave API de Anthropic**: usa `ANTHROPIC_API_KEY` si está presente o solicita una clave, luego la guarda para uso del demonio.
-   **OAuth de Anthropic (Claude Code CLI)**: en macOS el asistente verifica el elemento del Llavero “Claude Code-credentials” (elige “Permitir siempre” para que los inicios de launchd no se bloqueen); en Linux/Windows reutiliza `~/.claude/.credentials.json` si está presente.
-   **Token de Anthropic (pegar setup-token)**: ejecuta `claude setup-token` en cualquier máquina, luego pega el token (puedes nombrarlo; en blanco = predeterminado).
-   **Suscripción OpenAI Code (Codex) (Codex CLI)**: si existe `~/.codex/auth.json`, el asistente puede reutilizarlo.
-   **Suscripción OpenAI Code (Codex) (OAuth)**: flujo del navegador; pega el `code#state`.
    -   Establece `agents.defaults.model` a `openai-codex/gpt-5.2` cuando el modelo no está configurado o es `openai/*`.
-   **Clave API de OpenAI**: usa `OPENAI_API_KEY` si está presente o solicita una clave, luego la almacena en perfiles de autenticación.
-   **Clave API de xAI (Grok)**: solicita `XAI_API_KEY` y configura xAI como proveedor de modelos.
-   **OpenCode Zen (proxy multi-modelo)**: solicita `OPENCODE_API_KEY` (o `OPENCODE_ZEN_API_KEY`, consíguela en [https://opencode.ai/auth](https://opencode.ai/auth)).
-   **Clave API**: almacena la clave por ti.
-   **Vercel AI Gateway (proxy multi-modelo)**: solicita `AI_GATEWAY_API_KEY`.
-   Más detalles: [Vercel AI Gateway](../providers/vercel-ai-gateway.md)
-   **Cloudflare AI Gateway**: solicita ID de Cuenta, ID de Puerta de Enlace y `CLOUDFLARE_AI_GATEWAY_API_KEY`.
-   Más detalles: [Cloudflare AI Gateway](../providers/cloudflare-ai-gateway.md)
-   **MiniMax M2.5**: la configuración se escribe automáticamente.
-   Más detalles: [MiniMax](../providers/minimax.md)
-   **Synthetic (compatible con Anthropic)**: solicita `SYNTHETIC_API_KEY`.
-   Más detalles: [Synthetic](../providers/synthetic.md)
-   **Moonshot (Kimi K2)**: la configuración se escribe automáticamente.
-   **Kimi Coding**: la configuración se escribe automáticamente.
-   Más detalles: [Moonshot AI (Kimi + Kimi Coding)](../providers/moonshot.md)
-   **Omitir**: no se configura autenticación aún.
-   Elige un modelo predeterminado de las opciones detectadas (o ingresa proveedor/modelo manualmente). Para la mejor calidad y menor riesgo de inyección de prompts, elige el modelo más potente de última generación disponible en tu pila de proveedores.
-   El asistente ejecuta una verificación del modelo y advierte si el modelo configurado es desconocido o le falta autenticación.
-   El modo de almacenamiento de claves API por defecto es valores de perfil de autenticación en texto plano. Usa `--secret-input-mode ref` para almacenar referencias respaldadas por variables de entorno en su lugar (por ejemplo `keyRef: { source: "env", provider: "default", id: "OPENAI_API_KEY" }`).
-   Las credenciales OAuth residen en `~/.openclaw/credentials/oauth.json`; los perfiles de autenticación residen en `~/.openclaw/agents//agent/auth-profiles.json` (claves API + OAuth).
-   Más detalles: [/concepts/oauth](../concepts/oauth.md)

> **ℹ️** Consejo para modo headless/servidor: completa OAuth en una máquina con navegador, luego copia `~/.openclaw/credentials/oauth.json` (o `$OPENCLAW_STATE_DIR/credentials/oauth.json`) al host de la puerta de enlace.

### Paso 3: Espacio de trabajo

-   Predeterminado `~/.openclaw/workspace` (configurable).
-   Inicializa los archivos del espacio de trabajo necesarios para el ritual de arranque del agente.
-   Diseño completo del espacio de trabajo + guía de respaldo: [Espacio de trabajo del agente](../concepts/agent-workspace.md)

### Paso 4: Puerta de enlace

-   Puerto, enlace, modo de autenticación, exposición de tailscale.
-   Recomendación de autenticación: mantén **Token** incluso para loopback para que los clientes WS locales deban autenticarse.
-   En modo token, la incorporación interactiva ofrece:
    -   **Generar/almacenar token en texto plano** (predeterminado)
    -   **Usar SecretRef** (opt-in)
    -   El inicio rápido reutiliza las SecretRefs existentes de `gateway.auth.token` entre los proveedores `env`, `file` y `exec` para la sonda de incorporación/arranque del panel de control.
    -   Si esa SecretRef está configurada pero no se puede resolver, la incorporación falla temprano con un mensaje de solución claro en lugar de degradar silenciosamente la autenticación en tiempo de ejecución.
-   En modo contraseña, la incorporación interactiva también admite almacenamiento en texto plano o SecretRef.
-   Ruta de SecretRef de token no interactiva: `--gateway-token-ref-env <ENV_VAR>`.
    -   Requiere una variable de entorno no vacía en el entorno del proceso de incorporación.
    -   No se puede combinar con `--gateway-token`.
-   Deshabilita la autenticación solo si confías plenamente en cada proceso local.
-   Los enlaces que no son loopback aún requieren autenticación.

### Paso 5: Canales

-   [WhatsApp](../channels/whatsapp.md): inicio de sesión QR opcional.
-   [Telegram](../channels/telegram.md): token del bot.
-   [Discord](../channels/discord.md): token del bot.
-   [Google Chat](../channels/googlechat.md): JSON de cuenta de servicio + audiencia del webhook.
-   [Mattermost](../channels/mattermost.md) (plugin): token del bot + URL base.
-   [Signal](../channels/signal.md): instalación opcional de `signal-cli` + configuración de cuenta.
-   [BlueBubbles](../channels/bluebubbles.md): **recomendado para iMessage**; URL del servidor + contraseña + webhook.
-   [iMessage](../channels/imessage.md): ruta CLI heredada `imsg` + acceso a la base de datos.
-   Seguridad de DM: el valor predeterminado es emparejamiento. El primer DM envía un código; aprueba mediante `openclaw pairing approve  ` o usa listas de permitidos.

### Paso 6: Búsqueda web

-   Elige un proveedor: Perplexity, Brave, Gemini, Grok o Kimi (u omite).
-   Pega tu clave API (Inicio Rápido detecta automáticamente claves de variables de entorno o configuración existente).
-   Omite con `--skip-search`.
-   Configura más tarde: `openclaw configure --section web`.

### Paso 7: Instalación del demonio

-   macOS: LaunchAgent
    -   Requiere una sesión de usuario iniciada; para headless, usa un LaunchDaemon personalizado (no incluido).
-   Linux (y Windows vía WSL2): unidad de usuario de systemd
    -   El asistente intenta habilitar lingering mediante `loginctl enable-linger ` para que la Puerta de Enlace permanezca activa después del cierre de sesión.
    -   Puede solicitar sudo (escribe `/var/lib/systemd/linger`); primero intenta sin sudo.
-   **Selección del entorno de ejecución:** Node (recomendado; requerido para WhatsApp/Telegram). Bun **no es recomendado**.
-   Si la autenticación por token requiere un token y `gateway.auth.token` está gestionado por SecretRef, la instalación del demonio lo valida pero no persiste los valores de token en texto plano resueltos en los metadatos del entorno del servicio supervisor.
-   Si la autenticación por token requiere un token y la SecretRef de token configurada no está resuelta, la instalación del demonio se bloquea con orientación accionable.
-   Si tanto `gateway.auth.token` como `gateway.auth.password` están configurados y `gateway.auth.mode` no está establecido, la instalación del demonio se bloquea hasta que el modo se establezca explícitamente.

### Paso 8: Verificación de estado

-   Inicia la Puerta de Enlace (si es necesario) y ejecuta `openclaw health`.
-   Consejo: `openclaw status --deep` agrega sondas de estado de la puerta de enlace a la salida de estado (requiere una puerta de enlace alcanzable).

### Paso 9: Habilidades (recomendado)

-   Lee las habilidades disponibles y verifica los requisitos.
-   Te permite elegir un gestor de paquetes de node: **npm / pnpm** (bun no recomendado).
-   Instala dependencias opcionales (algunas usan Homebrew en macOS).

### Paso 10: Finalizar

-   Resumen + próximos pasos, incluyendo aplicaciones iOS/Android/macOS para funciones adicionales.

 

> **ℹ️** Si no se detecta una GUI, el asistente imprime instrucciones de reenvío de puertos SSH para la UI de Control en lugar de abrir un navegador. Si faltan los recursos de la UI de Control, el asistente intenta construirlos; el respaldo es `pnpm ui:build` (instala automáticamente las dependencias de la UI).

## Modo no interactivo

Usa `--non-interactive` para automatizar o escribir scripts de incorporación:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

Agrega `--json` para un resumen legible por máquina. SecretRef de token de puerta de enlace en modo no interactivo:

```bash
export OPENCLAW_GATEWAY_TOKEN="your-token"
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice skip \
  --gateway-auth token \
  --gateway-token-ref-env OPENCLAW_GATEWAY_TOKEN
```

`--gateway-token` y `--gateway-token-ref-env` son mutuamente excluyentes.

> **ℹ️** `--json` **no** implica modo no interactivo. Usa `--non-interactive` (y `--workspace`) para scripts.

 

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice gemini-api-key \
  --gemini-api-key "$GEMINI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice zai-api-key \
  --zai-api-key "$ZAI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice cloudflare-ai-gateway-api-key \
  --cloudflare-ai-gateway-account-id "your-account-id" \
  --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
  --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice moonshot-api-key \
  --moonshot-api-key "$MOONSHOT_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice synthetic-api-key \
  --synthetic-api-key "$SYNTHETIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice opencode-zen \
  --opencode-zen-api-key "$OPENCODE_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

### Agregar agente (no interactivo)

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

## RPC del asistente de la Puerta de Enlace

La Puerta de Enlace expone el flujo del asistente a través de RPC (`wizard.start`, `wizard.next`, `wizard.cancel`, `wizard.status`). Los clientes (aplicación macOS, UI de Control) pueden renderizar pasos sin reimplementar la lógica de incorporación.

## Configuración de Signal (signal-cli)

El asistente puede instalar `signal-cli` desde las versiones de GitHub:

-   Descarga el recurso de versión apropiado.
-   Lo almacena bajo `~/.openclaw/tools/signal-cli//`.
-   Escribe `channels.signal.cliPath` en tu configuración.

Notas:

-   Las compilaciones JVM requieren **Java 21**.
-   Se usan compilaciones nativas cuando están disponibles.
-   Windows usa WSL2; la instalación de signal-cli sigue el flujo de Linux dentro de WSL.

## Lo que escribe el asistente

Campos típicos en `~/.openclaw/openclaw.json`:

-   `agents.defaults.workspace`
-   `agents.defaults.model` / `models.providers` (si se elige Minimax)
-   `tools.profile` (la incorporación local por defecto es `"coding"` cuando no está configurado; los valores explícitos existentes se conservan)
-   `gateway.*` (modo, enlace, autenticación, tailscale)
-   `session.dmScope` (detalles de comportamiento: [Referencia de Incorporación CLI](../start/wizard-cli-reference.md#outputs-and-internals))
-   `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
-   Listas de permitidos de canales (Slack/Discord/Matrix/Microsoft Teams) cuando optas por participar durante las solicitudes (los nombres se resuelven a IDs cuando es posible).
-   `skills.install.nodeManager`
-   `wizard.lastRunAt`
-   `wizard.lastRunVersion`
-   `wizard.lastRunCommit`
-   `wizard.lastRunCommand`
-   `wizard.lastRunMode`

`openclaw agents add` escribe `agents.list[]` y `bindings` opcionales. Las credenciales de WhatsApp van bajo `~/.openclaw/credentials/whatsapp//`. Las sesiones se almacenan bajo `~/.openclaw/agents//sessions/`. Algunos canales se entregan como plugins. Cuando eliges uno durante la incorporación, el asistente solicitará instalarlo (npm o una ruta local) antes de que pueda configurarse.

## Documentación relacionada

-   Visión general del asistente: [Asistente de Incorporación](../start/wizard.md)
-   Incorporación de la aplicación macOS: [Incorporación](../start/onboarding.md)
-   Referencia de configuración: [Configuración de la Puerta de Enlace](../gateway/configuration.md)
-   Proveedores: [WhatsApp](../channels/whatsapp.md), [Telegram](../channels/telegram.md), [Discord](../channels/discord.md), [Google Chat](../channels/googlechat.md), [Signal](../channels/signal.md), [BlueBubbles](../channels/bluebubbles.md) (iMessage), [iMessage](../channels/imessage.md) (heredado)
-   Habilidades: [Habilidades](../tools/skills.md), [Configuración de habilidades](../tools/skills-config.md)

[USER](./templates/USER.md)[Uso y Costos de Tokens](./token-use.md)