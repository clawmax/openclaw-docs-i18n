

  Comandos CLI

  
# approvals

Gestiona las aprobaciones de ejecución para el **host local**, el **host de la puerta de enlace** o un **host de nodo**. Por defecto, los comandos apuntan al archivo de aprobaciones local en el disco. Usa `--gateway` para apuntar a la puerta de enlace, o `--node` para apuntar a un nodo específico. Relacionado:

-   Aprobaciones de ejecución: [Aprobaciones de ejecución](../tools/exec-approvals.md)
-   Nodos: [Nodos](../nodes.md)

## Comandos comunes

```bash
openclaw approvals get
openclaw approvals get --node <id|name|ip>
openclaw approvals get --gateway
```

## Reemplazar aprobaciones desde un archivo

```bash
openclaw approvals set --file ./exec-approvals.json
openclaw approvals set --node <id|name|ip> --file ./exec-approvals.json
openclaw approvals set --gateway --file ./exec-approvals.json
```

## Ayudas de lista de permitidos

```bash
openclaw approvals allowlist add "~/Projects/**/bin/rg"
openclaw approvals allowlist add --agent main --node <id|name|ip> "/usr/bin/uptime"
openclaw approvals allowlist add --agent "*" "/usr/bin/uname"

openclaw approvals allowlist remove "~/Projects/**/bin/rg"
```

## Notas

-   `--node` usa el mismo resolvedor que `openclaw nodes` (id, nombre, ip o prefijo de id).
-   `--agent` tiene como valor predeterminado `"*"`, que se aplica a todos los agentes.
-   El host del nodo debe anunciar `system.execApprovals.get/set` (aplicación macOS o host de nodo headless).
-   Los archivos de aprobaciones se almacenan por host en `~/.openclaw/exec-approvals.json`.

[agents](./agents.md)[browser](./browser.md)

---