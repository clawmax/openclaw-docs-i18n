

  Medios y dispositivos

  
# Solución de problemas de nodos

Usa esta página cuando un nodo sea visible en el estado pero las herramientas del nodo fallen.

## Escalera de comandos

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Luego ejecuta comprobaciones específicas del nodo:

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
```

Señales de salud:

-   El nodo está conectado y emparejado para el rol `node`.
-   `nodes describe` incluye la capacidad que estás llamando.
-   Las aprobaciones de ejecución muestran el modo/lista de permitidos esperado.

## Requisitos en primer plano

`canvas.*`, `camera.*` y `screen.*` solo funcionan en primer plano en nodos iOS/Android. Comprobación y solución rápida:

```bash
openclaw nodes describe --node <idOrNameOrIp>
openclaw nodes canvas snapshot --node <idOrNameOrIp>
openclaw logs --follow
```

Si ves `NODE_BACKGROUND_UNAVAILABLE`, lleva la aplicación del nodo al primer plano y vuelve a intentarlo.

## Matriz de permisos

| Capacidad | iOS | Android | Aplicación de nodo macOS | Código de error típico |
| --- | --- | --- | --- | --- |
| `camera.snap`, `camera.clip` | Cámara (+ micrófono para audio de clip) | Cámara (+ micrófono para audio de clip) | Cámara (+ micrófono para audio de clip) | `*_PERMISSION_REQUIRED` |
| `screen.record` | Grabación de pantalla (+ micrófono opcional) | Solicitud de captura de pantalla (+ micrófono opcional) | Grabación de pantalla | `*_PERMISSION_REQUIRED` |
| `location.get` | Mientras se usa o Siempre (depende del modo) | Ubicación en primer/segundo plano según el modo | Permiso de ubicación | `LOCATION_PERMISSION_REQUIRED` |
| `system.run` | n/a (ruta del host del nodo) | n/a (ruta del host del nodo) | Se requieren aprobaciones de ejecución | `SYSTEM_RUN_DENIED` |

## Emparejamiento versus aprobaciones

Estas son barreras diferentes:

1.  **Emparejamiento del dispositivo**: ¿puede este nodo conectarse a la puerta de enlace?
2.  **Aprobaciones de ejecución**: ¿puede este nodo ejecutar un comando de shell específico?

Comprobaciones rápidas:

```bash
openclaw devices list
openclaw nodes status
openclaw approvals get --node <idOrNameOrIp>
openclaw approvals allowlist add --node <idOrNameOrIp> "/usr/bin/uname"
```

Si falta el emparejamiento, aprueba primero el dispositivo del nodo. Si el emparejamiento está bien pero `system.run` falla, corrige las aprobaciones de ejecución/lista de permitidos.

## Códigos de error comunes del nodo

-   `NODE_BACKGROUND_UNAVAILABLE` → la aplicación está en segundo plano; llévala al primer plano.
-   `CAMERA_DISABLED` → el interruptor de la cámara está desactivado en la configuración del nodo.
-   `*_PERMISSION_REQUIRED` → permiso del sistema operativo faltante/denegado.
-   `LOCATION_DISABLED` → el modo de ubicación está apagado.
-   `LOCATION_PERMISSION_REQUIRED` → el modo de ubicación solicitado no fue concedido.
-   `LOCATION_BACKGROUND_UNAVAILABLE` → la aplicación está en segundo plano pero solo existe el permiso "Mientras se usa".
-   `SYSTEM_RUN_DENIED: approval required` → la solicitud de ejecución necesita aprobación explícita.
-   `SYSTEM_RUN_DENIED: allowlist miss` → el comando está bloqueado por el modo de lista de permitidos. En hosts de nodos Windows, formas como envoltorios de shell `cmd.exe /c ...` se tratan como fallos de lista de permitidos en modo de lista de permitidos a menos que se aprueben mediante el flujo de solicitud.

## Bucle de recuperación rápida

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
openclaw logs --follow
```

Si aún estás atascado:

-   Re-aprueba el emparejamiento del dispositivo.
-   Reabre la aplicación del nodo (en primer plano).
-   Re-concede los permisos del sistema operativo.
-   Recrea/ajusta la política de aprobación de ejecución.

Relacionado:

-   [/nodes/index](./index.md)
-   [/nodes/camera](./camera.md)
-   [/nodes/location-command](./location-command.md)
-   [/tools/exec-approvals](../tools/exec-approvals.md)
-   [/gateway/pairing](../gateway/pairing.md)

[Nodos](../nodes.md)[Comprensión de medios](./media-understanding.md)