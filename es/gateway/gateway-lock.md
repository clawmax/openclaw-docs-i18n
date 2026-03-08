

  Configuración y operaciones

  
# Bloqueo de Puerta de Enlace

Última actualización: 2025-12-11

## Por qué

-   Asegurar que solo una instancia de puerta de enlace se ejecute por puerto base en el mismo host; las puertas de enlace adicionales deben usar perfiles aislados y puertos únicos.
-   Sobrevivir a caídas/SIGKILL sin dejar archivos de bloqueo obsoletos.
-   Fallar rápidamente con un error claro cuando el puerto de control ya está ocupado.

## Mecanismo

-   La puerta de enlace vincula el oyente WebSocket (por defecto `ws://127.0.0.1:18789`) inmediatamente al inicio usando un oyente TCP exclusivo.
-   Si la vinculación falla con `EADDRINUSE`, el inicio lanza `GatewayLockError("another gateway instance is already listening on ws://127.0.0.1:")`.
-   El sistema operativo libera el oyente automáticamente en cualquier salida del proceso, incluyendo caídas y SIGKILL—no se necesita un archivo de bloqueo separado ni un paso de limpieza.
-   Al apagarse, la puerta de enlace cierra el servidor WebSocket y el servidor HTTP subyacente para liberar el puerto rápidamente.

## Superficie de error

-   Si otro proceso tiene el puerto, el inicio lanza `GatewayLockError("another gateway instance is already listening on ws://127.0.0.1:")`.
-   Otros fallos de vinculación se muestran como `GatewayLockError("failed to bind gateway socket on ws://127.0.0.1:: …")`.

## Notas operativas

-   Si el puerto está ocupado por *otro* proceso, el error es el mismo; libera el puerto o elige otro con `openclaw gateway --port `.
-   La aplicación de macOS aún mantiene su propia guardia ligera de PID antes de generar la puerta de enlace; el bloqueo en tiempo de ejecución se aplica mediante la vinculación WebSocket.

[Registro](./logging.md)[Ejecución en Fondo y Herramienta de Procesos](./background-process.md)