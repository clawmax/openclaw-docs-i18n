

  Comandos CLI

  
# nodes

Gestiona nodos (dispositivos) emparejados e invoca sus capacidades. Relacionado:

-   Descripción general de nodos: [Nodos](../nodes.md)
-   Cámara: [Nodos de cámara](../nodes/camera.md)
-   Imágenes: [Nodos de imágenes](../nodes/images.md)

Opciones comunes:

-   `--url`, `--token`, `--timeout`, `--json`

## Comandos comunes

```bash
openclaw nodes list
openclaw nodes list --connected
openclaw nodes list --last-connected 24h
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes status
openclaw nodes status --connected
openclaw nodes status --last-connected 24h
```

`nodes list` imprime tablas de pendientes/emparejados. Las filas de emparejados incluyen la antigüedad de la conexión más reciente (Última Conexión). Usa `--connected` para mostrar solo los nodos actualmente conectados. Usa `--last-connected <duración>` para filtrar nodos que se conectaron dentro de una duración (ej. `24h`, `7d`).

## Invocar / ejecutar

```bash
openclaw nodes invoke --node <id|name|ip> --command <command> --params <json>
openclaw nodes run --node <id|name|ip> <command...>
openclaw nodes run --raw "git status"
openclaw nodes run --agent main --node <id|name|ip> --raw "git status"
```

Banderas de invocación:

-   `--params `: cadena de objeto JSON (por defecto `{}`).
-   `--invoke-timeout `: tiempo de espera para invocar nodo (por defecto `15000`).
-   `--idempotency-key `: clave de idempotencia opcional.

### Valores predeterminados estilo exec

`nodes run` refleja el comportamiento exec del modelo (valores predeterminados + aprobaciones):

-   Lee `tools.exec.*` (más anulaciones `agents.list[].tools.exec.*`).
-   Usa aprobaciones exec (`exec.approval.request`) antes de invocar `system.run`.
-   `--node` se puede omitir cuando `tools.exec.node` está configurado.
-   Requiere un nodo que anuncie `system.run` (aplicación complementaria macOS o host de nodo headless).

Banderas:

-   `--cwd `: directorio de trabajo.
-   `--env <clave=valor>`: anulación de variable de entorno (repetible). Nota: los hosts de nodo ignoran las anulaciones de `PATH` (y `tools.exec.pathPrepend` no se aplica a hosts de nodo).
-   `--command-timeout `: tiempo de espera del comando.
-   `--invoke-timeout `: tiempo de espera para invocar nodo (por defecto `30000`).
-   `--needs-screen-recording`: requiere permiso de grabación de pantalla.
-   `--raw `: ejecuta una cadena de shell (`/bin/sh -lc` o `cmd.exe /c`). En modo de lista de permitidos en hosts de nodo Windows, las ejecuciones con envoltura de shell `cmd.exe /c` requieren aprobación (la entrada en la lista de permitidos por sí sola no permite automáticamente la forma con envoltura).
-   `--agent `: aprobaciones/listas de permitidos con alcance de agente (por defecto el agente configurado).
-   `--ask <off|on-miss|always>`, `--security <deny|allowlist|full>`: anulaciones.

[node](./node.md)[onboard](./onboard.md)

---