

  Navegador

  
# Extensión de Chrome

La extensión OpenClaw para Chrome permite que el agente controle tus **pestañas existentes de Chrome** (tu ventana normal de Chrome) en lugar de lanzar un perfil de Chrome administrado por openclaw por separado. La conexión/desconexión se realiza mediante un **solo botón en la barra de herramientas de Chrome**.

## Qué es (concepto)

Hay tres partes:

-   **Servicio de control del navegador** (Gateway o nodo): la API que llama el agente/herramienta (a través del Gateway)
-   **Servidor de relé local** (CDP de bucle local): puente entre el servidor de control y la extensión (`http://127.0.0.1:18792` por defecto)
-   **Extensión MV3 de Chrome**: se conecta a la pestaña activa usando `chrome.debugger` y canaliza los mensajes CDP al relé

OpenClaw luego controla la pestaña conectada a través de la superficie normal de la herramienta `browser` (seleccionando el perfil correcto).

## Instalar / cargar (desempaquetada)

1.  Instala la extensión en una ruta local estable:

```bash
openclaw browser extension install
```

2.  Imprime la ruta del directorio de la extensión instalada:

```bash
openclaw browser extension path
```

3.  Chrome → `chrome://extensions`

-   Activa el "Modo de desarrollador"
-   "Cargar extensión desempaquetada" → selecciona el directorio impreso arriba

4.  Fija la extensión.

## Actualizaciones (sin paso de compilación)

La extensión se incluye dentro del lanzamiento de OpenClaw (paquete npm) como archivos estáticos. No hay un paso de "compilación" separado. Después de actualizar OpenClaw:

-   Vuelve a ejecutar `openclaw browser extension install` para actualizar los archivos instalados en tu directorio de estado de OpenClaw.
-   Chrome → `chrome://extensions` → haz clic en "Recargar" en la extensión.

## Usarla (configurar el token del gateway una vez)

OpenClaw incluye un perfil de navegador integrado llamado `chrome` que apunta al relé de la extensión en el puerto por defecto. Antes de la primera conexión, abre las Opciones de la extensión y configura:

-   `Puerto` (por defecto `18792`)
-   `Token del Gateway` (debe coincidir con `gateway.auth.token` / `OPENCLAW_GATEWAY_TOKEN`)

Úsalo:

-   CLI: `openclaw browser --browser-profile chrome tabs`
-   Herramienta del agente: `browser` con `profile="chrome"`

Si quieres un nombre diferente o un puerto de relé diferente, crea tu propio perfil:

```bash
openclaw browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

### Puertos personalizados del Gateway

Si estás usando un puerto personalizado para el gateway, el puerto del relé de la extensión se deriva automáticamente: **Puerto del Relé de la Extensión = Puerto del Gateway + 3** Ejemplo: si `gateway.port: 19001`, entonces:

-   Puerto del relé de la extensión: `19004` (gateway + 3)

Configura la extensión para usar el puerto de relé derivado en la página de Opciones de la extensión.

## Conectar / desconectar (botón de la barra de herramientas)

-   Abre la pestaña que quieres que OpenClaw controle.
-   Haz clic en el icono de la extensión.
    -   La insignia muestra `ON` cuando está conectada.
-   Haz clic de nuevo para desconectar.

## ¿Qué pestaña controla?

-   **No** controla automáticamente "cualquier pestaña que estés viendo".
-   Controla **solo la(s) pestaña(s) que conectaste explícitamente** haciendo clic en el botón de la barra de herramientas.
-   Para cambiar: abre la otra pestaña y haz clic en el icono de la extensión allí.

## Insignia + errores comunes

-   `ON`: conectada; OpenClaw puede controlar esa pestaña.
-   `…`: conectando al relé local.
-   `!`: relé no accesible/no autenticado (lo más común: servidor de relé no en ejecución, o token del gateway faltante/incorrecto).

Si ves `!`:

-   Asegúrate de que el Gateway se esté ejecutando localmente (configuración por defecto), o ejecuta un host de nodo en esta máquina si el Gateway se ejecuta en otro lugar.
-   Abre la página de Opciones de la extensión; valida la accesibilidad del relé + autenticación del token del gateway.

## Gateway remoto (usar un host de nodo)

### Gateway local (misma máquina que Chrome) — usualmente sin pasos extra

Si el Gateway se ejecuta en la misma máquina que Chrome, inicia el servicio de control del navegador en el bucle local y arranca automáticamente el servidor de relé. La extensión se comunica con el relé local; las llamadas CLI/herramienta van al Gateway.

### Gateway remoto (Gateway se ejecuta en otro lugar) — ejecutar un host de nodo

Si tu Gateway se ejecuta en otra máquina, inicia un host de nodo en la máquina que ejecuta Chrome. El Gateway enviará por proxy las acciones del navegador a ese nodo; la extensión + el relé permanecen locales a la máquina del navegador. Si hay múltiples nodos conectados, fija uno con `gateway.nodes.browser.node` o configura `gateway.nodes.browser.mode`.

## Sandboxing (contenedores de herramientas)

Si tu sesión de agente está en un sandbox (`agents.defaults.sandbox.mode != "off"`), la herramienta `browser` puede estar restringida:

-   Por defecto, las sesiones en sandbox a menudo apuntan al **navegador sandbox** (`target="sandbox"`), no a tu Chrome del host.
-   La toma de control del relé de la extensión de Chrome requiere controlar el servidor de control del navegador del **host**.

Opciones:

-   Más fácil: usa la extensión desde una sesión/agente **sin sandbox**.
-   O permite el control del navegador del host para sesiones en sandbox:

```json
{
  agents: {
    defaults: {
      sandbox: {
        browser: {
          allowHostControl: true,
        },
      },
    },
  },
}
```

Luego asegúrate de que la herramienta no sea denegada por la política de herramientas, y (si es necesario) llama a `browser` con `target="host"`. Depuración: `openclaw sandbox explain`

## Consejos para acceso remoto

-   Mantén el Gateway y el host de nodo en la misma tailnet; evita exponer los puertos del relé a la LAN o a Internet público.
-   Empareja nodos intencionalmente; desactiva el enrutamiento por proxy del navegador si no quieres control remoto (`gateway.nodes.browser.mode="off"`).

## Cómo funciona la "ruta de la extensión"

`openclaw browser extension path` imprime el directorio **instalado** en disco que contiene los archivos de la extensión. La CLI intencionalmente **no** imprime una ruta de `node_modules`. Siempre ejecuta `openclaw browser extension install` primero para copiar la extensión a una ubicación estable bajo tu directorio de estado de OpenClaw. Si mueves o eliminas ese directorio de instalación, Chrome marcará la extensión como rota hasta que la recargues desde una ruta válida.

## Implicaciones de seguridad (lee esto)

Esto es poderoso y riesgoso. Trátalo como darle al modelo "manos en tu navegador".

-   La extensión usa la API de depuración de Chrome (`chrome.debugger`). Cuando está conectada, el modelo puede:
    -   hacer clic/escribir/navegar en esa pestaña
    -   leer el contenido de la página
    -   acceder a lo que la sesión iniciada de la pestaña pueda acceder
-   **Esto no está aislado** como el perfil dedicado administrado por openclaw.
    -   Si te conectas a tu perfil/pestaña de uso diario, estás otorgando acceso a ese estado de cuenta.

Recomendaciones:

-   Prefiere un perfil de Chrome dedicado (separado de tu navegación personal) para el uso del relé de la extensión.
-   Mantén el Gateway y cualquier host de nodo solo en la tailnet; confía en la autenticación del Gateway + emparejamiento de nodos.
-   Evita exponer los puertos del relé a través de la LAN (`0.0.0.0`) y evita Funnel (público).
-   El relé bloquea orígenes que no son de la extensión y requiere autenticación con token del gateway tanto para `/cdp` como para `/extension`.

Relacionado:

-   Resumen de la herramienta del navegador: [Navegador](./browser.md)
-   Auditoría de seguridad: [Seguridad](../gateway/security.md)
-   Configuración de Tailscale: [Tailscale](../gateway/tailscale.md)

[Inicio de sesión en el navegador](./browser-login.md)[Solución de problemas del navegador](./browser-linux-troubleshooting.md)