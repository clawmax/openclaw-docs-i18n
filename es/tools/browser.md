

  Navegador

  
# Navegador (administrado por OpenClaw)

OpenClaw puede ejecutar un **perfil dedicado de Chrome/Brave/Edge/Chromium** que el agente controla. Está aislado de tu navegador personal y se administra a través de un pequeño servicio de control local dentro del Gateway (solo loopback). Vista para principiantes:

-   Piensa en él como un **navegador separado, solo para el agente**.
-   El perfil `openclaw` **no** toca tu perfil de navegador personal.
-   El agente puede **abrir pestañas, leer páginas, hacer clic y escribir** en un carril seguro.
-   El perfil predeterminado `chrome` utiliza el **navegador Chromium predeterminado del sistema** a través del relé de extensión; cambia a `openclaw` para el navegador administrado aislado.

## Lo que obtienes

-   Un perfil de navegador separado llamado **openclaw** (acento naranja por defecto).
-   Control determinista de pestañas (listar/abrir/enfocar/cerrar).
-   Acciones del agente (clic/escribir/arrastrar/seleccionar), instantáneas, capturas de pantalla, PDFs.
-   Soporte opcional para múltiples perfiles (`openclaw`, `work`, `remote`, …).

Este navegador **no** es tu navegador de uso diario. Es una superficie segura y aislada para la automatización y verificación del agente.

## Inicio rápido

```bash
openclaw browser --browser-profile openclaw status
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

Si obtienes "Browser disabled", habilítalo en la configuración (ver abajo) y reinicia el Gateway.

## Perfiles: openclaw vs chrome

-   `openclaw`: navegador administrado, aislado (no requiere extensión).
-   `chrome`: relé de extensión a tu **navegador del sistema** (requiere que la extensión OpenClaw esté adjunta a una pestaña).

Establece `browser.defaultProfile: "openclaw"` si quieres el modo administrado por defecto.

## Configuración

Los ajustes del navegador residen en `~/.openclaw/openclaw.json`.

```json
{
  browser: {
    enabled: true, // predeterminado: true
    ssrfPolicy: {
      dangerouslyAllowPrivateNetwork: true, // modo red-confiable predeterminado
      // allowPrivateNetwork: true, // alias heredado
      // hostnameAllowlist: ["*.example.com", "example.com"],
      // allowedHostnames: ["localhost"],
    },
    // cdpUrl: "http://127.0.0.1:18792", // anulación heredada de perfil único
    remoteCdpTimeoutMs: 1500, // tiempo de espera HTTP para CDP remoto (ms)
    remoteCdpHandshakeTimeoutMs: 3000, // tiempo de espera de handshake WebSocket para CDP remoto (ms)
    defaultProfile: "chrome",
    color: "#FF4500",
    headless: false,
    noSandbox: false,
    attachOnly: false,
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" },
    },
  },
}
```

Notas:

-   El servicio de control del navegador se vincula a loopback en un puerto derivado de `gateway.port` (predeterminado: `18791`, que es gateway + 2). El relé usa el siguiente puerto (`18792`).
-   Si anulas el puerto del Gateway (`gateway.port` o `OPENCLAW_GATEWAY_PORT`), los puertos derivados del navegador cambian para permanecer en la misma "familia".
-   `cdpUrl` por defecto es el puerto del relé cuando no está establecido.
-   `remoteCdpTimeoutMs` se aplica a las comprobaciones de accesibilidad de CDP remoto (no loopback).
-   `remoteCdpHandshakeTimeoutMs` se aplica a las comprobaciones de accesibilidad de WebSocket de CDP remoto.
-   La navegación del navegador/apertura de pestaña está protegida contra SSRF antes de la navegación y se vuelve a verificar con el mejor esfuerzo en la URL `http(s)` final después de la navegación.
-   `browser.ssrfPolicy.dangerouslyAllowPrivateNetwork` por defecto es `true` (modelo de red confiable). Establécelo en `false` para una navegación estrictamente pública.
-   `browser.ssrfPolicy.allowPrivateNetwork` sigue siendo compatible como un alias heredado.
-   `attachOnly: true` significa "nunca iniciar un navegador local; solo adjuntar si ya está en ejecución".
-   `color` + `color` por perfil tiñen la interfaz de usuario del navegador para que puedas ver qué perfil está activo.
-   El perfil predeterminado es `openclaw` (navegador independiente administrado por OpenClaw). Usa `defaultProfile: "chrome"` para optar por el relé de extensión de Chrome.
-   Orden de auto-detección: navegador predeterminado del sistema si es basado en Chromium; de lo contrario Chrome → Brave → Edge → Chromium → Chrome Canary.
-   Los perfiles locales `openclaw` asignan automáticamente `cdpPort`/`cdpUrl` — establece esos solo para CDP remoto.

## Usar Brave (u otro navegador basado en Chromium)

Si tu navegador **predeterminado del sistema** está basado en Chromium (Chrome/Brave/Edge/etc), OpenClaw lo usa automáticamente. Establece `browser.executablePath` para anular la auto-detección: ejemplo CLI:

```bash
openclaw config set browser.executablePath "/usr/bin/google-chrome"
```

```
// macOS
{
  browser: {
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser"
  }
}

// Windows
{
  browser: {
    executablePath: "C:\\Program Files\\BraveSoftware\\Brave-Browser\\Application\\brave.exe"
  }
}

// Linux
{
  browser: {
    executablePath: "/usr/bin/brave-browser"
  }
}
```

## Control local vs remoto

-   **Control local (predeterminado):** el Gateway inicia el servicio de control loopback y puede lanzar un navegador local.
-   **Control remoto (nodo host):** ejecuta un nodo host en la máquina que tiene el navegador; el Gateway redirige las acciones del navegador a él.
-   **CDP remoto:** establece `browser.profiles..cdpUrl` (o `browser.cdpUrl`) para adjuntar a un navegador remoto basado en Chromium. En este caso, OpenClaw no lanzará un navegador local.

Las URLs de CDP remoto pueden incluir autenticación:

-   Tokens de consulta (ej., `https://provider.example?token=`)
-   Autenticación HTTP Basic (ej., `https://user:pass@provider.example`)

OpenClaw preserva la autenticación al llamar a los endpoints `/json/*` y al conectarse al WebSocket de CDP. Prefiere variables de entorno o gestores de secretos para los tokens en lugar de comprometerlos en archivos de configuración.

## Proxy de navegador de nodo (predeterminado sin configuración)

Si ejecutas un **nodo host** en la máquina que tiene tu navegador, OpenClaw puede redirigir automáticamente las llamadas de herramientas del navegador a ese nodo sin ninguna configuración adicional del navegador. Esta es la ruta predeterminada para gateways remotos. Notas:

-   El nodo host expone su servidor de control de navegador local a través de un **comando proxy**.
-   Los perfiles provienen de la propia configuración `browser.profiles` del nodo (igual que local).
-   Deshabilítalo si no lo quieres:
    -   En el nodo: `nodeHost.browserProxy.enabled=false`
    -   En el gateway: `gateway.nodes.browser.mode="off"`

## Browserless (CDP remoto alojado)

[Browserless](https://browserless.io) es un servicio alojado de Chromium que expone endpoints de CDP a través de HTTPS. Puedes apuntar un perfil de navegador OpenClaw a un endpoint de región de Browserless y autenticarte con tu clave API. Ejemplo:

```json
{
  browser: {
    enabled: true,
    defaultProfile: "browserless",
    remoteCdpTimeoutMs: 2000,
    remoteCdpHandshakeTimeoutMs: 4000,
    profiles: {
      browserless: {
        cdpUrl: "https://production-sfo.browserless.io?token=<BROWSERLESS_API_KEY>",
        color: "#00AA00",
      },
    },
  },
}
```

Notas:

-   Reemplaza `<BROWSERLESS_API_KEY>` con tu token real de Browserless.
-   Elige el endpoint de región que coincida con tu cuenta de Browserless (consulta su documentación).

## Seguridad

Ideas clave:

-   El control del navegador es solo loopback; el acceso fluye a través de la autenticación del Gateway o el emparejamiento de nodos.
-   Si el control del navegador está habilitado y no hay autenticación configurada, OpenClaw genera automáticamente `gateway.auth.token` al inicio y lo persiste en la configuración.
-   Mantén el Gateway y cualquier nodo host en una red privada (Tailscale); evita la exposición pública.
-   Trata las URLs/tokens de CDP remoto como secretos; prefiere variables de entorno o un gestor de secretos.

Consejos para CDP remoto:

-   Prefiere endpoints HTTPS y tokens de corta duración cuando sea posible.
-   Evita incrustar tokens de larga duración directamente en archivos de configuración.

## Perfiles (multi-navegador)

OpenClaw admite múltiples perfiles con nombre (configuraciones de enrutamiento). Los perfiles pueden ser:

-   **Administrado por openclaw**: una instancia de navegador basada en Chromium dedicada con su propio directorio de datos de usuario + puerto CDP
-   **Remoto**: una URL CDP explícita (navegador basado en Chromium ejecutándose en otro lugar)
-   **Relé de extensión**: tus pestañas de Chrome existentes a través del relé local + extensión de Chrome

Predeterminados:

-   El perfil `openclaw` se crea automáticamente si falta.
-   El perfil `chrome` está integrado para el relé de extensión de Chrome (apunta a `http://127.0.0.1:18792` por defecto).
-   Los puertos CDP locales se asignan desde **18800–18899** por defecto.
-   Eliminar un perfil mueve su directorio de datos local a la Papelera.

Todos los endpoints de control aceptan `?profile=`; la CLI usa `--browser-profile`.

## Relé de extensión de Chrome (usa tu Chrome existente)

OpenClaw también puede controlar **tus pestañas de Chrome existentes** (sin una instancia de Chrome "openclaw" separada) a través de un relé CDP local + una extensión de Chrome. Guía completa: [Extensión de Chrome](./chrome-extension.md) Flujo:

-   El Gateway se ejecuta localmente (misma máquina) o un nodo host se ejecuta en la máquina del navegador.
-   Un **servidor de relé** local escucha en una `cdpUrl` de loopback (predeterminado: `http://127.0.0.1:18792`).
-   Haces clic en el icono de la extensión **OpenClaw Browser Relay** en una pestaña para adjuntarla (no se adjunta automáticamente).
-   El agente controla esa pestaña a través de la herramienta normal `browser`, seleccionando el perfil correcto.

Si el Gateway se ejecuta en otro lugar, ejecuta un nodo host en la máquina del navegador para que el Gateway pueda redirigir las acciones del navegador.

### Sesiones en sandbox

Si la sesión del agente está en sandbox, la herramienta `browser` puede usar por defecto `target="sandbox"` (navegador sandbox). La toma de control del relé de extensión de Chrome requiere control del navegador host, así que:

-   ejecuta la sesión sin sandbox, o
-   establece `agents.defaults.sandbox.browser.allowHostControl: true` y usa `target="host"` al llamar a la herramienta.

### Configuración

1.  Carga la extensión (dev/unpacked):

```bash
openclaw browser extension install
```

-   Chrome → `chrome://extensions` → habilita "Modo desarrollador"
-   "Cargar extensión descomprimida" → selecciona el directorio impreso por `openclaw browser extension path`
-   Fija la extensión, luego haz clic en ella en la pestaña que quieres controlar (la insignia muestra `ON`).

2.  Úsala:

-   CLI: `openclaw browser --browser-profile chrome tabs`
-   Herramienta del agente: `browser` con `profile="chrome"`

Opcional: si quieres un nombre o puerto de relé diferente, crea tu propio perfil:

```bash
openclaw browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

Notas:

-   Este modo depende de Playwright-on-CDP para la mayoría de las operaciones (capturas de pantalla/instantáneas/acciones).
-   Desadjunta haciendo clic en el icono de la extensión nuevamente.

## Garantías de aislamiento

-   **Directorio de datos de usuario dedicado**: nunca toca tu perfil de navegador personal.
-   **Puertos dedicados**: evita `9222` para prevenir colisiones con flujos de trabajo de desarrollo.
-   **Control determinista de pestañas**: apunta a pestañas por `targetId`, no por "última pestaña".

## Selección del navegador

Al lanzar localmente, OpenClaw elige el primero disponible:

1.  Chrome
2.  Brave
3.  Edge
4.  Chromium
5.  Chrome Canary

Puedes anularlo con `browser.executablePath`. Plataformas:

-   macOS: verifica `/Applications` y `~/Applications`.
-   Linux: busca `google-chrome`, `brave`, `microsoft-edge`, `chromium`, etc.
-   Windows: verifica ubicaciones de instalación comunes.

## API de control (opcional)

Solo para integraciones locales, el Gateway expone una pequeña API HTTP loopback:

-   Estado/inicio/detención: `GET /`, `POST /start`, `POST /stop`
-   Pestañas: `GET /tabs`, `POST /tabs/open`, `POST /tabs/focus`, `DELETE /tabs/:targetId`
-   Instantánea/captura de pantalla: `GET /snapshot`, `POST /screenshot`
-   Acciones: `POST /navigate`, `POST /act`
-   Hooks: `POST /hooks/file-chooser`, `POST /hooks/dialog`
-   Descargas: `POST /download`, `POST /wait/download`
-   Depuración: `GET /console`, `POST /pdf`
-   Depuración: `GET /errors`, `GET /requests`, `POST /trace/start`, `POST /trace/stop`, `POST /highlight`
-   Red: `POST /response/body`
-   Estado: `GET /cookies`, `POST /cookies/set`, `POST /cookies/clear`
-   Estado: `GET /storage/:kind`, `POST /storage/:kind/set`, `POST /storage/:kind/clear`
-   Configuraciones: `POST /set/offline`, `POST /set/headers`, `POST /set/credentials`, `POST /set/geolocation`, `POST /set/media`, `POST /set/timezone`, `POST /set/locale`, `POST /set/device`

Todos los endpoints aceptan `?profile=`. Si la autenticación del gateway está configurada, las rutas HTTP del navegador también requieren autenticación:

-   `Authorization: Bearer `
-   `x-openclaw-password: ` o autenticación HTTP Basic con esa contraseña

### Requisito de Playwright

Algunas características (navegar/act/instantánea IA/instantánea de rol, capturas de pantalla de elementos, PDF) requieren Playwright. Si Playwright no está instalado, esos endpoints devuelven un error 501 claro. Las instantáneas ARIA y las capturas de pantalla básicas aún funcionan para Chrome administrado por openclaw. Para el controlador de relé de extensión de Chrome, las instantáneas ARIA y las capturas de pantalla requieren Playwright. Si ves `Playwright is not available in this gateway build`, instala el paquete completo de Playwright (no `playwright-core`) y reinicia el gateway, o reinstala OpenClaw con soporte de navegador.

#### Instalación de Playwright en Docker

Si tu Gateway se ejecuta en Docker, evita `npx playwright` (conflictos de anulación de npm). Usa la CLI incluida en su lugar:

```bash
docker compose run --rm openclaw-cli \
  node /app/node_modules/playwright-core/cli.js install chromium
```

Para persistir las descargas del navegador, establece `PLAYWRIGHT_BROWSERS_PATH` (por ejemplo, `/home/node/.cache/ms-playwright`) y asegúrate de que `/home/node` esté persistido a través de `OPENCLAW_HOME_VOLUME` o un bind mount. Ver [Docker](../install/docker.md).

## Cómo funciona (interno)

Flujo de alto nivel:

-   Un pequeño **servidor de control** acepta solicitudes HTTP.
-   Se conecta a navegadores basados en Chromium (Chrome/Brave/Edge/Chromium) a través de **CDP**.
-   Para acciones avanzadas (clic/escribir/instantánea/PDF), usa **Playwright** sobre CDP.
-   Cuando falta Playwright, solo están disponibles las operaciones que no son de Playwright.

Este diseño mantiene al agente en una interfaz estable y determinista mientras te permite intercambiar navegadores y perfiles locales/remotos.

## Referencia rápida de CLI

Todos los comandos aceptan `--browser-profile ` para apuntar a un perfil específico. Todos los comandos también aceptan `--json` para salida legible por máquina (cargas útiles estables). Básicos:

-   `openclaw browser status`
-   `openclaw browser start`
-   `openclaw browser stop`
-   `openclaw browser tabs`
-   `openclaw browser tab`
-   `openclaw browser tab new`
-   `openclaw browser tab select 2`
-   `openclaw browser tab close 2`
-   `openclaw browser open https://example.com`
-   `openclaw browser focus abcd1234`
-   `openclaw browser close abcd1234`

Inspección:

-   `openclaw browser screenshot`
-   `openclaw browser screenshot --full-page`
-   `openclaw browser screenshot --ref 12`
-   `openclaw browser screenshot --ref e12`
-   `openclaw browser snapshot`
-   `openclaw browser snapshot --format aria --limit 200`
-   `openclaw browser snapshot --interactive --compact --depth 6`
-   `openclaw browser snapshot --efficient`
-   `openclaw browser snapshot --labels`
-   `openclaw browser snapshot --selector "#main" --interactive`
-   `openclaw browser snapshot --frame "iframe#main" --interactive`
-   `openclaw browser console --level error`
-   `openclaw browser errors --clear`
-   `openclaw browser requests --filter api --clear`
-   `openclaw browser pdf`
-   `openclaw browser responsebody "**/api" --max-chars 5000`

Acciones:

-   `openclaw browser navigate https://example.com`
-   `openclaw browser resize 1280 720`
-   `openclaw browser click 12 --double`
-   `openclaw browser click e12 --double`
-   `openclaw browser type 23 "hello" --submit`
-   `openclaw browser press Enter`
-   `openclaw browser hover 44`
-   `openclaw browser scrollintoview e12`
-   `openclaw browser drag 10 11`
-   `openclaw browser select 9 OptionA OptionB`
-   `openclaw browser download e12 report.pdf`
-   `openclaw browser waitfordownload report.pdf`
-   `openclaw browser upload /tmp/openclaw/uploads/file.pdf`
-   `openclaw browser fill --fields '[{"ref":"1","type":"text","value":"Ada"}]'`
-   `openclaw browser dialog --accept`
-   `openclaw browser wait --text "Done"`
-   `openclaw browser wait "#main" --url "**/dash" --load networkidle --fn "window.ready===true"`
-   `openclaw browser evaluate --fn '(el) => el.textContent' --ref 7`
-   `openclaw browser highlight e12`
-   `openclaw browser trace start`
-   `openclaw browser trace stop`

Estado:

-   `openclaw browser cookies`
-   `openclaw browser cookies set session abc123 --url "https://example.com"`
-   `openclaw browser cookies clear`
-   `openclaw browser storage local get`
-   `openclaw browser storage local set theme dark`
-   `openclaw browser storage session clear`
-   `openclaw browser set offline on`
-   `openclaw browser set headers --headers-json '{"X-Debug":"1"}'`
-   `openclaw browser set credentials user pass`
-   `openclaw browser set credentials --clear`
-   `openclaw browser set geo 37.7749 -122.4194 --origin "https://example.com"`
-   `openclaw browser set geo --clear`
-   `openclaw browser set media dark`
-   `openclaw browser set timezone America/New_York`
-   `openclaw browser set locale en-US`
-   `openclaw browser set device "iPhone 14"`

Notas:

-   `upload` y `dialog` son llamadas de **armado**; ejecútalas antes del clic/presión que activa el selector/cuadro de diálogo.
-   Las rutas de salida de descarga y traza están restringidas a las raíces temporales de OpenClaw:
    -   trazas: `/tmp/openclaw` (alternativa: `${os.tmpdir()}/openclaw`)
    -   descargas: `/tmp/openclaw/downloads` (alternativa: `${os.tmpdir()}/openclaw/downloads`)
-   Las rutas de carga están restringidas a una raíz de cargas temporal de OpenClaw:
    -   cargas: `/tmp/openclaw/uploads` (alternativa: `${os.tmpdir()}/openclaw/uploads`)
-   `upload` también puede establecer entradas de archivo directamente a través de `--input-ref` o `--element`.
-   `snapshot`:
    -   `--format ai` (predeterminado cuando Playwright está instalado): devuelve una instantánea IA con referencias numéricas (`aria-ref=""`).
    -   `--format aria`: devuelve el árbol de accesibilidad (sin referencias; solo inspección).
    -   `--efficient` (o `--mode efficient`): preajuste de instantánea de rol compacta (interactivo + compacto + profundidad + maxChars más bajo).
    -   Predeterminado de configuración (solo herramienta/CLI): establece `browser.snapshotDefaults.mode: "efficient"` para usar instantáneas eficientes cuando el llamador no pasa un modo (ver [Configuración del Gateway](../gateway/configuration.md#browser-openclaw-managed-browser)).
    -   Las opciones de instantánea de rol (`--interactive`, `--compact`, `--depth`, `--selector`) fuerzan una instantánea basada en rol con referencias como `ref=e12`.
    -   `--frame ""` limita las instantáneas de rol a un iframe (se empareja con referencias de rol como `e12`).
    -   `--interactive` produce una lista plana y fácil de seleccionar de elementos interactivos (mejor para acciones de control).
    -   `--labels` agrega una captura de pantalla solo de la ventana gráfica con etiquetas de referencia superpuestas (imprime `MEDIA:`).
-   `click`/`type`/etc requieren una `ref` de `snapshot` (ya sea numérica `12` o referencia de rol `e12`). Los selectores CSS no están soportados intencionalmente para acciones.

## Instantáneas y referencias

OpenClaw admite dos estilos de "instantánea":

-   **Instantánea IA (referencias numéricas):** `openclaw browser snapshot` (predeterminado; `--format ai`)
    -   Salida: una instantánea de texto que incluye referencias numéricas.
    -   Acciones: `openclaw browser click 12`, `openclaw browser type 23 "hello"`.
    -   Internamente, la referencia se resuelve a través de `aria-ref` de Playwright.
-   **Instantánea de rol (referencias de rol como `e12`):** `openclaw browser snapshot --interactive` (o `--compact`, `--depth`, `--selector`, `--frame`)
    -   Salida: una lista/árbol basado en rol con `[ref=e12]` (y opcionalmente `[nth=1]`).
    -   Acciones: `openclaw browser click e12`, `openclaw browser highlight e12`.
    -   Internamente, la referencia se resuelve a través de `getByRole(...)` (más `nth()` para duplicados).
    -   Agrega `--labels` para incluir una captura de pantalla de la ventana gráfica con etiquetas `e12` superpuestas.

Comportamiento de las referencias:

-   Las referencias **no son estables entre navegaciones**; si algo falla, vuelve a ejecutar `snapshot` y usa una referencia nueva.
-   Si la instantánea de rol se tomó con `--frame`, las referencias de rol están limitadas a ese iframe hasta la siguiente instantánea de rol.

## Potenciadores de espera

Puedes esperar más que solo tiempo/texto:

-   Esperar por URL (globs soportados por Playwright):
    -   `openclaw browser wait --url "**/dash"`
-   Esperar por estado de carga:
    -   `openclaw browser wait --load networkidle`
-   Esperar por un predicado JS:
    -   `openclaw browser wait --fn "window.ready===true"`
-   Esperar por un selector que se vuelva visible:
    -   `openclaw browser wait "#main"`

Estos se pueden combinar:

```bash
openclaw browser wait "#main" \
  --url "**/dash" \
  --load networkidle \
  --fn "window.ready===true" \
  --timeout-ms 15000
```

## Flujos de trabajo de depuración

Cuando una acción falla (ej. "no visible", "violación de modo estricto", "cubierto"):

1.  `openclaw browser snapshot --interactive`
2.  Usa `click ` / `type ` (prefiere referencias de rol en modo interactivo)
3.  Si aún falla: `openclaw browser highlight ` para ver qué está apuntando Playwright
4.  Si la página se comporta de manera extraña:
    -   `openclaw browser errors --clear`
    -   `openclaw browser requests --filter api --clear`
5.  Para depuración profunda: graba una traza:
    -   `openclaw browser trace start`
    -   reproduce el problema
    -   `openclaw browser trace stop` (imprime `TRACE:`)

## Salida JSON

`--json` es para scripting y herramientas estructuradas. Ejemplos:

```bash
openclaw browser status --json
openclaw browser snapshot --interactive --json
openclaw browser requests --filter api --json
openclaw browser cookies --json
```

Las instantáneas de rol en JSON incluyen `refs` más un pequeño bloque `stats` (líneas/caracteres/refs/interactivo) para que las herramientas puedan razonar sobre el tamaño y densidad de la carga útil.

## Perillas de estado y entorno

Estas son útiles para flujos de trabajo de "hacer que el sitio se comporte como X":

-   Cookies: `cookies`, `cookies set`, `cookies clear`
-   Almacenamiento: `storage local|session get|set|clear`
-   Sin conexión: `set offline on|off`
-   Encabezados: `set headers --headers-json '{"X-Debug":"1"}'` (el heredado `set headers --json '{"X-Debug":"1"}'` sigue siendo compatible)
-   Autenticación HTTP básica: `set credentials user pass` (o `--clear`)
-   Geolocalización: `set geo   --origin "https://example.com"` (o `--clear`)
-   Medios: `set media dark|light|no-preference|none`
-   Zona horaria / localidad: `set timezone ...`, `set locale ...`
-   Dispositivo / ventana gráfica:
    -   `set device "iPhone 14"` (preajustes de dispositivo de Playwright)
    -   `set viewport 1280 720`

## Seguridad y privacidad

-   El perfil de navegador openclaw puede contener sesiones iniciadas; trátalo como sensible.
-   `browser act kind=evaluate` / `openclaw browser evaluate` y `wait --fn` ejecutan JavaScript arbitrario en el contexto de la página. La inyección de prompt puede dirigir esto. Deshabilítalo con `browser.evaluateEnabled=false` si no lo necesitas.
-   Para inicios de sesión y notas anti-bot (X/Twitter, etc.), consulta [Inicio de sesión en navegador + publicación en X/Twitter](./browser-login.md).
-   Mantén el Gateway/nodo host privado (solo loopback o tailnet).
-   Los endpoints de CDP remoto son poderosos; túnelalos y protégelos.

Ejemplo de modo estricto (bloquear destinos privados/internos por defecto):

```json
{
  browser: {
    ssrfPolicy: {
      dangerouslyAllowPrivateNetwork: false,
      hostnameAllowlist: ["*.example.com", "example.com"],
      allowedHostnames: ["localhost"], // permiso exacto opcional
    },
  },
}
```

## Solución de problemas

Para problemas específicos de Linux (especialmente Chromium snap), consulta [Solución de problemas del navegador en Linux](./browser-linux-troubleshooting.md).

## Herramientas del agente + cómo funciona el control

El agente obtiene **una herramienta** para la automatización del navegador:

-   `browser` — estado/inicio/detención/pestañas/abrir/enfocar/cerrar/instantánea/captura de pantalla/navegar/act

Cómo se mapea:

-   `browser snapshot` devuelve un árbol de UI estable (IA o ARIA).
-   `browser act` usa los IDs `ref` de la instantánea para hacer clic/escribir/arrastrar/seleccionar.
-   `browser screenshot` captura píxeles (página completa o elemento).
-   `browser` acepta:
    -   `profile` para elegir un perfil de navegador con nombre (openclaw, chrome, o CDP remoto).
    -   `target` (`sandbox` | `host` | `node`) para seleccionar dónde reside el navegador.
    -   En sesiones en sandbox, `target: "host"` requiere `agents.defaults.sandbox.browser.allowHostControl=true`.
    -   Si se omite `target`: las sesiones en sandbox usan por defecto `sandbox`, las sesiones no sandbox usan por defecto `host`.
    -   Si un nodo con capacidad de navegador está conectado, la