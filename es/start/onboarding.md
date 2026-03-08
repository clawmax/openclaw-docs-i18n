

  Primeros pasos

  
# Incorporación (Aplicación macOS)

Este documento describe el flujo de incorporación **actual** del primer uso. El objetivo es una experiencia fluida del "día 0": elegir dónde se ejecuta el Gateway, conectar la autenticación, ejecutar el asistente y dejar que el agente se arranque automáticamente. Para una visión general de las rutas de incorporación, consulta [Visión General de la Incorporación](./onboarding-overview.md).

### Paso 1: Aprobar la advertencia de macOS

![](../images/start-01-macos-warning.jpeg.md)

### Paso 2: Aprobar la búsqueda de redes locales

![](../images/start-02-local-networks.jpeg.md)

### Paso 3: Bienvenida y aviso de seguridad

![](../images/start-03-security-notice.png.md)

Modelo de confianza de seguridad:

-   Por defecto, OpenClaw es un agente personal: un único límite de operador confiable.
-   Las configuraciones compartidas/multiusuario requieren bloqueo (separar límites de confianza, mantener el acceso a herramientas al mínimo y seguir [Seguridad](../gateway/security.md)).
-   La incorporación local ahora establece por defecto las nuevas configuraciones en `tools.profile: "coding"` para que las configuraciones locales nuevas mantengan las herramientas del sistema de archivos/tiempo de ejecución sin forzar el perfil sin restricciones `full`.
-   Si se habilitan hooks/webhooks u otras fuentes de contenido no confiables, utiliza un nivel de modelo moderno robusto y mantén una política/herramientas de aislamiento estricta.

### Paso 4: Local vs Remoto

![](../images/start-04-choose-gateway.png.md)

¿Dónde se ejecuta el **Gateway**?

-   **Este Mac (Solo local):** la incorporación puede configurar la autenticación y escribir las credenciales localmente.
-   **Remoto (sobre SSH/Tailnet):** la incorporación **no** configura la autenticación local; las credenciales deben existir en el host del gateway.
-   **Configurar más tarde:** omitir la configuración y dejar la aplicación sin configurar.

> **💡** **Consejo de autenticación del Gateway:**
>
> -   El asistente ahora genera un **token** incluso para loopback, por lo que los clientes WS locales deben autenticarse.
> -   Si deshabilitas la autenticación, cualquier proceso local puede conectarse; úsalo solo en máquinas completamente confiables.
> -   Usa un **token** para acceso multi-máquina o enlaces no loopback.

### Paso 5: Permisos

![](../images/start-05-permissions.png.md)

La incorporación solicita los permisos TCC necesarios para:

-   Automatización (AppleScript)
-   Notificaciones
-   Accesibilidad
-   Grabación de pantalla
-   Micrófono
-   Reconocimiento de voz
-   Cámara
-   Ubicación

### Paso 6: CLI

> **ℹ️** Este paso es opcional

 La aplicación puede instalar la CLI global `openclaw` vía npm/pnpm para que los flujos de trabajo de terminal y las tareas de launchd funcionen sin configuración adicional.

### Paso 7: Chat de Incorporación (sesión dedicada)

Después de la configuración, la aplicación abre una sesión de chat de incorporación dedicada para que el agente pueda presentarse y guiar los siguientes pasos. Esto mantiene la guía del primer uso separada de tu conversación normal. Consulta [Arranque Automático](./bootstrapping.md) para saber qué sucede en el host del gateway durante la primera ejecución del agente.

[Incorporación: CLI](./wizard.md)[Configuración del Asistente Personal](./openclaw.md)

---