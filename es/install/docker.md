

  Otros métodos de instalación

  
# Docker

Docker es **opcional**. Úsalo solo si quieres una puerta de enlace contenedorizada o para validar el flujo de Docker.

## ¿Es Docker adecuado para mí?

-   **Sí**: quieres un entorno de puerta de enlace aislado y desechable o ejecutar OpenClaw en un host sin instalaciones locales.
-   **No**: estás ejecutando en tu propia máquina y solo quieres el ciclo de desarrollo más rápido. Usa el flujo de instalación normal en su lugar.
-   **Nota sobre aislamiento**: el aislamiento de agentes también usa Docker, pero **no** requiere que toda la puerta de enlace se ejecute en Docker. Consulta [Aislamiento](../gateway/sandboxing.md).

Esta guía cubre:

-   Puerta de enlace contenedorizada (OpenClaw completo en Docker)
-   Aislamiento de agente por sesión (puerta de enlace en el host + herramientas de agente aisladas en Docker)

Detalles del aislamiento: [Aislamiento](../gateway/sandboxing.md)

## Requisitos

-   Docker Desktop (o Docker Engine) + Docker Compose v2
-   Al menos 2 GB de RAM para la construcción de la imagen (`pnpm install` puede ser eliminado por falta de memoria en hosts de 1 GB con salida 137)
-   Suficiente disco para imágenes + registros
-   Si se ejecuta en un VPS/host público, revisa [Refuerzo de seguridad para exposición de red](../gateway/security.md#04-network-exposure-bind--port--firewall), especialmente la política de firewall `DOCKER-USER` de Docker.

## Puerta de enlace contenedorizada (Docker Compose)

### Inicio rápido (recomendado)

> **ℹ️** Los valores predeterminados de Docker aquí asumen modos de enlace (`lan`/`loopback`), no alias de host. Usa valores de modo de enlace en `gateway.bind` (por ejemplo `lan` o `loopback`), no alias de host como `0.0.0.0` o `localhost`.

 Desde la raíz del repositorio:

```
./docker-setup.sh
```

Este script:

-   construye la imagen de la puerta de enlace localmente (o descarga una imagen remota si `OPENCLAW_IMAGE` está configurada)
-   ejecuta el asistente de configuración inicial
-   imprime sugerencias opcionales de configuración de proveedores
-   inicia la puerta de enlace mediante Docker Compose
-   genera un token de puerta de enlace y lo escribe en `.env`

Variables de entorno opcionales:

-   `OPENCLAW_IMAGE` — usa una imagen remota en lugar de construir localmente (ej. `ghcr.io/openclaw/openclaw:latest`)
-   `OPENCLAW_DOCKER_APT_PACKAGES` — instala paquetes apt adicionales durante la construcción
-   `OPENCLAW_EXTENSIONS` — pre-instala dependencias de extensiones en el momento de la construcción (nombres de extensiones separados por espacios, ej. `diagnostics-otel matrix`)
-   `OPENCLAW_EXTRA_MOUNTS` — añade montajes de host adicionales
-   `OPENCLAW_HOME_VOLUME` — persiste `/home/node` en un volumen con nombre
-   `OPENCLAW_SANDBOX` — opta por el arranque del sandbox de puerta de enlace Docker. Solo los valores explícitamente verdaderos lo habilitan: `1`, `true`, `yes`, `on`
-   `OPENCLAW_INSTALL_DOCKER_CLI` — argumento de construcción para builds de imagen local (`1` instala Docker CLI en la imagen). `docker-setup.sh` lo configura automáticamente cuando `OPENCLAW_SANDBOX=1` para builds locales.
-   `OPENCLAW_DOCKER_SOCKET` — sobrescribe la ruta del socket de Docker (predeterminado: ruta `DOCKER_HOST=unix://...`, si no `/var/run/docker.sock`)
-   `OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1` — rompe-cristales: permite objetivos `ws://` de red privada confiable para rutas de cliente/asistente (predeterminado es solo loopback)
-   `OPENCLAW_BROWSER_DISABLE_GRAPHICS_FLAGS=0` — deshabilita las banderas de refuerzo del navegador en contenedor `--disable-3d-apis`, `--disable-software-rasterizer`, `--disable-gpu` cuando necesitas compatibilidad WebGL/3D.
-   `OPENCLAW_BROWSER_DISABLE_EXTENSIONS=0` — mantiene las extensiones habilitadas cuando los flujos del navegador las requieren (predeterminado mantiene las extensiones deshabilitadas en el navegador del sandbox).
-   `OPENCLAW_BROWSER_RENDERER_PROCESS_LIMIT=` — establece el límite de procesos de renderizado de Chromium; configúralo a `0` para omitir la bandera y usar el comportamiento predeterminado de Chromium.

Después de que termine:

-   Abre `http://127.0.0.1:18789/` en tu navegador.
-   Pega el token en la Interfaz de Control (Configuración → token).
-   ¿Necesitas la URL de nuevo? Ejecuta `docker compose run --rm openclaw-cli dashboard --no-open`.

### Habilitar sandbox de agente para puerta de enlace Docker (opt-in)

`docker-setup.sh` también puede preparar `agents.defaults.sandbox.*` para despliegues Docker. Habilítalo con:

```bash
export OPENCLAW_SANDBOX=1
./docker-setup.sh
```

Ruta de socket personalizada (por ejemplo Docker sin privilegios):

```bash
export OPENCLAW_SANDBOX=1
export OPENCLAW_DOCKER_SOCKET=/run/user/1000/docker.sock
./docker-setup.sh
```

Notas:

-   El script monta `docker.sock` solo después de que pasen los prerrequisitos del sandbox.
-   Si la configuración del sandbox no se puede completar, el script restablece `agents.defaults.sandbox.mode` a `off` para evitar configuraciones de sandbox obsoletas/rotas en re-ejecuciones.
-   Si falta `Dockerfile.sandbox`, el script imprime una advertencia y continúa; construye `openclaw-sandbox:bookworm-slim` con `scripts/sandbox-setup.sh` si es necesario.
-   Para valores `OPENCLAW_IMAGE` no locales, la imagen ya debe contener soporte de Docker CLI para la ejecución del sandbox.

### Automatización/CI (no interactivo, sin ruido TTY)

Para scripts y CI, deshabilita la asignación de pseudo-TTY de Compose con `-T`:

```bash
docker compose run -T --rm openclaw-cli gateway probe
docker compose run -T --rm openclaw-cli devices list --json
```

Si tu automatización no exporta variables de sesión de Claude, dejarlas sin configurar ahora se resuelve a valores vacíos por defecto en `docker-compose.yml` para evitar repetidas advertencias de "variable no configurada".

### Nota de seguridad de red compartida (CLI + puerta de enlace)

`openclaw-cli` usa `network_mode: "service:openclaw-gateway"` para que los comandos CLI puedan alcanzar de manera confiable la puerta de enlace a través de `127.0.0.1` en Docker. Trata esto como un límite de confianza compartido: el enlace loopback no es aislamiento entre estos dos contenedores. Si necesitas una separación más fuerte, ejecuta comandos desde una ruta de red de contenedor/host separada en lugar del servicio `openclaw-cli` incluido. Para reducir el impacto si el proceso CLI se ve comprometido, la configuración de compose elimina `NET_RAW`/`NET_ADMIN` y habilita `no-new-privileges` en `openclaw-cli`. Escribe configuración/espacio de trabajo en el host:

-   `~/.openclaw/`
-   `~/.openclaw/workspace`

¿Ejecutando en un VPS? Consulta [Hetzner (VPS Docker)](./hetzner.md).

### Usar una imagen remota (omitir construcción local)

Las imágenes preconstruidas oficiales se publican en:

-   [Paquete del Registro de Contenedores de GitHub](https://github.com/openclaw/openclaw/pkgs/container/openclaw)

Usa el nombre de imagen `ghcr.io/openclaw/openclaw` (no las imágenes de Docker Hub con nombres similares). Etiquetas comunes:

-   `main` — última construcción desde `main`
-   `` — construcciones de etiquetas de lanzamiento (por ejemplo `2026.2.26`)
-   `latest` — última etiqueta de lanzamiento estable

### Metadatos de la imagen base

La imagen principal de Docker actualmente usa:

-   `node:22-bookworm`

La imagen de docker ahora publica anotaciones de imagen base OCI (sha256 es un ejemplo):

-   `org.opencontainers.image.base.name=docker.io/library/node:22-bookworm`
-   `org.opencontainers.image.base.digest=sha256:6d735b4d33660225271fda0a412802746658c3a1b975507b2803ed299609760a`
-   `org.opencontainers.image.source=https://github.com/openclaw/openclaw`
-   `org.opencontainers.image.url=https://openclaw.ai`
-   `org.opencontainers.image.documentation=https://docs.openclaw.ai/install/docker`
-   `org.opencontainers.image.licenses=MIT`
-   `org.opencontainers.image.title=OpenClaw`
-   `org.opencontainers.image.description=Imagen de contenedor de tiempo de ejecución de puerta de enlace y CLI de OpenClaw`
-   `org.opencontainers.image.revision=<git-sha>`
-   `org.opencontainers.image.version=<tag-or-main>`
-   `org.opencontainers.image.created=`

Referencia: [Anotaciones de imagen OCI](https://github.com/opencontainers/image-spec/blob/main/annotations.md) Contexto de lanzamiento: el historial etiquetado de este repositorio ya usa Bookworm en `v2026.2.22` y etiquetas anteriores de 2026 (por ejemplo `v2026.2.21`, `v2026.2.9`). Por defecto, el script de configuración construye la imagen desde el código fuente. Para descargar una imagen preconstruida en su lugar, configura `OPENCLAW_IMAGE` antes de ejecutar el script:

```bash
export OPENCLAW_IMAGE="ghcr.io/openclaw/openclaw:latest"
./docker-setup.sh
```

El script detecta que `OPENCLAW_IMAGE` no es el valor predeterminado `openclaw:local` y ejecuta `docker pull` en lugar de `docker build`. Todo lo demás (configuración inicial, inicio de puerta de enlace, generación de token) funciona de la misma manera. `docker-setup.sh` aún se ejecuta desde la raíz del repositorio porque usa el `docker-compose.yml` local y archivos auxiliares. `OPENCLAW_IMAGE` omite el tiempo de construcción de imagen local; no reemplaza el flujo de trabajo de compose/configuración.

### Ayudantes de Shell (opcional)

Para una gestión de Docker más fácil día a día, instala `ClawDock`:

```bash
mkdir -p ~/.clawdock && curl -sL https://raw.githubusercontent.com/openclaw/openclaw/main/scripts/shell-helpers/clawdock-helpers.sh -o ~/.clawdock/clawdock-helpers.sh
```

**Añade a tu configuración de shell (zsh):**

```bash
echo 'source ~/.clawdock/clawdock-helpers.sh' >> ~/.zshrc && source ~/.zshrc
```

Luego usa `clawdock-start`, `clawdock-stop`, `clawdock-dashboard`, etc. Ejecuta `clawdock-help` para todos los comandos. Consulta el README del ayudante [`ClawDock`](https://github.com/openclaw/openclaw/blob/main/scripts/shell-helpers/README.md) para más detalles.

### Flujo manual (compose)

```bash
docker build -t openclaw:local -f Dockerfile .
docker compose run --rm openclaw-cli onboard
docker compose up -d openclaw-gateway
```

Nota: ejecuta `docker compose ...` desde la raíz del repositorio. Si habilitaste `OPENCLAW_EXTRA_MOUNTS` o `OPENCLAW_HOME_VOLUME`, el script de configuración escribe `docker-compose.extra.yml`; inclúyelo cuando ejecutes Compose en otro lugar:

```bash
docker compose -f docker-compose.yml -f docker-compose.extra.yml <command>
```

### Token de Interfaz de Control + emparejamiento (Docker)

Si ves "no autorizado" o "desconectado (1008): emparejamiento requerido", obtén un enlace de panel de control nuevo y aprueba el dispositivo del navegador:

```bash
docker compose run --rm openclaw-cli dashboard --no-open
docker compose run --rm openclaw-cli devices list
docker compose run --rm openclaw-cli devices approve <requestId>
```

Más detalle: [Panel de control](../web/dashboard.md), [Dispositivos](../cli/devices.md).

### Montajes extra (opcional)

Si quieres montar directorios de host adicionales en los contenedores, configura `OPENCLAW_EXTRA_MOUNTS` antes de ejecutar `docker-setup.sh`. Esto acepta una lista separada por comas de montajes de enlace de Docker y los aplica tanto a `openclaw-gateway` como a `openclaw-cli` generando `docker-compose.extra.yml`. Ejemplo:

```bash
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

Notas:

-   Las rutas deben ser compartidas con Docker Desktop en macOS/Windows.
-   Cada entrada debe ser `fuente:destino[:opciones]` sin espacios, tabulaciones o saltos de línea.
-   Si editas `OPENCLAW_EXTRA_MOUNTS`, vuelve a ejecutar `docker-setup.sh` para regenerar el archivo compose extra.
-   `docker-compose.extra.yml` es generado. No lo edites manualmente.

### Persistir todo el directorio home del contenedor (opcional)

Si quieres que `/home/node` persista a través de la recreación del contenedor, configura un volumen con nombre mediante `OPENCLAW_HOME_VOLUME`. Esto crea un volumen de Docker y lo monta en `/home/node`, manteniendo los montajes de enlace estándar de configuración/espacio de trabajo. Usa un volumen con nombre aquí (no una ruta de enlace); para montajes de enlace, usa `OPENCLAW_EXTRA_MOUNTS`. Ejemplo:

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
./docker-setup.sh
```

Puedes combinar esto con montajes extra:

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

Notas:

-   Los volúmenes con nombre deben coincidir con `^[A-Za-z0-9][A-Za-z0-9_.-]*$`.
-   Si cambias `OPENCLAW_HOME_VOLUME`, vuelve a ejecutar `docker-setup.sh` para regenerar el archivo compose extra.
-   El volumen con nombre persiste hasta que se elimina con `docker volume rm `.

### Instalar paquetes apt extra (opcional)

Si necesitas paquetes del sistema dentro de la imagen (por ejemplo, herramientas de construcción o bibliotecas multimedia), configura `OPENCLAW_DOCKER_APT_PACKAGES` antes de ejecutar `docker-setup.sh`. Esto instala los paquetes durante la construcción de la imagen, por lo que persisten incluso si se elimina el contenedor. Ejemplo:

```bash
export OPENCLAW_DOCKER_APT_PACKAGES="ffmpeg build-essential"
./docker-setup.sh
```

Notas:

-   Esto acepta una lista separada por espacios de nombres de paquetes apt.
-   Si cambias `OPENCLAW_DOCKER_APT_PACKAGES`, vuelve a ejecutar `docker-setup.sh` para reconstruir la imagen.

### Pre-instalar dependencias de extensiones (opcional)

Las extensiones con su propio `package.json` (ej. `diagnostics-otel`, `matrix`, `msteams`) instalan sus dependencias npm en la primera carga. Para integrar esas dependencias en la imagen, configura `OPENCLAW_EXTENSIONS` antes de ejecutar `docker-setup.sh`:

```bash
export OPENCLAW_EXTENSIONS="diagnostics-otel matrix"
./docker-setup.sh
```

O al construir directamente:

```bash
docker build --build-arg OPENCLAW_EXTENSIONS="diagnostics-otel matrix" .
```

Notas:

-   Esto acepta una lista separada por espacios de nombres de directorios de extensiones (bajo `extensions/`).
-   Solo las extensiones con un `package.json` se ven afectadas; los plugins ligeros sin uno son ignorados.
-   Si cambias `OPENCLAW_EXTENSIONS`, vuelve a ejecutar `docker-setup.sh` para reconstruir la imagen.

### Contenedor para usuarios avanzados / con todas las funciones (opt-in)

La imagen de Docker predeterminada es **primero seguridad** y se ejecuta como el usuario no root `node`. Esto mantiene la superficie de ataque pequeña, pero significa:

-   no hay instalaciones de paquetes del sistema en tiempo de ejecución
-   no hay Homebrew por defecto
-   no hay navegadores Chromium/Playwright incluidos

Si quieres un contenedor más completo, usa estas opciones de habilitación:

1.  **Persistir `/home/node`** para que las descargas de navegadores y cachés de herramientas sobrevivan:

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
./docker-setup.sh
```

2.  **Integrar dependencias del sistema en la imagen** (repetible + persistente):

```bash
export OPENCLAW_DOCKER_APT_PACKAGES="git curl jq"
./docker-setup.sh
```

3.  **Instalar navegadores Playwright sin `npx`** (evita conflictos de anulación de npm):

```bash
docker compose run --rm openclaw-cli \
  node /app/node_modules/playwright-core/cli.js install chromium
```

Si necesitas que Playwright instale dependencias del sistema, reconstruye la imagen con `OPENCLAW_DOCKER_APT_PACKAGES` en lugar de usar `--with-deps` en tiempo de ejecución.

4.  **Persistir descargas de navegadores Playwright**:

-   Configura `PLAYWRIGHT_BROWSERS_PATH=/home/node/.cache/ms-playwright` en `docker-compose.yml`.
-   Asegúrate de que `/home/node` persista mediante `OPENCLAW_HOME_VOLUME`, o monta `/home/node/.cache/ms-playwright` mediante `OPENCLAW_EXTRA_MOUNTS`.

### Permisos + EACCES

La imagen se ejecuta como `node` (uid 1000). Si ves errores de permisos en `/home/node/.openclaw`, asegúrate de que tus montajes de enlace de host sean propiedad del uid 1000. Ejemplo (host Linux):

```bash
sudo chown -R 1000:1000 /path/to/openclaw-config /path/to/openclaw-workspace
```

Si eliges ejecutar como root por conveniencia, aceptas la compensación de seguridad.

### Reconstrucciones más rápidas (recomendado)

Para acelerar las reconstrucciones, ordena tu Dockerfile para que las capas de dependencia se almacenen en caché. Esto evita volver a ejecutar `pnpm install` a menos que cambien los archivos de bloqueo:

```dockerfile
FROM node:22-bookworm

# Instalar Bun (requerido para scripts de construcción)
RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/root/.bun/bin:${PATH}"

RUN corepack enable

WORKDIR /app

# Almacenar dependencias en caché a menos que cambien los metadatos del paquete
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

### Configuración de canales (opcional)

Usa el contenedor CLI para configurar canales, luego reinicia la puerta de enlace si es necesario. WhatsApp (QR):

```bash
docker compose run --rm openclaw-cli channels login
```

Telegram (token de bot):

```bash
docker compose run --rm openclaw-cli channels add --channel telegram --token "<token>"
```

Discord (token de bot):

```bash
docker compose run --rm openclaw-cli channels add --channel discord --token "<token>"
```

Documentación: [WhatsApp](../channels/whatsapp.md), [Telegram](../channels/telegram.md), [Discord](../channels/discord.md)

### OAuth de OpenAI Codex (Docker sin interfaz gráfica)

Si eliges OAuth de OpenAI Codex en el asistente, abre una URL del navegador e intenta capturar una devolución de llamada en `http://127.0.0.1:1455/auth/callback`. En configuraciones Docker o sin interfaz gráfica, esa devolución de llamada puede mostrar un error del navegador. Copia la URL de redirección completa en la que aterrizas y pégala de nuevo en el asistente para terminar la autenticación.

### Comprobaciones de salud

Endpoints de sondeo del contenedor (sin autenticación requerida):

```bash
curl -fsS http://127.0.0.1:18789/healthz
curl -fsS http://127.0.0.1:18789/readyz
```

Alias: `/health` y `/ready`. `/healthz` es una sonda de vida superficial para "el proceso de puerta de enlace está activo". `/readyz` permanece listo durante la gracia de inicio, luego se convierte en `503` solo si los canales administrados requeridos aún están desconectados después de la gracia o se desconectan más tarde. La imagen de Docker incluye un `HEALTHCHECK` incorporado que hace ping a `/healthz` en segundo plano. En términos simples: Docker sigue comprobando si OpenClaw sigue respondiendo. Si las comprobaciones siguen fallando, Docker marca el contenedor como `unhealthy`, y los sistemas de orquestación (política de reinicio de Docker Compose, Swarm, Kubernetes, etc.) pueden reiniciarlo o reemplazarlo automáticamente. Instantánea de salud profunda autenticada (puerta de enlace + canales):

```bash
docker compose exec openclaw-gateway node dist/index.js health --token "$OPENCLAW_GATEWAY_TOKEN"
```

### Prueba de humo E2E (Docker)

```
scripts/e2e/onboard-docker.sh
```

### Prueba de humo de importación QR (Docker)

```bash
pnpm test:docker:qr
```

### LAN vs loopback (Docker Compose)

`docker-setup.sh` establece por defecto `OPENCLAW_GATEWAY_BIND=lan` para que el acceso del host a `http://127.0.0.1:18789` funcione con la publicación de puertos de Docker.

-   `lan` (predeterminado): navegador del host + CLI del host pueden alcanzar el puerto de puerta de enlace publicado.
-   `loopback`: solo los procesos dentro del espacio de nombres de red del contenedor pueden alcanzar la puerta de enlace directamente; el acceso al puerto publicado del host puede fallar.

El script de configuración también fija `gateway.mode=local` después de la configuración inicial para que los comandos CLI de Docker apunten por defecto al loopback local. Nota de configuración heredada: usa valores de modo de enlace en `gateway.bind` (`lan` / `loopback` / `custom` / `tailnet` / `auto`), no alias de host (`0.0.0.0`, `127.0.0.1`, `localhost`, `::`, `::1`). Si ves `Gateway target: ws://172.x.x.x:18789` o errores repetidos de `pairing required` de comandos CLI de Docker, ejecuta:

```bash
docker compose run --rm openclaw-cli config set gateway.mode local
docker compose run --rm openclaw-cli config set gateway.bind lan
docker compose run --rm openclaw-cli devices list --url ws://127.0.0.1:18789
```

### Notas

-   El enlace de puerta de enlace predeterminado es `lan` para uso en contenedor (`OPENCLAW_GATEWAY_BIND`).
-   El CMD de Dockerfile usa `--allow-unconfigured`; la configuración montada con `gateway.mode` no `local` aún se iniciará. Sobrescribe CMD para aplicar la guarda.
-   El contenedor de puerta de enlace es la fuente de verdad para sesiones (`~/.openclaw/agents//sessions/`).

### Modelo de almacenamiento

-   **Datos persistentes del host:** Docker Compose monta `OPENCLAW_CONFIG_DIR` en `/home/node/.openclaw` y `OPENCLAW_WORKSPACE_DIR` en `/home/node/.openclaw/workspace`, por lo que esas rutas sobreviven al reemplazo del contenedor.
-   **tmpfs efímero de sandbox:** cuando `agents.defaults.sandbox` está habilitado, los contenedores de sandbox usan `tmpfs` para `/tmp`, `/var/tmp` y `/run`. Esos montajes están separados de la pila Compose de nivel superior y desaparecen con el contenedor de sandbox.
-   **Puntos calientes de crecimiento de disco:** observa `media/`, `agents//sessions/sessions.json`, archivos JSONL de transcripción, `cron/runs/*.jsonl` y registros de archivo rotativos bajo `/tmp/openclaw/` (o tu `logging.file` configurado). Si también ejecutas la aplicación macOS fuera de Docker, sus registros de servicio están separados nuevamente: `~/.openclaw/logs/gateway.log`, `~/.openclaw/logs/gateway.err.log` y `/tmp/openclaw/openclaw-gateway.log`.

## Sandbox de agente (puerta de enlace en host + herramientas Docker)

Profundización: [Aislamiento](../gateway/sandboxing.md)

### Qué hace

Cuando `agents.defaults.sandbox` está habilitado, las **sesiones no principales** ejecutan herramientas dentro de un contenedor Docker. La puerta de enlace permanece en tu host, pero la ejecución de herramientas está aislada:

-   alcance: `"agent"` por defecto (un contenedor + espacio de trabajo por agente)
-   alcance: `"session"` para aislamiento por sesión
-   carpeta de espacio de trabajo por alcance montada en `/workspace`
-   acceso opcional al espacio de trabajo del agente (`agents.defaults.sandbox.workspaceAccess`)
-   política de herramientas permitir/denegar (denegar gana)
-   los medios entrantes se copian al espacio de trabajo de sandbox activo (`media/inbound/*`) para que las herramientas puedan leerlos (con `workspaceAccess: "rw"`, esto termina en el espacio de trabajo del agente)

Advertencia: `scope: "shared"` deshabilita el aislamiento entre sesiones. Todas las sesiones comparten un contenedor y un espacio de trabajo.

### Perfiles de sandbox por agente (multi-agente)

Si usas enrutamiento multi-agente, cada agente puede sobrescribir configuraciones de sandbox + herramientas: `agents.list[].sandbox` y `agents.list[].tools` (además de `agents.list[].tools.sandbox.tools`). Esto te permite ejecutar niveles de acceso mixtos en una puerta de enlace:

-   Acceso completo (agente personal)
-   Herramientas de solo lectura + espacio de trabajo de solo lectura (agente familiar/laboral)
-   Sin herramientas de sistema de archivos/shell (agente público)

Consulta [Sandbox y herramientas multi-agente](../tools/multi-agent-sandbox-tools.md) para ejemplos, precedencia y solución de problemas.

### Comportamiento predeterminado

-   Imagen: `openclaw-sandbox:bookworm-slim`
-   Un contenedor por agente
-   Acceso al espacio de trabajo del agente: `workspaceAccess: "none"` (predeterminado) usa `~/.openclaw/sandboxes`
    -   `"ro"` mantiene el espacio de trabajo del sandbox en `/workspace` y monta el espacio de trabajo del agente de solo lectura en `/agent` (deshabilita `write`/`edit`/`apply_patch`)
    -   `"rw"` monta el espacio de trabajo del agente de lectura/escritura en `/workspace`
-   Poda automática: inactivo > 24h O antigüedad > 7d
-   Red: `none` por defecto (optar explícitamente si necesitas salida)
    -   `host` está bloqueado.
    -   `container:` está bloqueado por defecto (riesgo de unión a espacio de nombres).
-   Permitir por defecto: `exec`, `process`, `read`, `write`, `edit`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
-   Denegar por defecto: `browser`, `canvas`, `nodes`, `cron`, `discord`, `gateway`

### Habilitar aislamiento

Si planeas instalar paquetes en `setupCommand`, nota:

-   El `docker.network` predeterminado es `"none"` (sin salida).
-   `docker.network: "host"` está bloqueado.
-   `docker.network: "container:"` está bloqueado por defecto.
-   Anulación de rompe-cristales: `agents.defaults.sandbox.docker.dangerouslyAllowContainerNamespaceJoin: true`.
-   `readOnlyRoot: true` bloquea instalaciones de paquetes.
-   `user` debe ser root para `apt-get` (omite `user` o configura `user: "0:0"`). OpenClaw vuelve a crear contenedores automáticamente cuando `setupCommand` (o configuración docker) cambia a menos que el contenedor haya sido **usado recientemente** (dentro de ~5 minutos). Los contenedores calientes registran una advertencia con el comando exacto `openclaw sandbox recreate ...`.

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared (agent es predeterminado)
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256,
          },
          seccompProfile: "/path/to/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"],
        },
        prune: {
          idleHours: 24, // 0 deshabilita la poda por inactividad
          maxAgeDays: 7, // 0 deshabilita la poda por antigüedad máxima
        },
      },
    },
  },
  tools: {
    sandbox: {
      tools: {
        allow: [
          "exec",
          "process",
          "read",
          "write",
          "edit",
          "sessions_list",
          "sessions_history",
          "sessions_send",
          "sessions_spawn",
          "session_status",
        ],
        deny: ["browser", "canvas", "nodes", "cron", "