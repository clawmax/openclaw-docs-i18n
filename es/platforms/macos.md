title: "Guía de la aplicación OpenClaw para macOS: Aplicación de barra de menú y Gateway"
description: "Aprende a usar la aplicación de barra de menú de OpenClaw para macOS para gestionar permisos, conectarte al Gateway y exponer capacidades de macOS como Canvas y Grabación de Pantalla a agentes de IA."
keywords: ["openclaw macos", "aplicación barra de menú macos", "conexión gateway", "permisos tcc macos", "aprobaciones system.run", "modo remoto ssh", "control launchd", "automatización canvas"]
---

  Descripción general de plataformas

  
# Aplicación para macOS

La aplicación para macOS es el **compañero de la barra de menú** para OpenClaw. Es responsable de los permisos, gestiona/se conecta al Gateway localmente (launchd o manual) y expone las capacidades de macOS al agente como un nodo.

## Qué hace

-   Muestra notificaciones nativas y estado en la barra de menú.
-   Gestiona las solicitudes de TCC (Notificaciones, Accesibilidad, Grabación de Pantalla, Micrófono, Reconocimiento de Voz, Automatización/AppleScript).
-   Ejecuta o se conecta al Gateway (local o remoto).
-   Expone herramientas exclusivas de macOS (Canvas, Cámara, Grabación de Pantalla, `system.run`).
-   Inicia el servicio local del host del nodo en modo **remoto** (launchd) y lo detiene en modo **local**.
-   Opcionalmente aloja **PeekabooBridge** para automatización de UI.
-   Instala la CLI global (`openclaw`) vía npm/pnpm bajo petición (no se recomienda bun para el entorno de ejecución del Gateway).

## Modo local vs remoto

-   **Local** (por defecto): la aplicación se conecta a un Gateway local en ejecución si está presente; de lo contrario, habilita el servicio launchd mediante `openclaw gateway install`.
-   **Remoto**: la aplicación se conecta a un Gateway a través de SSH/Tailscale y nunca inicia un proceso local. La aplicación inicia el **servicio local del host del nodo** para que el Gateway remoto pueda alcanzar este Mac. La aplicación no genera el Gateway como un proceso hijo.

## Control de Launchd

La aplicación gestiona un LaunchAgent por usuario etiquetado como `ai.openclaw.gateway` (o `ai.openclaw.` cuando se usa `--profile`/`OPENCLAW_PROFILE`; el legado `com.openclaw.*` aún se descarga).

```bash
launchctl kickstart -k gui/$UID/ai.openclaw.gateway
launchctl bootout gui/$UID/ai.openclaw.gateway
```

Reemplaza la etiqueta por `ai.openclaw.` cuando ejecutes un perfil con nombre. Si el LaunchAgent no está instalado, habilítalo desde la aplicación o ejecuta `openclaw gateway install`.

## Capacidades del nodo (mac)

La aplicación de macOS se presenta como un nodo. Comandos comunes:

-   Canvas: `canvas.present`, `canvas.navigate`, `canvas.eval`, `canvas.snapshot`, `canvas.a2ui.*`
-   Cámara: `camera.snap`, `camera.clip`
-   Pantalla: `screen.record`
-   Sistema: `system.run`, `system.notify`

El nodo reporta un mapa de `permissions` para que los agentes puedan decidir qué está permitido. Servicio del nodo + IPC de la aplicación:

-   Cuando el servicio headless del host del nodo está en ejecución (modo remoto), se conecta al WS del Gateway como un nodo.
-   `system.run` se ejecuta en la aplicación de macOS (contexto de UI/TCC) a través de un socket Unix local; las solicitudes y la salida permanecen en la aplicación.

Diagrama (SCI):

```
Gateway -> Servicio del Nodo (WS)
                 |  IPC (UDS + token + HMAC + TTL)
                 v
             Aplicación Mac (UI + TCC + system.run)
```

## Aprobaciones de ejecución (system.run)

`system.run` está controlado por las **Aprobaciones de ejecución** en la aplicación de macOS (Configuración → Aprobaciones de ejecución). La seguridad + preguntar + lista de permitidos se almacenan localmente en el Mac en:

```
~/.openclaw/exec-approvals.json
```

Ejemplo:

```json
{
  "version": 1,
  "defaults": {
    "security": "deny",
    "ask": "on-miss"
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "allowlist": [{ "pattern": "/opt/homebrew/bin/rg" }]
    }
  }
}
```

Notas:

-   Las entradas de `allowlist` son patrones glob para las rutas de binarios resueltas.
-   El texto de comando de shell en bruto que contiene sintaxis de control o expansión de shell (`&&`, `||`, `;`, `|`, ```, `$`, `<`, `>`, `(`, `)`) se trata como un fallo en la lista de permitidos y requiere aprobación explícita (o incluir el binario del shell en la lista de permitidos).
-   Elegir "Permitir siempre" en la solicitud agrega ese comando a la lista de permitidos.
-   Las anulaciones de entorno de `system.run` se filtran (elimina `PATH`, `DYLD_*`, `LD_*`, `NODE_OPTIONS`, `PYTHON*`, `PERL*`, `RUBYOPT`, `SHELLOPTS`, `PS4`) y luego se fusionan con el entorno de la aplicación.
-   Para envoltorios de shell (`bash|sh|zsh ... -c/-lc`), las anulaciones de entorno con alcance de solicitud se reducen a una pequeña lista de permitidos explícita (`TERM`, `LANG`, `LC_*`, `COLORTERM`, `NO_COLOR`, `FORCE_COLOR`).
-   Para decisiones de permitir-siempre en modo de lista de permitidos, los envoltorios de despacho conocidos (`env`, `nice`, `nohup`, `stdbuf`, `timeout`) persisten las rutas del ejecutable interno en lugar de las rutas del envoltorio. Si el desenvuelto no es seguro, no se persiste automáticamente ninguna entrada en la lista de permitidos.

## Enlaces profundos (Deep links)

La aplicación registra el esquema de URL `openclaw://` para acciones locales.

### openclaw://agent

Activa una solicitud `agent` del Gateway.

```bash
open 'openclaw://agent?message=Hello%20from%20deep%20link'
```

Parámetros de consulta:

-   `message` (requerido)
-   `sessionKey` (opcional)
-   `thinking` (opcional)
-   `deliver` / `to` / `channel` (opcional)
-   `timeoutSeconds` (opcional)
-   `key` (opcional clave para modo desatendido)

Seguridad:

-   Sin `key`, la aplicación solicita confirmación.
-   Sin `key`, la aplicación aplica un límite de mensaje corto para la solicitud de confirmación e ignora `deliver` / `to` / `channel`.
-   Con una `key` válida, la ejecución es desatendida (destinada a automatizaciones personales).

## Flujo de incorporación (típico)

1.  Instala y ejecuta **OpenClaw.app**.
2.  Completa la lista de verificación de permisos (solicitudes TCC).
3.  Asegúrate de que el modo **Local** esté activo y el Gateway esté en ejecución.
4.  Instala la CLI si deseas acceso por terminal.

## Ubicación del directorio de estado (macOS)

Evita colocar tu directorio de estado de OpenClaw en iCloud u otras carpetas sincronizadas en la nube. Las rutas respaldadas por sincronización pueden agregar latencia y ocasionalmente causar bloqueos de archivos/carreras de sincronización para sesiones y credenciales. Prefiere una ruta de estado local no sincronizada, como:

```
OPENCLAW_STATE_DIR=~/.openclaw
```

Si `openclaw doctor` detecta el estado bajo:

-   `~/Library/Mobile Documents/com~apple~CloudDocs/...`
-   `~/Library/CloudStorage/...`

avisará y recomendará volver a una ruta local.

## Flujo de trabajo de compilación y desarrollo (nativo)

-   `cd apps/macos && swift build`
-   `swift run OpenClaw` (o Xcode)
-   Empaquetar aplicación: `scripts/package-mac-app.sh`

## Depurar conectividad del gateway (CLI de macOS)

Usa la CLI de depuración para ejercitar la misma lógica de descubrimiento y protocolo de enlace WebSocket del Gateway que usa la aplicación de macOS, sin lanzar la aplicación.

```bash
cd apps/macos
swift run openclaw-mac connect --json
swift run openclaw-mac discover --timeout 3000 --json
```

Opciones de conexión:

-   `--url <ws://host:port>`: anular configuración
-   `--mode <local|remote>`: resolver desde configuración (por defecto: configuración o local)
-   `--probe`: forzar un sondeo de salud nuevo
-   `--timeout <ms>`: tiempo de espera de la solicitud (por defecto: `15000`)
-   `--json`: salida estructurada para comparar

Opciones de descubrimiento:

-   `--include-local`: incluir gateways que serían filtrados como "locales"
-   `--timeout <ms>`: ventana total de descubrimiento (por defecto: `2000`)
-   `--json`: salida estructurada para comparar

Consejo: compara con `openclaw gateway discover --json` para ver si la canalización de descubrimiento de la aplicación de macOS (NWBrowser + fallback DNS‑SD de tailnet) difiere del descubrimiento basado en `dns-sd` de la CLI de Node.

## Tuberías de conexión remota (túneles SSH)

Cuando la aplicación de macOS se ejecuta en modo **Remoto**, abre un túnel SSH para que los componentes de UI locales puedan comunicarse con un Gateway remoto como si estuviera en localhost.

### Túnel de control (puerto WebSocket del Gateway)

-   **Propósito:** comprobaciones de salud, estado, Web Chat, configuración y otras llamadas del plano de control.
-   **Puerto local:** el puerto del Gateway (por defecto `18789`), siempre estable.
-   **Puerto remoto:** el mismo puerto del Gateway en el host remoto.
-   **Comportamiento:** sin puerto local aleatorio; la aplicación reutiliza un túnel saludable existente o lo reinicia si es necesario.
-   **Forma SSH:** `ssh -N -L <local>:127.0.0.1:<remote>` con opciones BatchMode + ExitOnForwardFailure + keepalive.
-   **Reporte de IP:** el túnel SSH usa loopback, por lo que el gateway verá la IP del nodo como `127.0.0.1`. Usa el transporte **Directo (ws/wss)** si quieres que aparezca la IP real del cliente (ver [Acceso remoto en macOS](/platforms/mac/remote)).

Para los pasos de configuración, consulta [Acceso remoto en macOS](/platforms/mac/remote). Para detalles del protocolo, consulta [Protocolo del Gateway](/gateway/protocol).

## Documentación relacionada

-   [Manual del Gateway](/gateway)
-   [Gateway (macOS)](/platforms/mac/bundled-gateway)
-   [Permisos de macOS](/platforms/mac/permissions)
-   [Canvas](/platforms/mac/canvas)

[Plataformas](/platforms)[Aplicación Linux](/platforms/linux)

---