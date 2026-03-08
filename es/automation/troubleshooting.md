

  Automatización

  
# Solución de Problemas de Automatización

Usa esta página para problemas del programador y la entrega (`cron` + `heartbeat`).

## Escalera de comandos

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Luego ejecuta las comprobaciones de automatización:

```bash
openclaw cron status
openclaw cron list
openclaw system heartbeat last
```

## Cron no se ejecuta

```bash
openclaw cron status
openclaw cron list
openclaw cron runs --id <jobId> --limit 20
openclaw logs --follow
```

Una salida correcta se ve así:

-   `cron status` reporta habilitado y un `nextWakeAtMs` futuro.
-   El trabajo está habilitado y tiene un horario/zona horaria válida.
-   `cron runs` muestra `ok` o una razón explícita de omisión.

Firmas comunes:

-   `cron: scheduler disabled; jobs will not run automatically` → cron deshabilitado en config/env.
-   `cron: timer tick failed` → el ciclo del programador falló; inspecciona el contexto de la pila/log circundante.
-   `reason: not-due` en la salida de ejecución → ejecución manual llamada sin `--force` y el trabajo aún no está programado.

## Cron se ejecutó pero no hay entrega

```bash
openclaw cron runs --id <jobId> --limit 20
openclaw cron list
openclaw channels status --probe
openclaw logs --follow
```

Una salida correcta se ve así:

-   El estado de ejecución es `ok`.
-   El modo/objetivo de entrega están configurados para trabajos aislados.
-   La sonda del canal reporta que el canal objetivo está conectado.

Firmas comunes:

-   La ejecución tuvo éxito pero el modo de entrega es `none` → no se espera un mensaje externo.
-   Objetivo de entrega faltante/no válido (`channel`/`to`) → la ejecución puede tener éxito internamente pero omitir el envío saliente.
-   Errores de autenticación del canal (`unauthorized`, `missing_scope`, `Forbidden`) → entrega bloqueada por credenciales/permisos del canal.

## Heartbeat suprimido u omitido

```bash
openclaw system heartbeat last
openclaw logs --follow
openclaw config get agents.defaults.heartbeat
openclaw channels status --probe
```

Una salida correcta se ve así:

-   Heartbeat habilitado con un intervalo distinto de cero.
-   El último resultado de heartbeat es `ran` (o se entiende la razón de omisión).

Firmas comunes:

-   `heartbeat skipped` con `reason=quiet-hours` → fuera de `activeHours`.
-   `requests-in-flight` → carril principal ocupado; heartbeat diferido.
-   `empty-heartbeat-file` → el heartbeat de intervalo se omitió porque `HEARTBEAT.md` no tiene contenido accionable y no hay ningún evento cron etiquetado en cola.
-   `alerts-disabled` → la configuración de visibilidad suprime los mensajes de heartbeat salientes.

## Trampas de zona horaria y activeHours

```bash
openclaw config get agents.defaults.heartbeat.activeHours
openclaw config get agents.defaults.heartbeat.activeHours.timezone
openclaw config get agents.defaults.userTimezone || echo "agents.defaults.userTimezone not set"
openclaw cron list
openclaw logs --follow
```

Reglas rápidas:

-   `Config path not found: agents.defaults.userTimezone` significa que la clave no está configurada; heartbeat recurre a la zona horaria del host (o a `activeHours.timezone` si está configurada).
-   Cron sin `--tz` usa la zona horaria del host del gateway.
-   `activeHours` de heartbeat usa la resolución de zona horaria configurada (`user`, `local`, o IANA tz explícita).
-   Las marcas de tiempo ISO sin zona horaria se tratan como UTC para los horarios `at` de cron.

Firmas comunes:

-   Los trabajos se ejecutan a la hora de reloj incorrecta después de cambios en la zona horaria del host.
-   Heartbeat siempre se omite durante tu horario diurno porque `activeHours.timezone` es incorrecta.

Relacionado:

-   [/automation/cron-jobs](./cron-jobs.md)
-   [/gateway/heartbeat](../gateway/heartbeat.md)
-   [/automation/cron-vs-heartbeat](./cron-vs-heartbeat.md)
-   [/concepts/timezone](../concepts/timezone.md)

[Cron vs Heartbeat](./cron-vs-heartbeat.md)[Webhooks](./webhook.md)