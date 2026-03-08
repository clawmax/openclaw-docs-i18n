title: "Monitorizar el estado de salud de la aplicación macOS y los canales vinculados"
description: "Aprende a comprobar el estado de salud de tus canales vinculados de WhatsApp y Telegram desde la aplicación de la barra de menús de macOS, interpretar los indicadores de estado y ejecutar pruebas manuales."
keywords: ["comprobación de salud macos", "salud openclaw", "estado barra de menús", "salud baileys", "estado canal vinculado", "prueba whatsapp", "prueba telegram", "aplicación compañera mac"]
---

  Aplicación compañera de macOS

  
# Comprobaciones de Salud

Cómo ver si el canal vinculado está saludable desde la aplicación de la barra de menús.

## Barra de menús

-   El punto de estado ahora refleja la salud de Baileys:
    -   Verde: vinculado + socket abierto recientemente.
    -   Naranja: conectando/reintentando.
    -   Rojo: desconectado o la prueba falló.
-   La línea secundaria muestra "vinculado · auth 12m" o muestra el motivo del fallo.
-   El elemento de menú "Ejecutar Comprobación de Salud" activa una prueba bajo demanda.

## Configuración

-   La pestaña General incluye una tarjeta de Salud que muestra: antigüedad de la autenticación vinculada, ruta/recuento del almacén de sesiones, hora de la última comprobación, último error/código de estado, y botones para Ejecutar Comprobación de Salud / Mostrar Registros.
-   Utiliza una instantánea en caché para que la interfaz de usuario cargue al instante y degrade suavemente cuando esté sin conexión.
-   La **pestaña Canales** muestra el estado del canal + controles para WhatsApp/Telegram (código QR de inicio de sesión, cerrar sesión, prueba, última desconexión/error).

## Cómo funciona la prueba

-   La aplicación ejecuta `openclaw health --json` a través de `ShellExecutor` aproximadamente cada ~60s y bajo demanda. La prueba carga las credenciales e informa del estado sin enviar mensajes.
-   Almacena en caché la última instantánea correcta y el último error por separado para evitar parpadeos; muestra la marca de tiempo de cada uno.

## En caso de duda

-   Todavía puedes usar el flujo de CLI en [Salud de la Pasarela](../../gateway/health.md) (`openclaw status`, `openclaw status --deep`, `openclaw health --json`) y seguir la cola de `/tmp/openclaw/openclaw-*.log` para `web-heartbeat` / `web-reconnect`.

[Ciclo de Vida de la Pasarela](./child-process.md)[Icono de la Barra de Menús](./icon.md)