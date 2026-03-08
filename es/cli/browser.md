

  Comandos CLI

  
# browser

Gestiona el servidor de control del navegador de OpenClaw y ejecuta acciones del navegador (pestañas, instantáneas, capturas de pantalla, navegación, clics, escritura). Relacionado:

-   Herramienta Browser + API: [Herramienta Browser](../tools/browser.md)
-   Relé de extensión de Chrome: [Extensión de Chrome](../tools/chrome-extension.md)

## Banderas comunes

-   `--url `: URL WebSocket del Gateway (por defecto desde la configuración).
-   `--token `: Token del Gateway (si es requerido).
-   `--timeout `: tiempo de espera de la solicitud (ms).
-   `--browser-profile `: elige un perfil de navegador (por defecto desde la configuración).
-   `--json`: salida legible por máquina (donde esté soportado).

## Inicio rápido (local)

```bash
openclaw browser --browser-profile chrome tabs
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

## Perfiles

Los perfiles son configuraciones de enrutamiento del navegador con nombre. En la práctica:

-   `openclaw`: lanza/se conecta a una instancia de Chrome gestionada por OpenClaw (directorio de datos de usuario aislado).
-   `chrome`: controla tu(s) pestaña(s) existente(s) de Chrome a través del relé de la extensión de Chrome.

```bash
openclaw browser profiles
openclaw browser create-profile --name work --color "#FF5A36"
openclaw browser delete-profile --name work
```

Usa un perfil específico:

```bash
openclaw browser --browser-profile work tabs
```

## Pestañas

```bash
openclaw browser tabs
openclaw browser open https://docs.openclaw.ai
openclaw browser focus <targetId>
openclaw browser close <targetId>
```

## Instantánea / captura de pantalla / acciones

Instantánea:

```bash
openclaw browser snapshot
```

Captura de pantalla:

```bash
openclaw browser screenshot
```

Navegar/hacer clic/escribir (automatización de UI basada en referencias):

```bash
openclaw browser navigate https://example.com
openclaw browser click <ref>
openclaw browser type <ref> "hello"
```

## Relé de extensión de Chrome (conectar mediante botón de barra de herramientas)

Este modo permite al agente controlar una pestaña de Chrome existente que conectas manualmente (no se conecta automáticamente). Instala la extensión sin empaquetar en una ruta estable:

```bash
openclaw browser extension install
openclaw browser extension path
```

Luego en Chrome → `chrome://extensions` → activar "Modo desarrollador" → "Cargar extensión sin empaquetar" → seleccionar la carpeta impresa. Guía completa: [Extensión de Chrome](../tools/chrome-extension.md)

## Control remoto del navegador (proxy de nodo host)

Si el Gateway se ejecuta en una máquina diferente al navegador, ejecuta un **nodo host** en la máquina que tiene Chrome/Brave/Edge/Chromium. El Gateway enviará por proxy las acciones del navegador a ese nodo (no se requiere un servidor de control del navegador separado). Usa `gateway.nodes.browser.mode` para controlar el enrutamiento automático y `gateway.nodes.browser.node` para fijar un nodo específico si hay varios conectados. Seguridad y configuración remota: [Herramienta Browser](../tools/browser.md), [Acceso remoto](../gateway/remote.md), [Tailscale](../gateway/tailscale.md), [Seguridad](../gateway/security.md)

[approvals](./approvals.md)[channels](./channels.md)

---