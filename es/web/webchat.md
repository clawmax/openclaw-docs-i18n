

  Interfaces web

  
# WebChat

Estado: la interfaz de chat SwiftUI para macOS/iOS se comunica directamente con el WebSocket del Gateway.

## Qué es

-   Una interfaz de chat nativa para el gateway (sin navegador embebido y sin servidor estático local).
-   Utiliza las mismas sesiones y reglas de enrutamiento que otros canales.
-   Enrutamiento determinista: las respuestas siempre regresan a WebChat.

## Inicio rápido

1.  Inicia el gateway.
2.  Abre la interfaz de usuario WebChat (aplicación macOS/iOS) o la pestaña de chat de la Interfaz de Control.
3.  Asegúrate de que la autenticación del gateway esté configurada (requerida por defecto, incluso en loopback).

## Cómo funciona (comportamiento)

-   La UI se conecta al WebSocket del Gateway y usa `chat.history`, `chat.send` y `chat.inject`.
-   `chat.history` tiene límites por estabilidad: el Gateway puede truncar campos de texto largos, omitir metadatos pesados y reemplazar entradas demasiado grandes con `[chat.history omitted: message too large]`.
-   `chat.inject` agrega una nota del asistente directamente a la transcripción y la transmite a la UI (sin ejecución del agente).
-   Las ejecuciones abortadas pueden mantener visible en la UI la salida parcial del asistente.
-   El Gateway persiste el texto parcial del asistente abortado en el historial de transcripción cuando existe salida en búfer, y marca esas entradas con metadatos de aborto.
-   El historial siempre se obtiene del gateway (sin observación de archivos locales).
-   Si el gateway es inalcanzable, WebChat es de solo lectura.

## Panel de herramientas de agentes en la Interfaz de Control

-   El panel de Herramientas `/agents` de la Interfaz de Control obtiene un catálogo en tiempo de ejecución mediante `tools.catalog` y etiqueta cada herramienta como `core` o `plugin:` (más `optional` para herramientas de plugin opcionales).
-   Si `tools.catalog` no está disponible, el panel recurre a una lista estática integrada.
-   El panel edita la configuración de perfil y anulaciones, pero el acceso efectivo en tiempo de ejecución aún sigue la precedencia de políticas (`allow`/`deny`, anulaciones por agente y proveedor/canal).

## Uso remoto

-   El modo remoto tuneliza el WebSocket del gateway a través de SSH/Tailscale.
-   No necesitas ejecutar un servidor WebChat separado.

## Referencia de configuración (WebChat)

Configuración completa: [Configuración](../gateway/configuration.md) Opciones de canal:

-   No hay un bloque dedicado `webchat.*`. WebChat usa la configuración de endpoint + autenticación del gateway que se indica a continuación.

Opciones globales relacionadas:

-   `gateway.port`, `gateway.bind`: host/puerto del WebSocket.
-   `gateway.auth.mode`, `gateway.auth.token`, `gateway.auth.password`: autenticación WebSocket (token/contraseña).
-   `gateway.auth.mode: "trusted-proxy"`: autenticación de proxy inverso para clientes de navegador (ver [Autenticación de Proxy de Confianza](../gateway/trusted-proxy-auth.md)).
-   `gateway.remote.url`, `gateway.remote.token`, `gateway.remote.password`: destino del gateway remoto.
-   `session.*`: almacenamiento de sesión y valores predeterminados de clave principal.

[Dashboard](./dashboard.md)[TUI](./tui.md)

---