

  Aplicación complementaria para macOS

  
# WebChat

La aplicación de la barra de menús de macOS integra la interfaz de usuario de WebChat como una vista nativa de SwiftUI. Se conecta al Gateway y utiliza por defecto la **sesión principal** para el agente seleccionado (con un selector de sesiones para otras sesiones).

-   **Modo local**: se conecta directamente al WebSocket local del Gateway.
-   **Modo remoto**: reenvía el puerto de control del Gateway a través de SSH y utiliza ese túnel como plano de datos.

## Lanzamiento y depuración

-   Manual: Menú Lobster → "Abrir Chat".
-   Apertura automática para pruebas:
    
    Copiar
    
    ```bash
    dist/OpenClaw.app/Contents/MacOS/OpenClaw --webchat
    ```
    
-   Registros: `./scripts/clawlog.sh` (subsistema `ai.openclaw`, categoría `WebChatSwiftUI`).

## Cómo está conectado

-   Plano de datos: Métodos WS del Gateway `chat.history`, `chat.send`, `chat.abort`, `chat.inject` y eventos `chat`, `agent`, `presence`, `tick`, `health`.
-   Sesión: por defecto utiliza la sesión principal (`main`, o `global` cuando el alcance es global). La interfaz de usuario puede cambiar entre sesiones.
-   La incorporación utiliza una sesión dedicada para mantener la configuración inicial separada.

## Superficie de seguridad

-   El modo remoto solo reenvía el puerto de control del WebSocket del Gateway a través de SSH.

## Limitaciones conocidas

-   La interfaz de usuario está optimizada para sesiones de chat (no es un entorno de aislamiento de navegador completo).

[Superposición de Voz](./voice-overlay.md)[Lienzo](./canvas.md)

---