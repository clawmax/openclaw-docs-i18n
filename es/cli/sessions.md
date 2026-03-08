

  Comandos CLI

  
# sessions

Lista las sesiones de conversación almacenadas.

```bash
openclaw sessions
openclaw sessions --agent work
openclaw sessions --all-agents
openclaw sessions --active 120
openclaw sessions --json
```

Selección de alcance:

-   predeterminado: almacén del agente predeterminado configurado
-   `--agent `: un almacén de agente configurado
-   `--all-agents`: agregar todos los almacenes de agentes configurados
-   `--store `: ruta de almacén explícita (no se puede combinar con `--agent` o `--all-agents`)

Ejemplos JSON: `openclaw sessions --all-agents --json`:

```json
{
  "path": null,
  "stores": [
    { "agentId": "main", "path": "/home/user/.openclaw/agents/main/sessions/sessions.json" },
    { "agentId": "work", "path": "/home/user/.openclaw/agents/work/sessions/sessions.json" }
  ],
  "allAgents": true,
  "count": 2,
  "activeMinutes": null,
  "sessions": [
    { "agentId": "main", "key": "agent:main:main", "model": "gpt-5" },
    { "agentId": "work", "key": "agent:work:main", "model": "claude-opus-4-5" }
  ]
}
```

## Mantenimiento de limpieza

Ejecuta el mantenimiento ahora (en lugar de esperar al siguiente ciclo de escritura):

```bash
openclaw sessions cleanup --dry-run
openclaw sessions cleanup --agent work --dry-run
openclaw sessions cleanup --all-agents --dry-run
openclaw sessions cleanup --enforce
openclaw sessions cleanup --enforce --active-key "agent:main:telegram:dm:123"
openclaw sessions cleanup --json
```

`openclaw sessions cleanup` utiliza la configuración `session.maintenance` del archivo de configuración:

-   Nota sobre el alcance: `openclaw sessions cleanup` mantiene solo los almacenes/transcripciones de sesiones. No poda los registros de ejecución de cron (`cron/runs/.jsonl`), los cuales son gestionados por `cron.runLog.maxBytes` y `cron.runLog.keepLines` en la [Configuración de Cron](../automation/cron-jobs.md#configuration) y explicados en [Mantenimiento de Cron](../automation/cron-jobs.md#maintenance).
-   `--dry-run`: previsualiza cuántas entradas serían podadas/limitadas sin escribir.
    -   En modo texto, el dry-run imprime una tabla de acciones por sesión (`Acción`, `Clave`, `Antigüedad`, `Modelo`, `Banderas`) para que puedas ver qué se mantendría vs qué se eliminaría.
-   `--enforce`: aplica el mantenimiento incluso cuando `session.maintenance.mode` es `warn`.
-   `--active-key `: protege una clave activa específica de la expulsión por límite de disco.
-   `--agent `: ejecuta la limpieza para un almacén de agente configurado.
-   `--all-agents`: ejecuta la limpieza para todos los almacenes de agentes configurados.
-   `--store `: ejecuta contra un archivo `sessions.json` específico.
-   `--json`: imprime un resumen JSON. Con `--all-agents`, la salida incluye un resumen por almacén.

`openclaw sessions cleanup --all-agents --dry-run --json`:

```json
{
  "allAgents": true,
  "mode": "warn",
  "dryRun": true,
  "stores": [
    {
      "agentId": "main",
      "storePath": "/home/user/.openclaw/agents/main/sessions/sessions.json",
      "beforeCount": 120,
      "afterCount": 80,
      "pruned": 40,
      "capped": 0
    },
    {
      "agentId": "work",
      "storePath": "/home/user/.openclaw/agents/work/sessions/sessions.json",
      "beforeCount": 18,
      "afterCount": 18,
      "pruned": 0,
      "capped": 0
    }
  ]
}
```

Relacionado:

-   Configuración de sesión: [Referencia de configuración](../gateway/configuration-reference.md#session)

[security](./security.md)[setup](./setup.md)