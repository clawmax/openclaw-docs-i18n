

  Comandos CLI

  
# node

Ejecuta un **host de nodo sin cabeza** que se conecta al WebSocket de la Puerta de enlace y expone `system.run` / `system.which` en esta máquina.

## ¿Por qué usar un host de nodo?

Usa un host de nodo cuando quieras que los agentes **ejecuten comandos en otras máquinas** de tu red sin instalar allí una aplicación compañera completa de macOS. Casos de uso comunes:

-   Ejecutar comandos en máquinas Linux/Windows remotas (servidores de compilación, máquinas de laboratorio, NAS).
-   Mantener la ejecución **en un sandbox** en la puerta de enlace, pero delegar ejecuciones aprobadas a otros hosts.
-   Proporcionar un objetivo de ejecución ligero y sin cabeza para automatización o nodos de CI.

La ejecución sigue estando protegida por **aprobaciones de ejecución** y listas de permitidos por agente en el host de nodo, por lo que puedes mantener el acceso a comandos delimitado y explícito.

## Proxy del navegador (configuración cero)

Los hosts de nodo anuncian automáticamente un proxy del navegador si `browser.enabled` no está deshabilitado en el nodo. Esto permite que el agente use automatización del navegador en ese nodo sin configuración adicional. Deshabilítalo en el nodo si es necesario:

```json
{
  nodeHost: {
    browserProxy: {
      enabled: false,
    },
  },
}
```

## Ejecutar (en primer plano)

```bash
openclaw node run --host <gateway-host> --port 18789
```

Opciones:

-   `--host `: Host del WebSocket de la Puerta de enlace (predeterminado: `127.0.0.1`)
-   `--port `: Puerto del WebSocket de la Puerta de enlace (predeterminado: `18789`)
-   `--tls`: Usar TLS para la conexión a la puerta de enlace
-   `--tls-fingerprint `: Huella digital esperada del certificado TLS (sha256)
-   `--node-id `: Anular el id del nodo (borra el token de emparejamiento)
-   `--display-name `: Anular el nombre para mostrar del nodo

## Servicio (en segundo plano)

Instala un host de nodo sin cabeza como un servicio de usuario.

```bash
openclaw node install --host <gateway-host> --port 18789
```

Opciones:

-   `--host `: Host del WebSocket de la Puerta de enlace (predeterminado: `127.0.0.1`)
-   `--port `: Puerto del WebSocket de la Puerta de enlace (predeterminado: `18789`)
-   `--tls`: Usar TLS para la conexión a la puerta de enlace
-   `--tls-fingerprint `: Huella digital esperada del certificado TLS (sha256)
-   `--node-id `: Anular el id del nodo (borra el token de emparejamiento)
-   `--display-name `: Anular el nombre para mostrar del nodo
-   `--runtime `: Entorno de ejecución del servicio (`node` o `bun`)
-   `--force`: Reinstalar/sobrescribir si ya está instalado

Gestiona el servicio:

```bash
openclaw node status
openclaw node stop
openclaw node restart
openclaw node uninstall
```

Usa `openclaw node run` para un host de nodo en primer plano (sin servicio). Los comandos de servicio aceptan `--json` para salida legible por máquina.

## Emparejamiento

La primera conexión crea una solicitud de emparejamiento de dispositivo pendiente (`role: node`) en la Puerta de enlace. Aprueba mediante:

```bash
openclaw devices list
openclaw devices approve <requestId>
```

El host de nodo almacena su id de nodo, token, nombre para mostrar e información de conexión a la puerta de enlace en `~/.openclaw/node.json`.

## Aprobaciones de ejecución

`system.run` está controlado por aprobaciones de ejecución locales:

-   `~/.openclaw/exec-approvals.json`
-   [Aprobaciones de ejecución](../tools/exec-approvals.md)
-   `openclaw approvals --node <id|name|ip>` (editar desde la Puerta de enlace)

[models](./models.md)[nodes](./nodes.md)

---