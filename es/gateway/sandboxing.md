

  Seguridad y sandboxing

  
# Sandboxing

OpenClaw puede ejecutar **herramientas dentro de contenedores Docker** para reducir el radio de explosión. Esto es **opcional** y se controla mediante configuración (`agents.defaults.sandbox` o `agents.list[].sandbox`). Si el sandboxing está desactivado, las herramientas se ejecutan en el host. El Gateway permanece en el host; la ejecución de herramientas se realiza en un sandbox aislado cuando está habilitado. Esto no es un límite de seguridad perfecto, pero limita materialmente el acceso al sistema de archivos y procesos cuando el modelo hace algo tonto.

## Qué se ejecuta en sandbox

-   Ejecución de herramientas (`exec`, `read`, `write`, `edit`, `apply_patch`, `process`, etc.).
-   Navegador sandbox opcional (`agents.defaults.sandbox.browser`).
    -   Por defecto, el navegador sandbox se inicia automáticamente (asegura que CDP sea accesible) cuando la herramienta del navegador lo necesita. Configúralo mediante `agents.defaults.sandbox.browser.autoStart` y `agents.defaults.sandbox.browser.autoStartTimeoutMs`.
    -   Por defecto, los contenedores del navegador sandbox usan una red Docker dedicada (`openclaw-sandbox-browser`) en lugar de la red global `bridge`. Configúralo con `agents.defaults.sandbox.browser.network`.
    -   Opcionalmente, `agents.defaults.sandbox.browser.cdpSourceRange` restringe el ingreso CDP del borde del contenedor con una lista de permitidos CIDR (por ejemplo `172.21.0.1/32`).
    -   El acceso del observador noVNC está protegido por contraseña por defecto; OpenClaw emite una URL con token de corta duración que sirve una página de arranque local y abre noVNC con la contraseña en el fragmento de la URL (no en los registros de consulta/cabecera).
    -   `agents.defaults.sandbox.browser.allowHostControl` permite que las sesiones sandboxed apunten explícitamente al navegador del host.
    -   Las listas de permitidos opcionales controlan `target: "custom"`: `allowedControlUrls`, `allowedControlHosts`, `allowedControlPorts`.

No se ejecuta en sandbox:

-   El proceso del Gateway en sí.
-   Cualquier herramienta explícitamente permitida para ejecutarse en el host (ej. `tools.elevated`).
    -   **La ejecución elevada se ejecuta en el host y omite el sandboxing.**
    -   Si el sandboxing está desactivado, `tools.elevated` no cambia la ejecución (ya está en el host). Consulta [Modo Elevado](../tools/elevated.md).

## Modos

`agents.defaults.sandbox.mode` controla **cuándo** se usa el sandboxing:

-   `"off"`: sin sandboxing.
-   `"non-main"`: sandbox solo para sesiones **no principales** (por defecto si quieres chats normales en el host).
-   `"all"`: cada sesión se ejecuta en un sandbox. Nota: `"non-main"` se basa en `session.mainKey` (por defecto `"main"`), no en el id del agente. Las sesiones de grupo/canal usan sus propias claves, por lo que cuentan como no principales y se ejecutarán en sandbox.

## Alcance

`agents.defaults.sandbox.scope` controla **cuántos contenedores** se crean:

-   `"session"` (por defecto): un contenedor por sesión.
-   `"agent"`: un contenedor por agente.
-   `"shared"`: un contenedor compartido por todas las sesiones sandboxed.

## Acceso al espacio de trabajo

`agents.defaults.sandbox.workspaceAccess` controla **qué puede ver el sandbox**:

-   `"none"` (por defecto): las herramientas ven un espacio de trabajo sandbox bajo `~/.openclaw/sandboxes`.
-   `"ro"`: monta el espacio de trabajo del agente como solo lectura en `/agent` (deshabilita `write`/`edit`/`apply_patch`).
-   `"rw"`: monta el espacio de trabajo del agente con lectura/escritura en `/workspace`.

Los medios entrantes se copian al espacio de trabajo sandbox activo (`media/inbound/*`). Nota sobre habilidades: la herramienta `read` está anclada a la raíz del sandbox. Con `workspaceAccess: "none"`, OpenClaw refleja las habilidades elegibles en el espacio de trabajo sandbox (`.../skills`) para que puedan leerse. Con `"rw"`, las habilidades del espacio de trabajo son legibles desde `/workspace/skills`.

## Montajes bind personalizados

`agents.defaults.sandbox.docker.binds` monta directorios adicionales del host en el contenedor. Formato: `host:contenedor:modo` (ej., `"/home/user/source:/source:rw"`). Los montajes globales y por agente se **fusionan** (no se reemplazan). Bajo `scope: "shared"`, los montajes por agente se ignoran. `agents.defaults.sandbox.browser.binds` monta directorios adicionales del host solo en el contenedor del **navegador sandbox**.

-   Cuando se establece (incluyendo `[]`), reemplaza `agents.defaults.sandbox.docker.binds` para el contenedor del navegador.
-   Cuando se omite, el contenedor del navegador recurre a `agents.defaults.sandbox.docker.binds` (compatible con versiones anteriores).

Ejemplo (fuente de solo lectura + un directorio de datos extra):

```json
{
  agents: {
    defaults: {
      sandbox: {
        docker: {
          binds: ["/home/user/source:/source:ro", "/var/data/myapp:/data:ro"],
        },
      },
    },
    list: [
      {
        id: "build",
        sandbox: {
          docker: {
            binds: ["/mnt/cache:/cache:rw"],
          },
        },
      },
    ],
  },
}
```

Notas de seguridad:

-   Los montajes bind omiten el sistema de archivos del sandbox: exponen rutas del host con el modo que establezcas (`:ro` o `:rw`).
-   OpenClaw bloquea fuentes de montaje peligrosas (por ejemplo: `docker.sock`, `/etc`, `/proc`, `/sys`, `/dev`, y montajes padre que los expondrían).
-   Los montajes sensibles (secretos, claves SSH, credenciales de servicio) deben ser `:ro` a menos que sea absolutamente necesario.
-   Combina con `workspaceAccess: "ro"` si solo necesitas acceso de lectura al espacio de trabajo; los modos de montaje permanecen independientes.
-   Consulta [Sandbox vs Política de Herramientas vs Elevado](./sandbox-vs-tool-policy-vs-elevated.md) para ver cómo interactúan los montajes con la política de herramientas y la ejecución elevada.

## Imágenes + configuración

Imagen por defecto: `openclaw-sandbox:bookworm-slim` Construyela una vez:

```
scripts/sandbox-setup.sh
```

Nota: la imagen por defecto **no** incluye Node. Si una habilidad necesita Node (u otros entornos de ejecución), o bien crea una imagen personalizada o instálalo mediante `sandbox.docker.setupCommand` (requiere salida de red + raíz escribible + usuario root). Si quieres una imagen de sandbox más funcional con herramientas comunes (por ejemplo `curl`, `jq`, `nodejs`, `python3`, `git`), construye:

```
scripts/sandbox-common-setup.sh
```

Luego establece `agents.defaults.sandbox.docker.image` a `openclaw-sandbox-common:bookworm-slim`. Imagen del navegador sandbox:

```
scripts/sandbox-browser-setup.sh
```

Por defecto, los contenedores sandbox se ejecutan **sin red**. Anula esto con `agents.defaults.sandbox.docker.network`. La imagen del navegador sandbox incluida también aplica valores predeterminados de inicio de Chromium conservadores para cargas de trabajo en contenedores. Los valores predeterminados actuales del contenedor incluyen:

-   `--remote-debugging-address=127.0.0.1`
-   `--remote-debugging-port=`
-   `--user-data-dir=${HOME}/.chrome`
-   `--no-first-run`
-   `--no-default-browser-check`
-   `--disable-3d-apis`
-   `--disable-gpu`
-   `--disable-dev-shm-usage`
-   `--disable-background-networking`
-   `--disable-extensions`
-   `--disable-features=TranslateUI`
-   `--disable-breakpad`
-   `--disable-crash-reporter`
-   `--disable-software-rasterizer`
-   `--no-zygote`
-   `--metrics-recording-only`
-   `--renderer-process-limit=2`
-   `--no-sandbox` y `--disable-setuid-sandbox` cuando `noSandbox` está habilitado.
-   Las tres banderas de endurecimiento de gráficos (`--disable-3d-apis`, `--disable-software-rasterizer`, `--disable-gpu`) son opcionales y son útiles cuando los contenedores carecen de soporte GPU. Establece `OPENCLAW_BROWSER_DISABLE_GRAPHICS_FLAGS=0` si tu carga de trabajo requiere WebGL u otras características 3D/del navegador.
-   `--disable-extensions` está habilitado por defecto y puede deshabilitarse con `OPENCLAW_BROWSER_DISABLE_EXTENSIONS=0` para flujos que dependen de extensiones.
-   `--renderer-process-limit=2` está controlado por `OPENCLAW_BROWSER_RENDERER_PROCESS_LIMIT=`, donde `0` mantiene el valor predeterminado de Chromium.

Si necesitas un perfil de tiempo de ejecución diferente, usa una imagen de navegador personalizada y proporciona tu propio punto de entrada. Para perfiles de Chromium locales (no contenedores), usa `browser.extraArgs` para añadir banderas de inicio adicionales. Valores predeterminados de seguridad:

-   `network: "host"` está bloqueado.
-   `network: "container:"` está bloqueado por defecto (riesgo de omisión de unión de espacio de nombres).
-   Anulación de emergencia: `agents.defaults.sandbox.docker.dangerouslyAllowContainerNamespaceJoin: true`.

Las instalaciones de Docker y el gateway en contenedor se encuentran aquí: [Docker](../install/docker.md) Para implementaciones de gateway Docker, `docker-setup.sh` puede inicializar la configuración del sandbox. Establece `OPENCLAW_SANDBOX=1` (o `true`/`yes`/`on`) para habilitar esa ruta. Puedes anular la ubicación del socket con `OPENCLAW_DOCKER_SOCKET`. Configuración completa y referencia de entorno: [Docker](../install/docker.md#enable-agent-sandbox-for-docker-gateway-opt-in).

## setupCommand (configuración única del contenedor)

`setupCommand` se ejecuta **una vez** después de crear el contenedor sandbox (no en cada ejecución). Se ejecuta dentro del contenedor mediante `sh -lc`. Rutas:

-   Global: `agents.defaults.sandbox.docker.setupCommand`
-   Por agente: `agents.list[].sandbox.docker.setupCommand`

Problemas comunes:

-   Por defecto `docker.network` es `"none"` (sin salida), por lo que las instalaciones de paquetes fallarán.
-   `docker.network: "container:"` requiere `dangerouslyAllowContainerNamespaceJoin: true` y es solo para emergencias.
-   `readOnlyRoot: true` impide escrituras; establece `readOnlyRoot: false` o crea una imagen personalizada.
-   `user` debe ser root para instalaciones de paquetes (omite `user` o establece `user: "0:0"`).
-   La ejecución en sandbox **no** hereda `process.env` del host. Usa `agents.defaults.sandbox.docker.env` (o una imagen personalizada) para claves API de habilidades.

## Política de herramientas + escapes

Las políticas de permitir/denegar herramientas aún se aplican antes de las reglas del sandbox. Si una herramienta está denegada globalmente o por agente, el sandboxing no la recupera. `tools.elevated` es un escape explícito que ejecuta `exec` en el host. Las directivas `/exec` solo se aplican para remitentes autorizados y persisten por sesión; para deshabilitar `exec` de forma estricta, usa la política de denegación de herramientas (consulta [Sandbox vs Política de Herramientas vs Elevado](./sandbox-vs-tool-policy-vs-elevated.md)). Depuración:

-   Usa `openclaw sandbox explain` para inspeccionar el modo de sandbox efectivo, la política de herramientas y las claves de configuración de solución.
-   Consulta [Sandbox vs Política de Herramientas vs Elevado](./sandbox-vs-tool-policy-vs-elevated.md) para el modelo mental de "¿por qué está bloqueado esto?". Mantenlo bloqueado.

## Anulaciones multi-agente

Cada agente puede anular sandbox + herramientas: `agents.list[].sandbox` y `agents.list[].tools` (además de `agents.list[].tools.sandbox.tools` para la política de herramientas del sandbox). Consulta [Sandbox y Herramientas Multi-Agente](../tools/multi-agent-sandbox-tools.md) para la precedencia.

## Ejemplo mínimo de habilitación

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none",
      },
    },
  },
}
```

## Documentación relacionada

-   [Configuración de Sandbox](./configuration.md#agentsdefaults-sandbox)
-   [Sandbox y Herramientas Multi-Agente](../tools/multi-agent-sandbox-tools.md)
-   [Seguridad](./security.md)

[Seguridad](./security.md)[Sandbox vs Política de Herramientas vs Elevado](./sandbox-vs-tool-policy-vs-elevated.md)

---