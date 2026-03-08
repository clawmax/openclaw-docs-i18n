

  Primeros pasos

  
# Asistente de incorporación (CLI)

El asistente de incorporación es la forma **recomendada** de configurar OpenClaw en macOS, Linux o Windows (vía WSL2; muy recomendado). Configura una Puerta de enlace local o una conexión a una Puerta de enlace remota, además de canales, habilidades y valores predeterminados del espacio de trabajo en un flujo guiado.

```bash
openclaw onboard
```

> **ℹ️** Primer chat más rápido: abre la Interfaz de Control (no se necesita configuración de canal). Ejecuta `openclaw dashboard` y chatea en el navegador. Docs: [Dashboard](../web/dashboard.md).

 Para reconfigurar más tarde:

```bash
openclaw configure
openclaw agents add <name>
```

> **ℹ️** `--json` no implica modo no interactivo. Para scripts, usa `--non-interactive`.

 

> **💡** El asistente de incorporación incluye un paso de búsqueda web donde puedes elegir un proveedor (Perplexity, Brave, Gemini, Grok o Kimi) y pegar tu clave API para que el agente pueda usar `web_search`. También puedes configurar esto más tarde con `openclaw configure --section web`. Docs: [Herramientas web](../tools/web.md).

## Inicio rápido vs Avanzado

El asistente comienza con **Inicio rápido** (valores predeterminados) vs **Avanzado** (control total). 

-   Puerta de enlace local (loopback)
-   Espacio de trabajo predeterminado (o espacio de trabajo existente)
-   Puerto de la puerta de enlace **18789**
-   Autenticación de la puerta de enlace **Token** (generado automáticamente, incluso en loopback)
-   Política de herramientas predeterminada para nuevas configuraciones locales: `tools.profile: "coding"` (se preserva cualquier perfil explícito existente)
-   Aislamiento de MD predeterminado: la incorporación local escribe `session.dmScope: "per-channel-peer"` cuando no está configurado. Detalles: [Referencia de incorporación CLI](./wizard-cli-reference.md#outputs-and-internals)
-   Exposición de Tailscale **Desactivada**
-   Los MD de Telegram y WhatsApp por defecto usan **lista de permitidos** (se te pedirá tu número de teléfono)

-   Expone cada paso (modo, espacio de trabajo, puerta de enlace, canales, daemon, habilidades).

## Qué configura el asistente

**Modo local (predeterminado)** te guía a través de estos pasos:

1.  **Modelo/Autenticación** — elige cualquier proveedor/flujo de autenticación compatible (clave API, OAuth o token de configuración), incluido Proveedor personalizado (compatible con OpenAI, compatible con Anthropic o detección automática Desconocido). Elige un modelo predeterminado. Nota de seguridad: si este agente ejecutará herramientas o procesará contenido de webhook/hooks, prefiere el modelo de última generación más potente disponible y mantén la política de herramientas estricta. Los niveles más débiles/antiguos son más fáciles de inyectar mediante prompts. Para ejecuciones no interactivas, `--secret-input-mode ref` almacena referencias respaldadas por variables de entorno en los perfiles de autenticación en lugar de valores de clave API en texto plano. En modo no interactivo `ref`, la variable de entorno del proveedor debe estar configurada; pasar flags de clave en línea sin esa variable de entorno falla rápidamente. En ejecuciones interactivas, elegir el modo de referencia secreta te permite apuntar a una variable de entorno o a una referencia de proveedor configurada (`file` o `exec`), con una validación previa rápida antes de guardar.
2.  **Espacio de trabajo** — Ubicación para los archivos del agente (predeterminado `~/.openclaw/workspace`). Siembra archivos de arranque.
3.  **Puerta de enlace** — Puerto, dirección de enlace, modo de autenticación, exposición de Tailscale. En modo interactivo de token, elige almacenamiento predeterminado de token en texto plano u opta por SecretRef. Ruta de SecretRef de token no interactivo: `--gateway-token-ref-env <ENV_VAR>`.
4.  **Canales** — WhatsApp, Telegram, Discord, Google Chat, Mattermost, Signal, BlueBubbles o iMessage.
5.  **Daemon** — Instala un LaunchAgent (macOS) o una unidad de usuario systemd (Linux/WSL2). Si la autenticación por token requiere un token y `gateway.auth.token` está gestionado por SecretRef, la instalación del daemon lo valida pero no persiste el token resuelto en los metadatos del entorno del servicio supervisor. Si la autenticación por token requiere un token y la SecretRef configurada no está resuelta, la instalación del daemon se bloquea con orientación accionable. Si tanto `gateway.auth.token` como `gateway.auth.password` están configurados y `gateway.auth.mode` no está configurado, la instalación del daemon se bloquea hasta que el modo se establece explícitamente.
6.  **Verificación de estado** — Inicia la Puerta de enlace y verifica que esté en ejecución.
7.  **Habilidades** — Instala habilidades recomendadas y dependencias opcionales.

> **ℹ️** Volver a ejecutar el asistente **no** borra nada a menos que elijas explícitamente **Restablecer** (o pases `--reset`). El CLI `--reset` por defecto afecta a configuración, credenciales y sesiones; usa `--reset-scope full` para incluir el espacio de trabajo. Si la configuración es inválida o contiene claves heredadas, el asistente te pedirá que ejecutes `openclaw doctor` primero.

 **Modo remoto** solo configura el cliente local para conectarse a una Puerta de enlace en otro lugar. **No** instala ni cambia nada en el host remoto.

## Añadir otro agente

Usa `openclaw agents add ` para crear un agente separado con su propio espacio de trabajo, sesiones y perfiles de autenticación. Ejecutarlo sin `--workspace` inicia el asistente. Lo que configura:

-   `agents.list[].name`
-   `agents.list[].workspace`
-   `agents.list[].agentDir`

Notas:

-   Los espacios de trabajo predeterminados siguen `~/.openclaw/workspace-`.
-   Añade `bindings` para enrutar mensajes entrantes (el asistente puede hacer esto).
-   Flags no interactivos: `--model`, `--agent-dir`, `--bind`, `--non-interactive`.

## Referencia completa

Para desgloses detallados paso a paso, scripting no interactivo, configuración de Signal, API RPC y una lista completa de campos de configuración que escribe el asistente, consulta la [Referencia del asistente](../reference/wizard.md).

## Documentación relacionada

-   Referencia de comandos CLI: [`openclaw onboard`](../cli/onboard.md)
-   Descripción general de incorporación: [Descripción general de incorporación](./onboarding-overview.md)
-   Incorporación de la aplicación macOS: [Incorporación](./onboarding.md)
-   Ritual de primera ejecución del agente: [Arranque del agente](./bootstrapping.md)

[Descripción general de incorporación](./onboarding-overview.md)[Incorporación: Aplicación macOS](./onboarding.md)

---