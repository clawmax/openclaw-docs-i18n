

  Otros métodos de instalación

  
# Podman

Ejecuta la puerta de enlace OpenClaw en un contenedor Podman **sin privilegios de root**. Utiliza la misma imagen que Docker (construida desde el [Dockerfile](https://github.com/openclaw/openclaw/blob/main/Dockerfile) del repositorio).

## Requisitos

-   Podman (sin privilegios de root)
-   Sudo para configuración única (crear usuario, construir imagen)

## Inicio rápido

**1\. Configuración única** (desde la raíz del repositorio; crea usuario, construye imagen, instala script de lanzamiento):

```
./setup-podman.sh
```

Esto también crea un `~openclaw/.openclaw/openclaw.json` mínimo (establece `gateway.mode="local"`) para que la puerta de enlace pueda iniciarse sin ejecutar el asistente. Por defecto, el contenedor **no** se instala como un servicio systemd, lo inicias manualmente (ver más abajo). Para una configuración tipo producción con inicio automático y reinicios, instálalo como un servicio de usuario systemd Quadlet:

```bash
./setup-podman.sh --quadlet
```

(O establece `OPENCLAW_PODMAN_QUADLET=1`; usa `--container` para instalar solo el contenedor y el script de lanzamiento). Variables de entorno opcionales en tiempo de construcción (establécelas antes de ejecutar `setup-podman.sh`):

-   `OPENCLAW_DOCKER_APT_PACKAGES` — instala paquetes apt adicionales durante la construcción de la imagen
-   `OPENCLAW_EXTENSIONS` — pre-instala dependencias de extensiones (nombres de extensiones separados por espacios, ej. `diagnostics-otel matrix`)

**2\. Iniciar puerta de enlace** (manual, para pruebas rápidas):

```bash
./scripts/run-openclaw-podman.sh launch
```

**3\. Asistente de incorporación** (ej. para añadir canales o proveedores):

```bash
./scripts/run-openclaw-podman.sh launch setup
```

Luego abre `http://127.0.0.1:18789/` y usa el token de `~openclaw/.openclaw/.env` (o el valor impreso por el asistente).

## Systemd (Quadlet, opcional)

Si ejecutaste `./setup-podman.sh --quadlet` (o `OPENCLAW_PODMAN_QUADLET=1`), se instala una unidad [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html) para que la puerta de enlace se ejecute como un servicio de usuario systemd para el usuario openclaw. El servicio se habilita e inicia al final de la configuración.

-   **Iniciar:** `sudo systemctl --machine openclaw@ --user start openclaw.service`
-   **Detener:** `sudo systemctl --machine openclaw@ --user stop openclaw.service`
-   **Estado:** `sudo systemctl --machine openclaw@ --user status openclaw.service`
-   **Registros:** `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`

El archivo quadlet reside en `~openclaw/.config/containers/systemd/openclaw.container`. Para cambiar puertos o variables de entorno, edita ese archivo (o el `.env` que fuente), luego `sudo systemctl --machine openclaw@ --user daemon-reload` y reinicia el servicio. Al arrancar, el servicio se inicia automáticamente si está habilitado el "lingering" para openclaw (la configuración hace esto cuando loginctl está disponible). Para añadir quadlet **después** de una configuración inicial que no lo usó, vuelve a ejecutar: `./setup-podman.sh --quadlet`.

## El usuario openclaw (sin inicio de sesión)

`setup-podman.sh` crea un usuario de sistema dedicado `openclaw`:

-   **Shell:** `nologin` — sin inicio de sesión interactivo; reduce la superficie de ataque.
-   **Home:** ej. `/home/openclaw` — contiene `~/.openclaw` (configuración, espacio de trabajo) y el script de lanzamiento `run-openclaw-podman.sh`.
-   **Podman sin root:** El usuario debe tener un rango de **subuid** y **subgid**. Muchas distribuciones los asignan automáticamente cuando se crea el usuario. Si la configuración imprime una advertencia, añade líneas a `/etc/subuid` y `/etc/subgid`:
    
    Copiar
    
    ```
    openclaw:100000:65536
    ```
    
    Luego inicia la puerta de enlace como ese usuario (ej. desde cron o systemd):
    
    Copiar
    
    ```bash
    sudo -u openclaw /home/openclaw/run-openclaw-podman.sh
    sudo -u openclaw /home/openclaw/run-openclaw-podman.sh setup
    ```
    
-   **Configuración:** Solo `openclaw` y root pueden acceder a `/home/openclaw/.openclaw`. Para editar la configuración: usa la Interfaz de Control una vez que la puerta de enlace esté en ejecución, o `sudo -u openclaw $EDITOR /home/openclaw/.openclaw/openclaw.json`.

## Entorno y configuración

-   **Token:** Almacenado en `~openclaw/.openclaw/.env` como `OPENCLAW_GATEWAY_TOKEN`. `setup-podman.sh` y `run-openclaw-podman.sh` lo generan si falta (usa `openssl`, `python3`, o `od`).
-   **Opcional:** En ese `.env` puedes establecer claves de proveedores (ej. `GROQ_API_KEY`, `OLLAMA_API_KEY`) y otras variables de entorno de OpenClaw.
-   **Puertos del host:** Por defecto, el script mapea `18789` (puerta de enlace) y `18790` (bridge). Sobrescribe el mapeo de puertos del **host** con `OPENCLAW_PODMAN_GATEWAY_HOST_PORT` y `OPENCLAW_PODMAN_BRIDGE_HOST_PORT` al lanzar.
-   **Vinculación de la puerta de enlace:** Por defecto, `run-openclaw-podman.sh` inicia la puerta de enlace con `--bind loopback` para un acceso local seguro. Para exponer en la LAN, establece `OPENCLAW_GATEWAY_BIND=lan` y configura `gateway.controlUi.allowedOrigins` (o habilita explícitamente la fallback del encabezado del host) en `openclaw.json`.
-   **Rutas:** La configuración y el espacio de trabajo del host por defecto son `~openclaw/.openclaw` y `~openclaw/.openclaw/workspace`. Sobrescribe las rutas del host utilizadas por el script de lanzamiento con `OPENCLAW_CONFIG_DIR` y `OPENCLAW_WORKSPACE_DIR`.

## Modelo de almacenamiento

-   **Datos persistentes en el host:** `OPENCLAW_CONFIG_DIR` y `OPENCLAW_WORKSPACE_DIR` se montan en el contenedor y retienen el estado en el host.
-   **Tmpfs efímero del sandbox:** si habilitas `agents.defaults.sandbox`, los contenedores sandbox de herramientas montan `tmpfs` en `/tmp`, `/var/tmp`, y `/run`. Esas rutas están respaldadas en memoria y desaparecen con el contenedor sandbox; la configuración del contenedor Podman de nivel superior no añade sus propios montajes tmpfs.
-   **Puntos calientes de crecimiento del disco:** las rutas principales a vigilar son `media/`, `agents//sessions/sessions.json`, archivos JSONL de transcripción, `cron/runs/*.jsonl`, y registros de archivos rotativos bajo `/tmp/openclaw/` (o tu `logging.file` configurado).

`setup-podman.sh` ahora almacena provisionalmente el tar de la imagen en un directorio temporal privado e imprime el directorio base elegido durante la configuración. Para ejecuciones sin root, acepta `TMPDIR` solo cuando esa base es segura de usar; de lo contrario, recurre a `/var/tmp`, luego `/tmp`. El tar guardado permanece solo para el propietario y se transmite al `podman load` del usuario objetivo, por lo que los directorios temporales privados del llamante no bloquean la configuración.

## Comandos útiles

-   **Registros:** Con quadlet: `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`. Con script: `sudo -u openclaw podman logs -f openclaw`
-   **Detener:** Con quadlet: `sudo systemctl --machine openclaw@ --user stop openclaw.service`. Con script: `sudo -u openclaw podman stop openclaw`
-   **Iniciar de nuevo:** Con quadlet: `sudo systemctl --machine openclaw@ --user start openclaw.service`. Con script: vuelve a ejecutar el script de lanzamiento o `podman start openclaw`
-   **Eliminar contenedor:** `sudo -u openclaw podman rm -f openclaw` — la configuración y el espacio de trabajo en el host se conservan

## Solución de problemas

-   **Permiso denegado (EACCES) en configuración o perfiles de autenticación:** El contenedor por defecto usa `--userns=keep-id` y se ejecuta con el mismo uid/gid que el usuario del host que ejecuta el script. Asegúrate de que tu `OPENCLAW_CONFIG_DIR` y `OPENCLAW_WORKSPACE_DIR` en el host sean propiedad de ese usuario.
-   **Inicio de puerta de enlace bloqueado (falta `gateway.mode=local`):** Asegúrate de que `~openclaw/.openclaw/openclaw.json` exista y establezca `gateway.mode="local"`. `setup-podman.sh` crea este archivo si falta.
-   **Podman sin root falla para el usuario openclaw:** Verifica que `/etc/subuid` y `/etc/subgid` contengan una línea para `openclaw` (ej. `openclaw:100000:65536`). Añádela si falta y reinicia.
-   **Nombre de contenedor en uso:** El script de lanzamiento usa `podman run --replace`, por lo que el contenedor existente se reemplaza cuando inicias de nuevo. Para limpiar manualmente: `podman rm -f openclaw`.
-   **Script no encontrado al ejecutar como openclaw:** Asegúrate de que se ejecutó `setup-podman.sh` para que `run-openclaw-podman.sh` se copie al home de openclaw (ej. `/home/openclaw/run-openclaw-podman.sh`).
-   **Servicio Quadlet no encontrado o falla al iniciar:** Ejecuta `sudo systemctl --machine openclaw@ --user daemon-reload` después de editar el archivo `.container`. Quadlet requiere cgroups v2: `podman info --format '{{.Host.CgroupsVersion}}'` debería mostrar `2`.

## Opcional: ejecutar como tu propio usuario

Para ejecutar la puerta de enlace como tu usuario normal (sin usuario openclaw dedicado): construye la imagen, crea `~/.openclaw/.env` con `OPENCLAW_GATEWAY_TOKEN`, y ejecuta el contenedor con `--userns=keep-id` y montajes a tu `~/.openclaw`. El script de lanzamiento está diseñado para el flujo de usuario openclaw; para una configuración de usuario único puedes en su lugar ejecutar el comando `podman run` del script manualmente, apuntando la configuración y el espacio de trabajo a tu home. Recomendado para la mayoría de usuarios: usa `setup-podman.sh` y ejecuta como el usuario openclaw para que la configuración y el proceso estén aislados.

[Docker](./docker.md)[Nix](./nix.md)