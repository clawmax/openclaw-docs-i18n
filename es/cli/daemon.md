

  Comandos CLI

  
# daemon

Alias heredado para los comandos de gestión del servicio Gateway. `openclaw daemon ...` se asigna a la misma superficie de control de servicio que los comandos de servicio `openclaw gateway ...`.

## Uso

```bash
openclaw daemon status
openclaw daemon install
openclaw daemon start
openclaw daemon stop
openclaw daemon restart
openclaw daemon uninstall
```

## Subcomandos

-   `status`: muestra el estado de instalación del servicio y sondea la salud del Gateway
-   `install`: instala el servicio (`launchd`/`systemd`/`schtasks`)
-   `uninstall`: elimina el servicio
-   `start`: inicia el servicio
-   `stop`: detiene el servicio
-   `restart`: reinicia el servicio

## Opciones comunes

-   `status`: `--url`, `--token`, `--password`, `--timeout`, `--no-probe`, `--deep`, `--json`
-   `install`: `--port`, `--runtime <node|bun>`, `--token`, `--force`, `--json`
-   ciclo de vida (`uninstall|start|stop|restart`): `--json`

Notas:

-   `status` resuelve los SecretRefs de autenticación configurados para la autenticación de sondeo cuando es posible.
-   Cuando la autenticación por token requiere un token y `gateway.auth.token` está gestionado por SecretRef, `install` valida que el SecretRef sea resoluble pero no persiste el token resuelto en los metadatos del entorno del servicio.
-   Si la autenticación por token requiere un token y el SecretRef del token configurado no está resuelto, la instalación falla de forma segura (closed).
-   Si tanto `gateway.auth.token` como `gateway.auth.password` están configurados y `gateway.auth.mode` no está establecido, la instalación se bloquea hasta que el modo se establezca explícitamente.

## Se recomienda

Usa [`openclaw gateway`](./gateway.md) para la documentación y ejemplos actuales.

[cron](./cron.md)[dashboard](./dashboard.md)

---