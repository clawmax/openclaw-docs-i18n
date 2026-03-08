title: "Configuración de Gateway OpenClaw en macOS y Gestión de Servicios Launchd"
description: "Aprende a instalar la CLI de openclaw y configurar el Gateway como un servicio launchd de macOS para la aplicación complementaria OpenClaw. Incluye comprobaciones de versión y solución de problemas."
keywords: ["macos openclaw", "configuración de gateway", "servicio launchd", "cli openclaw", "aplicación complementaria macos", "gateway local", "node gateway", "gestión de servicios"]
---

  Aplicación complementaria para macOS

  
# Gateway en macOS

OpenClaw.app ya no incluye Node/Bun ni el entorno de ejecución del Gateway. La aplicación de macOS espera una instalación **externa** de la CLI `openclaw`, no inicia el Gateway como un proceso hijo, y gestiona un servicio launchd por usuario para mantener el Gateway en ejecución (o se conecta a un Gateway local existente si ya hay uno en funcionamiento).

## Instalar la CLI (requerido para el modo local)

Necesitas Node 22+ en el Mac, luego instala `openclaw` globalmente:

```bash
npm install -g openclaw@<version>
```

El botón **Instalar CLI** de la aplicación de macOS ejecuta el mismo flujo a través de npm/pnpm (no se recomienda bun para el entorno de ejecución del Gateway).

## Launchd (Gateway como LaunchAgent)

Etiqueta:

-   `ai.openclaw.gateway` (o `ai.openclaw.;` las antiguas `com.openclaw.*` pueden permanecer)

Ubicación del Plist (por usuario):

-   `~/Library/LaunchAgents/ai.openclaw.gateway.plist` (o `~/Library/LaunchAgents/ai.openclaw..plist`)

Gestor:

-   La aplicación de macOS es responsable de la instalación/actualización del LaunchAgent en el modo Local.
-   La CLI también puede instalarlo: `openclaw gateway install`.

Comportamiento:

-   "OpenClaw Activo" habilita/deshabilita el LaunchAgent.
-   Cerrar la aplicación **no** detiene el gateway (launchd lo mantiene activo).
-   Si ya hay un Gateway ejecutándose en el puerto configurado, la aplicación se conecta a él en lugar de iniciar uno nuevo.

Registros (Logging):

-   stdout/err de launchd: `/tmp/openclaw/openclaw-gateway.log`

## Compatibilidad de versiones

La aplicación de macOS verifica la versión del gateway con su propia versión. Si son incompatibles, actualiza la CLI global para que coincida con la versión de la aplicación.

## Comprobación rápida (Smoke check)

```bash
openclaw --version

OPENCLAW_SKIP_CHANNELS=1 \
OPENCLAW_SKIP_CANVAS_HOST=1 \
openclaw gateway --port 18999 --bind loopback
```

Luego:

```bash
openclaw gateway call health --url ws://127.0.0.1:18999 --timeout 3000
```

[Lanzamiento de macOS](./release.md)[IPC de macOS](./xpc.md)

---