

  Medios y dispositivos

  
# Nodos

Un **nodo** es un dispositivo complementario (macOS/iOS/Android/sin interfaz) que se conecta al **WebSocket** de la Puerta de Enlace (mismo puerto que los operadores) con `role: "node"` y expone una superficie de comandos (ej. `canvas.*`, `camera.*`, `device.*`, `notifications.*`, `system.*`) mediante `node.invoke`. Detalles del protocolo: [Protocolo de Puerta de Enlace](./gateway/protocol.md). Transporte heredado: [Protocolo Bridge](./gateway/bridge-protocol.md) (TCP JSONL; obsoleto/eliminado para nodos actuales). macOS también puede ejecutarse en **modo nodo**: la aplicación de la barra de menús se conecta al servidor WS de la Puerta de Enlace y expone sus comandos locales de lienzo/cámara como un nodo (así `openclaw nodes …` funciona contra este Mac). Notas:

-   Los nodos son **periféricos**, no puertas de enlace. No ejecutan el servicio de puerta de enlace.
-   Los mensajes de Telegram/WhatsApp/etc. llegan a la **puerta de enlace**, no a los nodos.
-   Manual de solución de problemas: [/nodes/troubleshooting](./nodes/troubleshooting.md)

## Emparejamiento + estado

**Los nodos WS usan emparejamiento de dispositivos.** Los nodos presentan una identidad de dispositivo durante `connect`; la Puerta de Enlace crea una solicitud de emparejamiento de dispositivo para `role: node`. Aprueba mediante la CLI de dispositivos (o la UI). CLI rápida:

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
```

Notas:

-   `nodes status` marca un nodo como **emparejado** cuando su rol de emparejamiento de dispositivo incluye `node`.
-   `node.pair.*` (CLI: `openclaw nodes pending/approve/reject`) es un almacén de emparejamiento de nodos separado propiedad de la puerta de enlace; **no** controla el protocolo de enlace `connect` de WS.

## Host de nodo remoto (system.run)

Usa un **host de nodo** cuando tu Puerta de Enlace se ejecuta en una máquina y quieres que los comandos se ejecuten en otra. El modelo aún se comunica con la **puerta de enlace**; la puerta de enlace reenvía las llamadas `exec` al **host de nodo** cuando se selecciona `host=node`.

### Qué se ejecuta dónde

-   **Host de la puerta de enlace**: recibe mensajes, ejecuta el modelo, enruta las llamadas a herramientas.
-   **Host del nodo**: ejecuta `system.run`/`system.which` en la máquina del nodo.
-   **Aprobaciones**: se aplican en el host del nodo mediante `~/.openclaw/exec-approvals.json`.

### Iniciar un host de nodo (en primer plano)

En la máquina del nodo:

```bash
openclaw node run --host <gateway-host> --port 18789 --display-name "Build Node"
```

### Puerta de enlace remota a través de túnel SSH (enlace a loopback)

Si la Puerta de Enlace se enlaza a loopback (`gateway.bind=loopback`, predeterminado en modo local), los hosts de nodo remotos no pueden conectarse directamente. Crea un túnel SSH y apunta el host del nodo al extremo local del túnel. Ejemplo (host del nodo -> host de la puerta de enlace):

```bash
# Terminal A (mantener ejecutándose): reenvía local 18790 -> puerta de enlace 127.0.0.1:18789
ssh -N -L 18790:127.0.0.1:18789 user@gateway-host

# Terminal B: exporta el token de la puerta de enlace y conéctate a través del túnel
export OPENCLAW_GATEWAY_TOKEN="<gateway-token>"
openclaw node run --host 127.0.0.1 --port 18790 --display-name "Build Node"
```

Notas:

-   El token es `gateway.auth.token` de la configuración de la puerta de enlace (`~/.openclaw/openclaw.json` en el host de la puerta de enlace).
-   `openclaw node run` lee `OPENCLAW_GATEWAY_TOKEN` para autenticación.

### Iniciar un host de nodo (servicio)

```bash
openclaw node install --host <gateway-host> --port 18789 --display-name "Build Node"
openclaw node restart
```

### Emparejar + nombrar

En el host de la puerta de enlace:

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw nodes status
```

Opciones de nombramiento:

-   `--display-name` en `openclaw node run` / `openclaw node install` (persiste en `~/.openclaw/node.json` en el nodo).
-   `openclaw nodes rename --node <id|name|ip> --name "Build Node"` (anulación en la puerta de enlace).

### Lista blanca de comandos

Las aprobaciones de ejecución son **por host de nodo**. Agrega entradas a la lista blanca desde la puerta de enlace:

```bash
openclaw approvals allowlist add --node <id|name|ip> "/usr/bin/uname"
openclaw approvals allowlist add --node <id|name|ip> "/usr/bin/sw_vers"
```

Las aprobaciones residen en el host del nodo en `~/.openclaw/exec-approvals.json`.

### Apuntar exec al nodo

Configurar valores predeterminados (configuración de la puerta de enlace):

```bash
openclaw config set tools.exec.host node
openclaw config set tools.exec.security allowlist
openclaw config set tools.exec.node "<id-or-name>"
```

O por sesión:

```bash
/exec host=node security=allowlist node=<id-or-name>
```

Una vez configurado, cualquier llamada `exec` con `host=node` se ejecuta en el host del nodo (sujeto a la lista blanca/aprobaciones del nodo). Relacionado:

-   [CLI del host de nodo](./cli/node.md)
-   [Herramienta Exec](./tools/exec.md)
-   [Aprobaciones de Exec](./tools/exec-approvals.md)

## Invocación de comandos

Bajo nivel (RPC crudo):

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command canvas.eval --params '{"javaScript":"location.href"}'
```

Existen ayudantes de alto nivel para los flujos de trabajo comunes de "dar al agente un archivo adjunto MEDIA".

## Capturas de pantalla (instantáneas del lienzo)

Si el nodo está mostrando el Lienzo (WebView), `canvas.snapshot` devuelve `{ format, base64 }`. Ayudante CLI (escribe en un archivo temporal e imprime `MEDIA:`):

```bash
openclaw nodes canvas snapshot --node <idOrNameOrIp> --format png
openclaw nodes canvas snapshot --node <idOrNameOrIp> --format jpg --max-width 1200 --quality 0.9
```

### Controles del lienzo

```bash
openclaw nodes canvas present --node <idOrNameOrIp> --target https://example.com
openclaw nodes canvas hide --node <idOrNameOrIp>
openclaw nodes canvas navigate https://example.com --node <idOrNameOrIp>
openclaw nodes canvas eval --node <idOrNameOrIp> --js "document.title"
```

Notas:

-   `canvas present` acepta URLs o rutas de archivos locales (`--target`), más opcional `--x/--y/--width/--height` para posicionamiento.
-   `canvas eval` acepta JS en línea (`--js`) o un argumento posicional.

### A2UI (Lienzo)

```bash
openclaw nodes canvas a2ui push --node <idOrNameOrIp> --text "Hello"
openclaw nodes canvas a2ui push --node <idOrNameOrIp> --jsonl ./payload.jsonl
openclaw nodes canvas a2ui reset --node <idOrNameOrIp>
```

Notas:

-   Solo se admite A2UI v0.8 JSONL (v0.9/createSurface es rechazado).

## Fotos + videos (cámara del nodo)

Fotos (`jpg`):

```bash
openclaw nodes camera list --node <idOrNameOrIp>
openclaw nodes camera snap --node <idOrNameOrIp>            # predeterminado: ambas orientaciones (2 líneas MEDIA)
openclaw nodes camera snap --node <idOrNameOrIp> --facing front
```

Fragmentos de video (`mp4`):

```bash
openclaw nodes camera clip --node <idOrNameOrIp> --duration 10s
openclaw nodes camera clip --node <idOrNameOrIp> --duration 3000 --no-audio
```

Notas:

-   El nodo debe estar en **primer plano** para `canvas.*` y `camera.*` (las llamadas en segundo plano devuelven `NODE_BACKGROUND_UNAVAILABLE`).
-   La duración del fragmento está limitada (actualmente `<= 60s`) para evitar cargas útiles base64 demasiado grandes.
-   Android solicitará permisos `CAMERA`/`RECORD_AUDIO` cuando sea posible; los permisos denegados fallan con `*_PERMISSION_REQUIRED`.

## Grabaciones de pantalla (nodos)

Los nodos exponen `screen.record` (mp4). Ejemplo:

```bash
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10 --no-audio
```

Notas:

-   `screen.record` requiere que la aplicación del nodo esté en primer plano.
-   Android mostrará la solicitud de captura de pantalla del sistema antes de grabar.
-   Las grabaciones de pantalla están limitadas a `<= 60s`.
-   `--no-audio` desactiva la captura del micrófono (compatible con iOS/Android; macOS usa captura de audio del sistema).
-   Usa `--screen ` para seleccionar una pantalla cuando hay múltiples disponibles.

## Ubicación (nodos)

Los nodos exponen `location.get` cuando la Ubicación está habilitada en la configuración. Ayudante CLI:

```bash
openclaw nodes location get --node <idOrNameOrIp>
openclaw nodes location get --node <idOrNameOrIp> --accuracy precise --max-age 15000 --location-timeout 10000
```

Notas:

-   La ubicación está **desactivada por defecto**.
-   "Siempre" requiere permiso del sistema; la obtención en segundo plano es de mejor esfuerzo.
-   La respuesta incluye lat/lon, precisión (metros) y marca de tiempo.

## SMS (nodos Android)

Los nodos Android pueden exponer `sms.send` cuando el usuario otorga permiso **SMS** y el dispositivo admite telefonía. Invocación de bajo nivel:

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command sms.send --params '{"to":"+15555550123","message":"Hello from OpenClaw"}'
```

Notas:

-   La solicitud de permiso debe ser aceptada en el dispositivo Android antes de que se anuncie la capacidad.
-   Los dispositivos solo con Wi-Fi sin telefonía no anunciarán `sms.send`.

## Comandos de dispositivo Android y datos personales

Los nodos Android pueden anunciar familias de comandos adicionales cuando se habilitan las capacidades correspondientes. Familias disponibles:

-   `device.status`, `device.info`, `device.permissions`, `device.health`
-   `notifications.list`, `notifications.actions`
-   `photos.latest`
-   `contacts.search`, `contacts.add`
-   `calendar.events`, `calendar.add`
-   `motion.activity`, `motion.pedometer`
-   `app.update`

Ejemplos de invocación:

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command device.status --params '{}'
openclaw nodes invoke --node <idOrNameOrIp> --command notifications.list --params '{}'
openclaw nodes invoke --node <idOrNameOrIp> --command photos.latest --params '{"limit":1}'
```

Notas:

-   Los comandos de movimiento están limitados por capacidades según los sensores disponibles.
-   `app.update` está limitado por permisos y políticas del tiempo de ejecución del nodo.

## Comandos del sistema (host de nodo / nodo mac)

El nodo macOS expone `system.run`, `system.notify` y `system.execApprovals.get/set`. El host de nodo sin interfaz expone `system.run`, `system.which` y `system.execApprovals.get/set`. Ejemplos:

```bash
openclaw nodes run --node <idOrNameOrIp> -- echo "Hello from mac node"
openclaw nodes notify --node <idOrNameOrIp> --title "Ping" --body "Gateway ready"
```

Notas:

-   `system.run` devuelve stdout/stderr/código de salida en la carga útil.
-   `system.notify` respeta el estado del permiso de notificación en la aplicación macOS.
-   Los metadatos de `platform` / `deviceFamily` de nodo no reconocidos usan una lista blanca predeterminada conservadora que excluye `system.run` y `system.which`. Si intencionalmente necesitas esos comandos para una plataforma desconocida, agrégalos explícitamente mediante `gateway.nodes.allowCommands`.
-   `system.run` admite `--cwd`, `--env KEY=VAL`, `--command-timeout` y `--needs-screen-recording`.
-   Para envoltorios de shell (`bash|sh|zsh ... -c/-lc`), los valores `--env` de ámbito de solicitud se reducen a una lista blanca explícita (`TERM`, `LANG`, `LC_*`, `COLORTERM`, `NO_COLOR`, `FORCE_COLOR`).
-   Para decisiones de permitir-siempre en modo lista blanca, los envoltorios de despacho conocidos (`env`, `nice`, `nohup`, `stdbuf`, `timeout`) persisten las rutas del ejecutable interno en lugar de las rutas del envoltorio. Si el desenvuelto no es seguro, no se persiste automáticamente ninguna entrada en la lista blanca.
-   En hosts de nodo Windows en modo lista blanca, las ejecuciones de envoltorio de shell a través de `cmd.exe /c` requieren aprobación (la entrada en la lista blanca por sí sola no permite automáticamente la forma del envoltorio).
-   `system.notify` admite `--priority <passive|active|timeSensitive>` y `--delivery <system|overlay|auto>`.
-   Los hosts de nodo ignoran las anulaciones de `PATH` y eliminan claves peligrosas de inicio/shell (`DYLD_*`, `LD_*`, `NODE_OPTIONS`, `PYTHON*`, `PERL*`, `RUBYOPT`, `SHELLOPTS`, `PS4`). Si necesitas entradas PATH adicionales, configura el entorno del servicio del host de nodo (o instala herramientas en ubicaciones estándar) en lugar de pasar `PATH` mediante `--env`.
-   En modo nodo macOS, `system.run` está controlado por aprobaciones de ejecución en la aplicación macOS (Configuración → Aprobaciones de ejecución). Preguntar/lista blanca/completo se comportan igual que en el host de nodo sin interfaz; las solicitudes denegadas devuelven `SYSTEM_RUN_DENIED`.
-   En el host de nodo sin interfaz, `system.run` está controlado por aprobaciones de ejecución (`~/.openclaw/exec-approvals.json`).

## Enlace de exec a nodo

Cuando hay múltiples nodos disponibles, puedes enlazar exec a un nodo específico. Esto establece el nodo predeterminado para `exec host=node` (y puede ser anulado por agente). Predeterminado global:

```bash
openclaw config set tools.exec.node "node-id-or-name"
```

Anulación por agente:

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

Deshacer para permitir cualquier nodo:

```bash
openclaw config unset tools.exec.node
openclaw config unset agents.list[0].tools.exec.node
```

## Mapa de permisos

Los nodos pueden incluir un mapa `permissions` en `node.list` / `node.describe`, indexado por nombre de permiso (ej. `screenRecording`, `accessibility`) con valores booleanos (`true` = concedido).

## Host de nodo sin interfaz (multiplataforma)

OpenClaw puede ejecutar un **host de nodo sin interfaz** (sin UI) que se conecta al WebSocket de la Puerta de Enlace y expone `system.run` / `system.which`. Esto es útil en Linux/Windows o para ejecutar un nodo mínimo junto a un servidor. Inícialo:

```bash
openclaw node run --host <gateway-host> --port 18789
```

Notas:

-   El emparejamiento aún es necesario (la Puerta de Enlace mostrará una solicitud de emparejamiento de dispositivo).
-   El host del nodo almacena su id de nodo, token, nombre para mostrar e información de conexión de la puerta de enlace en `~/.openclaw/node.json`.
-   Las aprobaciones de ejecución se aplican localmente mediante `~/.openclaw/exec-approvals.json` (ver [Aprobaciones de Exec](./tools/exec-approvals.md)).
-   En macOS, el host de nodo sin interfaz ejecuta `system.run` localmente por defecto. Establece `OPENCLAW_NODE_EXEC_HOST=app` para enrutar `system.run` a través del host de ejecución de la aplicación complementaria; agrega `OPENCLAW_NODE_EXEC_FALLBACK=0` para requerir el host de la aplicación y fallar cerrado si no está disponible.
-   Agrega `--tls` / `--tls-fingerprint` cuando el WS de la Puerta de Enlace usa TLS.

## Modo nodo Mac

-   La aplicación de la barra de menús de macOS se conecta al servidor WS de la Puerta de Enlace como un nodo (así `openclaw nodes …` funciona contra este Mac).
-   En modo remoto, la aplicación abre un túnel SSH para el puerto de la Puerta de Enlace y se conecta a `localhost`.

[Monitoreo de Autenticación](./automation/auth-monitoring.md)[Solución de Problemas de Nodos](./nodes/troubleshooting.md)