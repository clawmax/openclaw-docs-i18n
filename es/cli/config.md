

  Comandos CLI

  
# config

Ayudantes de configuración: obtener/establecer/eliminar/validar valores por ruta e imprimir el archivo de configuración activo. Ejecutar sin un subcomando abre el asistente de configuración (igual que `openclaw configure`).

## Ejemplos

```bash
openclaw config file
openclaw config get browser.executablePath
openclaw config set browser.executablePath "/usr/bin/google-chrome"
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
openclaw config unset tools.web.search.apiKey
openclaw config validate
openclaw config validate --json
```

## Rutas

Las rutas usan notación de punto o corchetes:

```bash
openclaw config get agents.defaults.workspace
openclaw config get agents.list[0].id
```

Usa el índice de la lista de agentes para apuntar a un agente específico:

```bash
openclaw config get agents.list
openclaw config set agents.list[1].tools.exec.node "node-id-or-name"
```

## Valores

Los valores se analizan como JSON5 cuando es posible; de lo contrario, se tratan como cadenas. Usa `--strict-json` para requerir análisis JSON5. `--json` sigue siendo compatible como un alias heredado.

```bash
openclaw config set agents.defaults.heartbeat.every "0m"
openclaw config set gateway.port 19001 --strict-json
openclaw config set channels.whatsapp.groups '["*"]' --strict-json
```

## Subcomandos

-   `config file`: Imprime la ruta del archivo de configuración activo (resuelta desde `OPENCLAW_CONFIG_PATH` o la ubicación por defecto).

Reinicia la puerta de enlace después de las ediciones.

## Validar

Valida la configuración actual contra el esquema activo sin iniciar la puerta de enlace.

```bash
openclaw config validate
openclaw config validate --json
```

[completion](./completion.md)[configure](./configure.md)

---