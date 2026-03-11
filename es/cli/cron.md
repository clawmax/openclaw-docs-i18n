

  Comandos CLI

  
# cron

Gestiona trabajos cron para el programador del Gateway. Relacionado:

-   Trabajos cron: [Trabajos cron](../automation/cron-jobs.md)

Consejo: ejecuta `openclaw cron --help` para ver toda la superficie del comando. Nota: los trabajos aislados `cron add` tienen por defecto entrega `--announce`. Usa `--no-deliver` para mantener la salida interna. `--deliver` permanece como un alias obsoleto para `--announce`. Nota: los trabajos de una sola ejecución (`--at`) se eliminan tras el éxito por defecto. Usa `--keep-after-run` para conservarlos. Nota: los trabajos recurrentes ahora usan retroceso exponencial de reintentos tras errores consecutivos (30s → 1m → 5m → 15m → 60m), luego vuelven al horario normal tras la siguiente ejecución exitosa. Nota: la retención/poda se controla en la configuración:

-   `cron.sessionRetention` (por defecto `24h`) poda las sesiones de ejecución aisladas completadas.
-   `cron.runLog.maxBytes` + `cron.runLog.keepLines` podan `~/.openclaw/cron/runs/.jsonl`.

## Ediciones comunes

Actualiza la configuración de entrega sin cambiar el mensaje:

```bash
openclaw cron edit <job-id> --announce --channel telegram --to "123456789"
```

Desactiva la entrega para un trabajo aislado:

```bash
openclaw cron edit <job-id> --no-deliver
```

Habilita el contexto de arranque ligero para un trabajo aislado:

```bash
openclaw cron edit <job-id> --light-context
```

Anuncia a un canal específico:

```bash
openclaw cron edit <job-id> --announce --channel slack --to "channel:C1234567890"
```

Crea un trabajo aislado con contexto de arranque ligero:

```bash
openclaw cron add \
  --name "Resumen matutino ligero" \
  --cron "0 7 * * *" \
  --session isolated \
  --message "Resume las actualizaciones nocturnas." \
  --light-context \
  --no-deliver
```

`--light-context` se aplica solo a trabajos aislados de turno de agente. Para ejecuciones cron, el modo ligero mantiene el contexto de arranque vacío en lugar de inyectar el conjunto completo de arranque del espacio de trabajo.

[configurar](./configure.md)[daemon](./daemon.md)