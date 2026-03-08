

  Aplicación complementaria de macOS

  
# Ciclo de Vida del Gateway

La aplicación de macOS **gestiona el Gateway a través de launchd** por defecto y no genera el Gateway como un proceso hijo. Primero intenta adjuntarse a un Gateway ya en ejecución en el puerto configurado; si no se puede alcanzar ninguno, habilita el servicio de launchd mediante la CLI externa `openclaw` (sin tiempo de ejecución integrado). Esto te proporciona un inicio automático confiable al iniciar sesión y reinicio ante fallos. El modo de proceso hijo (Gateway generado directamente por la aplicación) **no está en uso** actualmente. Si necesitas un acoplamiento más estrecho con la interfaz de usuario, ejecuta el Gateway manualmente en una terminal.

## Comportamiento por defecto (launchd)

-   La aplicación instala un LaunchAgent por usuario etiquetado como `ai.openclaw.gateway` (o `ai.openclaw.` cuando se usa `--profile`/`OPENCLAW_PROFILE`; se admite el legado `com.openclaw.*`).
-   Cuando el modo Local está habilitado, la aplicación se asegura de que el LaunchAgent esté cargado e inicia el Gateway si es necesario.
-   Los registros se escriben en la ruta de registro del gateway de launchd (visible en Configuración de Depuración).

Comandos comunes:

```bash
launchctl kickstart -k gui/$UID/ai.openclaw.gateway
launchctl bootout gui/$UID/ai.openclaw.gateway
```

Reemplaza la etiqueta con `ai.openclaw.` cuando ejecutes un perfil con nombre.

## Compilaciones de desarrollo no firmadas

`scripts/restart-mac.sh --no-sign` es para compilaciones locales rápidas cuando no tienes claves de firma. Para evitar que launchd apunte a un binario de relay no firmado, hace lo siguiente:

-   Escribe `~/.openclaw/disable-launchagent`.

Las ejecuciones firmadas de `scripts/restart-mac.sh` borran esta anulación si el marcador está presente. Para restablecer manualmente:

```bash
rm ~/.openclaw/disable-launchagent
```

## Modo solo adjuntar

Para forzar a que la aplicación de macOS **nunca instale o gestione launchd**, lánzala con `--attach-only` (o `--no-launchd`). Esto establece `~/.openclaw/disable-launchagent`, por lo que la aplicación solo se adjunta a un Gateway ya en ejecución. Puedes alternar el mismo comportamiento en Configuración de Depuración.

## Modo remoto

El modo remoto nunca inicia un Gateway local. La aplicación utiliza un túnel SSH hacia el host remoto y se conecta a través de ese túnel.

## Por qué preferimos launchd

-   Inicio automático al iniciar sesión.
-   Semántica de reinicio/KeepAlive incorporada.
-   Registros y supervisión predecibles.

Si alguna vez se necesita nuevamente un modo verdadero de proceso hijo, debe documentarse como un modo separado y explícito solo para desarrollo.

[Canvas](./canvas.md)[Comprobaciones de Salud](./health.md)

---