

  Guías

  
# Configuración del Asistente Personal

OpenClaw es una puerta de enlace para agentes **Pi** en WhatsApp + Telegram + Discord + iMessage. Los complementos añaden Mattermost. Esta guía es la configuración de "asistente personal": un número de WhatsApp dedicado que se comporta como tu agente siempre activo.

## ⚠️ Seguridad primero

Estás poniendo a un agente en una posición para:

-   ejecutar comandos en tu máquina (dependiendo de tu configuración de herramientas Pi)
-   leer/escribir archivos en tu espacio de trabajo
-   enviar mensajes de vuelta a través de WhatsApp/Telegram/Discord/Mattermost (complemento)

Comienza de forma conservadora:

-   Siempre configura `channels.whatsapp.allowFrom` (nunca ejecutes abierto al mundo en tu Mac personal).
-   Usa un número de WhatsApp dedicado para el asistente.
-   Los latidos ahora están configurados por defecto cada 30 minutos. Desactívalos hasta que confíes en la configuración estableciendo `agents.defaults.heartbeat.every: "0m"`.

## Prerrequisitos

-   OpenClaw instalado y configurado — consulta [Primeros pasos](./getting-started.md) si aún no lo has hecho
-   Un segundo número de teléfono (SIM/eSIM/prepago) para el asistente

## La configuración de dos teléfonos (recomendada)

Esto es lo que quieres: Si vinculas tu WhatsApp personal a OpenClaw, cada mensaje para ti se convierte en "entrada del agente". Rara vez es eso lo que quieres.

## Inicio rápido de 5 minutos

1.  Empareja WhatsApp Web (muestra un código QR; escanéalo con el teléfono del asistente):

```bash
openclaw channels login
```

2.  Inicia la Puerta de enlace (déjala ejecutándose):

```bash
openclaw gateway --port 18789
```

3.  Coloca una configuración mínima en `~/.openclaw/openclaw.json`:

```json
{
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

Ahora envía un mensaje al número del asistente desde tu teléfono permitido. Cuando finalice la configuración inicial, abrimos automáticamente el panel de control e imprimimos un enlace limpio (sin token). Si solicita autenticación, pega el token de `gateway.auth.token` en la configuración de la Interfaz de Control. Para reabrirlo más tarde: `openclaw dashboard`.

## Dale al agente un espacio de trabajo (AGENTS)

OpenClaw lee las instrucciones de operación y la "memoria" desde su directorio de espacio de trabajo. Por defecto, OpenClaw usa `~/.openclaw/workspace` como el espacio de trabajo del agente, y lo creará (junto con los archivos iniciales `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`) automáticamente durante la configuración/primer ejecución del agente. `BOOTSTRAP.md` solo se crea cuando el espacio de trabajo es completamente nuevo (no debería reaparecer después de eliminarlo). `MEMORY.md` es opcional (no se crea automáticamente); cuando está presente, se carga para sesiones normales. Las sesiones de subagentes solo inyectan `AGENTS.md` y `TOOLS.md`. Consejo: trata esta carpeta como la "memoria" de OpenClaw y hazla un repositorio git (idealmente privado) para que tus archivos `AGENTS.md` + de memoria estén respaldados. Si git está instalado, los espacios de trabajo nuevos se inicializan automáticamente.

```bash
openclaw setup
```

Diseño completo del espacio de trabajo + guía de respaldo: [Espacio de trabajo del agente](../concepts/agent-workspace.md) Flujo de trabajo de memoria: [Memoria](../concepts/memory.md) Opcional: elige un espacio de trabajo diferente con `agents.defaults.workspace` (soporta `~`).

```json
{
  agent: {
    workspace: "~/.openclaw/workspace",
  },
}
```

Si ya envías tus propios archivos de espacio de trabajo desde un repositorio, puedes desactivar completamente la creación de archivos de arranque:

```json
{
  agent: {
    skipBootstrap: true,
  },
}
```

## La configuración que lo convierte en "un asistente"

OpenClaw tiene por defecto una buena configuración de asistente, pero normalmente querrás ajustar:

-   persona/instrucciones en `SOUL.md`
-   valores predeterminados de pensamiento (si se desea)
-   latidos (una vez que confíes en él)

Ejemplo:

```json
{
  logging: { level: "info" },
  agent: {
    model: "anthropic/claude-opus-4-6",
    workspace: "~/.openclaw/workspace",
    thinkingDefault: "high",
    timeoutSeconds: 1800,
    // Comienza con 0; habilita más tarde.
    heartbeat: { every: "0m" },
  },
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true },
      },
    },
  },
  routing: {
    groupChat: {
      mentionPatterns: ["@openclaw", "openclaw"],
    },
  },
  session: {
    scope: "per-sender",
    resetTriggers: ["/new", "/reset"],
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 10080,
    },
  },
}
```

## Sesiones y memoria

-   Archivos de sesión: `~/.openclaw/agents//sessions/{{SessionId}}.jsonl`
-   Metadatos de sesión (uso de tokens, última ruta, etc.): `~/.openclaw/agents//sessions/sessions.json` (heredado: `~/.openclaw/sessions/sessions.json`)
-   `/new` o `/reset` inicia una sesión nueva para ese chat (configurable mediante `resetTriggers`). Si se envía solo, el agente responde con un breve saludo para confirmar el reinicio.
-   `/compact [instrucciones]` compacta el contexto de la sesión e informa del presupuesto de contexto restante.

## Latidos (modo proactivo)

Por defecto, OpenClaw ejecuta un latido cada 30 minutos con el mensaje: `Lee HEARTBEAT.md si existe (contexto del espacio de trabajo). Síguelo estrictamente. No infieras ni repitas tareas antiguas de chats anteriores. Si no hay nada que requiera atención, responde HEARTBEAT_OK.` Establece `agents.defaults.heartbeat.every: "0m"` para desactivar.

-   Si `HEARTBEAT.md` existe pero está efectivamente vacío (solo líneas en blanco y encabezados de markdown como `# Encabezado`), OpenClaw omite la ejecución del latido para ahorrar llamadas a la API.
-   Si el archivo falta, el latido aún se ejecuta y el modelo decide qué hacer.
-   Si el agente responde con `HEARTBEAT_OK` (opcionalmente con un breve relleno; ver `agents.defaults.heartbeat.ackMaxChars`), OpenClaw suprime la entrega saliente para ese latido.
-   Por defecto, se permite la entrega de latidos a objetivos de estilo DM `user:`. Establece `agents.defaults.heartbeat.directPolicy: "block"` para suprimir la entrega a objetivos directos mientras mantienes activas las ejecuciones de latidos.
-   Los latidos ejecutan turnos completos del agente — intervalos más cortos consumen más tokens.

```json
{
  agent: {
    heartbeat: { every: "30m" },
  },
}
```

## Medios de entrada y salida

Los archivos adjuntos entrantes (imágenes/audio/documentos) pueden mostrarse a tu comando mediante plantillas:

-   `{{MediaPath}}` (ruta de archivo temporal local)
-   `{{MediaUrl}}` (URL pseudo)
-   `{{Transcript}}` (si la transcripción de audio está habilitada)

Archivos adjuntos salientes del agente: incluye `MEDIA:<ruta-o-url>` en su propia línea (sin espacios). Ejemplo:

```
Aquí está la captura de pantalla.
MEDIA:https://example.com/screenshot.png
```

OpenClaw extrae estos y los envía como medios junto con el texto.

## Lista de verificación de operaciones

```bash
openclaw status          # estado local (credenciales, sesiones, eventos en cola)
openclaw status --all    # diagnóstico completo (solo lectura, para pegar)
openclaw status --deep   # añade sondeos de salud de la puerta de enlace (Telegram + Discord)
openclaw health --json   # instantánea de salud de la puerta de enlace (WS)
```

Los registros se encuentran en `/tmp/openclaw/` (predeterminado: `openclaw-YYYY-MM-DD.log`).

## Próximos pasos

-   WebChat: [WebChat](../web/webchat.md)
-   Operaciones de la puerta de enlace: [Manual de operaciones de la puerta de enlace](../gateway.md)
-   Cron + activaciones: [Trabajos Cron](../automation/cron-jobs.md)
-   Aplicación de barra de menús de macOS: [Aplicación OpenClaw para macOS](../platforms/macos.md)
-   Aplicación de nodo para iOS: [Aplicación iOS](../platforms/ios.md)
-   Aplicación de nodo para Android: [Aplicación Android](../platforms/android.md)
-   Estado de Windows: [Windows (WSL2)](../platforms/windows.md)
-   Estado de Linux: [Aplicación Linux](../platforms/linux.md)
-   Seguridad: [Seguridad](../gateway/security.md)

[Configuración inicial: Aplicación macOS](./onboarding.md)[Referencia CLI](./wizard-cli-reference.md)