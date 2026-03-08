

  Aplicación complementaria para macOS

  
# Puente Peekaboo

OpenClaw puede alojar **PeekabooBridge** como un intermediario local y consciente de permisos para la automatización de la interfaz de usuario. Esto permite que la CLI `peekaboo` impulse la automatización de la interfaz de usuario mientras reutiliza los permisos TCC de la aplicación de macOS.

## Qué es esto (y qué no es)

-   **Anfitrión**: OpenClaw.app puede actuar como anfitrión de PeekabooBridge.
-   **Cliente**: usa la CLI `peekaboo` (no hay una superficie separada `openclaw ui ...`).
-   **Interfaz de usuario**: las superposiciones visuales permanecen en Peekaboo.app; OpenClaw es un anfitrión intermediario ligero.

## Habilitar el puente

En la aplicación de macOS:

-   Configuración → **Habilitar Puente Peekaboo**

Cuando está habilitado, OpenClaw inicia un servidor de socket UNIX local. Si se deshabilita, el anfitrión se detiene y `peekaboo` recurrirá a otros anfitriones disponibles.

## Orden de descubrimiento del cliente

Los clientes de Peekaboo normalmente intentan los anfitriones en este orden:

1.  Peekaboo.app (experiencia de usuario completa)
2.  Claude.app (si está instalada)
3.  OpenClaw.app (intermediario ligero)

Usa `peekaboo bridge status --verbose` para ver qué anfitrión está activo y qué ruta de socket se está utilizando. Puedes anularlo con:

```bash
export PEEKABOO_BRIDGE_SOCKET=/path/to/bridge.sock
```

## Seguridad y permisos

-   El puente valida las **firmas de código del llamante**; se aplica una lista de permitidos de TeamID (TeamID del anfitrión Peekaboo + TeamID de la aplicación OpenClaw).
-   Las solicitudes caducan después de ~10 segundos.
-   Si faltan los permisos requeridos, el puente devuelve un mensaje de error claro en lugar de abrir Preferencias del Sistema.

## Comportamiento de instantáneas (automatización)

Las instantáneas se almacenan en memoria y caducan automáticamente después de un breve período. Si necesitas una retención más larga, vuelve a capturar desde el cliente.

## Resolución de problemas

-   Si `peekaboo` informa "bridge client is not authorized", asegúrate de que el cliente esté firmado correctamente o ejecuta el anfitrión con `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1` solo en modo **depuración**.
-   Si no se encuentran anfitriones, abre una de las aplicaciones anfitrionas (Peekaboo.app o OpenClaw.app) y confirma que se hayan otorgado los permisos.

[Habilidades](./skills.md)

---