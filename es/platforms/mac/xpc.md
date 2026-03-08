

  Aplicación complementaria para macOS

  
# IPC de macOS

**Modelo actual:** un socket Unix local conecta el **servicio host node** con la **aplicación de macOS** para aprobaciones de ejecución y `system.run`. Existe una CLI de depuración `openclaw-mac` para verificación de descubrimiento/conexión; las acciones del agente aún fluyen a través del WebSocket del Gateway y `node.invoke`. La automatización de UI utiliza PeekabooBridge.

## Objetivos

-   Una única instancia de aplicación GUI que maneja todo el trabajo relacionado con TCC (notificaciones, grabación de pantalla, micrófono, voz, AppleScript).
-   Una superficie pequeña para automatización: Gateway + comandos node, más PeekabooBridge para automatización de UI.
-   Permisos predecibles: siempre el mismo ID de paquete firmado, lanzado por launchd, para que las concesiones de TCC persistan.

## Cómo funciona

### Transporte Gateway + node

-   La aplicación ejecuta el Gateway (modo local) y se conecta a él como un nodo.
-   Las acciones del agente se realizan mediante `node.invoke` (por ejemplo, `system.run`, `system.notify`, `canvas.*`).

### IPC del servicio node + aplicación

-   Un servicio host node sin interfaz se conecta al WebSocket del Gateway.
-   Las solicitudes de `system.run` se reenvían a la aplicación de macOS a través de un socket Unix local.
-   La aplicación realiza la ejecución en el contexto de UI, solicita confirmación si es necesario, y devuelve la salida.

Diagrama (SCI):

```
Agente -> Gateway -> Servicio Node (WS)
                      |  IPC (UDS + token + HMAC + TTL)
                      v
                  Aplicación Mac (UI + TCC + system.run)
```

### PeekabooBridge (automatización de UI)

-   La automatización de UI utiliza un socket UNIX separado llamado `bridge.sock` y el protocolo JSON de PeekabooBridge.
-   Orden de preferencia del host (lado del cliente): Peekaboo.app → Claude.app → OpenClaw.app → ejecución local.
-   Seguridad: los hosts del puente requieren un TeamID permitido; la vía de escape solo-DEBUG para el mismo UID está protegida por `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1` (convención de Peekaboo).
-   Ver: [Uso de PeekabooBridge](./peekaboo.md) para más detalles.

## Flujos operativos

-   Reinicio/recompilación: `SIGN_IDENTITY="Apple Development:  ()" scripts/restart-mac.sh`
    -   Termina las instancias existentes
    -   Compilación Swift + empaquetado
    -   Escribe/arranca/inicia el LaunchAgent
-   Instancia única: la aplicación termina temprano si otra instancia con el mismo ID de paquete está en ejecución.

## Notas de fortalecimiento

-   Preferir requerir una coincidencia de TeamID para todas las superficies privilegiadas.
-   PeekabooBridge: `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1` (solo-DEBUG) puede permitir llamantes del mismo UID para desarrollo local.
-   Toda la comunicación permanece solo local; no se exponen sockets de red.
-   Las solicitudes de TCC se originan únicamente desde el paquete de la aplicación GUI; mantener el ID del paquete firmado estable entre recompilaciones.
-   Fortalecimiento de IPC: modo de socket `0600`, token, verificaciones de UID del par, desafío/respuesta HMAC, TTL corto.

[Gateway en macOS](./bundled-gateway.md)[Habilidades](./skills.md)