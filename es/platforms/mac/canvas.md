title: "Documentación y Guía de Uso de la Aplicación OpenClaw Canvas para macOS"
description: "Aprende a usar el panel Canvas de OpenClaw en macOS para espacios de trabajo HTML/CSS/JS y A2UI. La guía cubre almacenamiento de archivos, API del agente, comandos CLI y seguridad."
keywords: ["openclaw canvas", "panel canvas macos", "integración a2ui", "api del agente", "wkwebview", "espacio de trabajo canvas", "documentación de openclaw", "comandos cli canvas"]
---

  Aplicación complementaria para macOS

  
# Canvas

La aplicación de macOS incorpora un **panel Canvas** controlado por el agente usando `WKWebView`. Es un espacio de trabajo visual ligero para HTML/CSS/JS, A2UI y pequeñas superficies de UI interactivas.

## Dónde reside Canvas

El estado de Canvas se almacena en Application Support:

-   `~/Library/Application Support/OpenClaw/canvas//...`

El panel Canvas sirve esos archivos a través de un **esquema de URL personalizado**:

-   `openclaw-canvas:///`

Ejemplos:

-   `openclaw-canvas://main/` → `/main/index.html`
-   `openclaw-canvas://main/assets/app.css` → `/main/assets/app.css`
-   `openclaw-canvas://main/widgets/todo/` → `/main/widgets/todo/index.html`

Si no existe un `index.html` en la raíz, la aplicación muestra una **página de andamiaje integrada**.

## Comportamiento del panel

-   Panel sin bordes, redimensionable, anclado cerca de la barra de menús (o del cursor del ratón).
-   Recuerda el tamaño/posición por sesión.
-   Se recarga automáticamente cuando cambian los archivos locales de canvas.
-   Solo un panel Canvas es visible a la vez (la sesión se cambia según sea necesario).

Canvas se puede desactivar desde Configuración → **Permitir Canvas**. Cuando está desactivado, los comandos del nodo canvas devuelven `CANVAS_DISABLED`.

## Superficie de la API del Agente

Canvas se expone a través del **WebSocket de la Pasarela (Gateway)**, por lo que el agente puede:

-   mostrar/ocultar el panel
-   navegar a una ruta o URL
-   evaluar JavaScript
-   capturar una imagen instantánea

Ejemplos de CLI:

```bash
openclaw nodes canvas present --node <id>
openclaw nodes canvas navigate --node <id> --url "/"
openclaw nodes canvas eval --node <id> --js "document.title"
openclaw nodes canvas snapshot --node <id>
```

Notas:

-   `canvas.navigate` acepta **rutas locales de canvas**, URLs `http(s)` y URLs `file://`.
-   Si pasas `"/"`, el Canvas muestra el andamiaje local o `index.html`.

## A2UI en Canvas

A2UI está alojado por el host canvas de la Pasarela y se renderiza dentro del panel Canvas. Cuando la Pasarela anuncia un host Canvas, la aplicación de macOS navega automáticamente a la página del host A2UI en la primera apertura. URL del host A2UI por defecto:

```
http://<gateway-host>:18789/__openclaw__/a2ui/
```

### Comandos de A2UI (v0.8)

Canvas actualmente acepta mensajes **A2UI v0.8** de servidor→cliente:

-   `beginRendering`
-   `surfaceUpdate`
-   `dataModelUpdate`
-   `deleteSurface`

`createSurface` (v0.9) no es compatible. Ejemplo de CLI:

```bash
cat > /tmp/a2ui-v0.8.jsonl <<'EOFA2'
{"surfaceUpdate":{"surfaceId":"main","components":[{"id":"root","component":{"Column":{"children":{"explicitList":["title","content"]}}}},{"id":"title","component":{"Text":{"text":{"literalString":"Canvas (A2UI v0.8)"},"usageHint":"h1"}}},{"id":"content","component":{"Text":{"text":{"literalString":"If you can read this, A2UI push works."},"usageHint":"body"}}}]}}
{"beginRendering":{"surfaceId":"main","root":"root"}}
EOFA2

openclaw nodes canvas a2ui push --jsonl /tmp/a2ui-v0.8.jsonl --node <id>
```

Prueba rápida:

```bash
openclaw nodes canvas a2ui push --node <id> --text "Hello from A2UI"
```

## Desencadenar ejecuciones del agente desde Canvas

Canvas puede desencadenar nuevas ejecuciones del agente a través de enlaces profundos (deep links):

-   `openclaw://agent?...`

Ejemplo (en JS):

```bash
window.location.href = "openclaw://agent?message=Review%20this%20design";
```

La aplicación solicita confirmación a menos que se proporcione una clave válida.

## Notas de seguridad

-   El esquema Canvas bloquea el salto de directorios (directory traversal); los archivos deben residir bajo la raíz de la sesión.
-   El contenido local de Canvas usa un esquema personalizado (no se requiere un servidor de loopback).
-   Las URLs externas `http(s)` solo se permiten cuando se navega explícitamente a ellas.

[WebChat](./webchat.md)[Ciclo de Vida de la Pasarela (Gateway)](./child-process.md)

---