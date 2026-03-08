

  Ayuda

  
# Resolución de Problemas

Si solo tienes 2 minutos, usa esta página como puerta de entrada de triaje.

## Primeros 60 segundos

Ejecuta esta escalera exacta en orden:

```bash
openclaw status
openclaw status --all
openclaw gateway probe
openclaw gateway status
openclaw doctor
openclaw channels status --probe
openclaw logs --follow
```

Buena salida en una línea:

-   `openclaw status` → muestra los canales configurados y sin errores de autenticación obvios.
-   `openclaw status --all` → el informe completo está presente y es compartible.
-   `openclaw gateway probe` → el objetivo de la puerta de enlace esperado es alcanzable.
-   `openclaw gateway status` → `Runtime: running` y `RPC probe: ok`.
-   `openclaw doctor` → sin errores de configuración/servicio bloqueantes.
-   `openclaw channels status --probe` → los canales reportan `connected` o `ready`.
-   `openclaw logs --follow` → actividad constante, sin errores fatales repetitivos.

## Contexto largo de Anthropic 429

Si ves: `HTTP 429: rate_limit_error: Extra usage is required for long context requests`, ve a [/gateway/troubleshooting#anthropic-429-extra-usage-required-for-long-context](../gateway/troubleshooting.md#anthropic-429-extra-usage-required-for-long-context).

## La instalación del complemento falla con extensiones openclaw faltantes

Si la instalación falla con `package.json missing openclaw.extensions`, el paquete del complemento está usando un formato antiguo que OpenClaw ya no acepta. Solución en el paquete del complemento:

1.  Agrega `openclaw.extensions` a `package.json`.
2.  Apunta las entradas a los archivos de tiempo de ejecución compilados (generalmente `./dist/index.js`).
3.  Vuelve a publicar el complemento y ejecuta `openclaw plugins install <npm-spec>` nuevamente.

Ejemplo:

```json
{
  "name": "@openclaw/my-plugin",
  "version": "1.2.3",
  "openclaw": {
    "extensions": ["./dist/index.js"]
  }
}
```

Referencia: [/tools/plugin#distribution-npm](../tools/plugin.md#distribution-npm)

## Árbol de decisión

```bash
openclaw status
openclaw gateway status
openclaw channels status --probe
openclaw pairing list --channel <channel> [--account <id>]
openclaw logs --follow
```

La buena salida se ve así:

-   `Runtime: running`
-   `RPC probe: ok`
-   Tu canal muestra conectado/listo en `channels status --probe`
-   El remitente aparece aprobado (o la política de DM está abierta/en lista blanca)

Firmas de registro comunes:

-   `drop guild message (mention required` → el control de menciones bloqueó el mensaje en Discord.
-   `pairing request` → el remitente no está aprobado y espera la aprobación de emparejamiento por DM.
-   `blocked` / `allowlist` en los registros del canal → remitente, sala o grupo está filtrado.

Páginas profundas:

-   [/gateway/troubleshooting#no-replies](../gateway/troubleshooting.md#no-replies)
-   [/channels/troubleshooting](../channels/troubleshooting.md)
-   [/channels/pairing](../channels/pairing.md)

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

La buena salida se ve así:

-   `Dashboard: http://...` se muestra en `openclaw gateway status`
-   `RPC probe: ok`
-   Sin bucle de autenticación en los registros

Firmas de registro comunes:

-   `device identity required` → el contexto HTTP/no seguro no puede completar la autenticación del dispositivo.
-   `unauthorized` / bucle de reconexión → token/contraseña incorrecta o modo de autenticación no coincide.
-   `gateway connect failed:` → la UI está apuntando a la URL/puerto incorrecto o la puerta de enlace es inalcanzable.

Páginas profundas:

-   [/gateway/troubleshooting#dashboard-control-ui-connectivity](../gateway/troubleshooting.md#dashboard-control-ui-connectivity)
-   [/web/control-ui](../web/control-ui.md)
-   [/gateway/authentication](../gateway/authentication.md)

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

La buena salida se ve así:

-   `Service: ... (loaded)`
-   `Runtime: running`
-   `RPC probe: ok`

Firmas de registro comunes:

-   `Gateway start blocked: set gateway.mode=local` → el modo de puerta de enlace no está configurado/remoto.
-   `refusing to bind gateway ... without auth` → enlace no local sin token/contraseña.
-   `another gateway instance is already listening` o `EADDRINUSE` → el puerto ya está ocupado.

Páginas profundas:

-   [/gateway/troubleshooting#gateway-service-not-running](../gateway/troubleshooting.md#gateway-service-not-running)
-   [/gateway/background-process](../gateway/background-process.md)
-   [/gateway/configuration](../gateway/configuration.md)

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

La buena salida se ve así:

-   El transporte del canal está conectado.
-   Las comprobaciones de emparejamiento/lista blanca pasan.
-   Se detectan menciones donde se requieren.

Firmas de registro comunes:

-   `mention required` → el control de menciones grupales bloqueó el procesamiento.
-   `pairing` / `pending` → el remitente por DM aún no está aprobado.
-   `not_in_channel`, `missing_scope`, `Forbidden`, `401/403` → problema de token de permiso del canal.

Páginas profundas:

-   [/gateway/troubleshooting#channel-connected-messages-not-flowing](../gateway/troubleshooting.md#channel-connected-messages-not-flowing)
-   [/channels/troubleshooting](../channels/troubleshooting.md)

```bash
openclaw status
openclaw gateway status
openclaw cron status
openclaw cron list
openclaw cron runs --id <jobId> --limit 20
openclaw logs --follow
```

La buena salida se ve así:

-   `cron.status` muestra habilitado con un próximo despertar.
-   `cron runs` muestra entradas `ok` recientes.
-   Heartbeat está habilitado y no fuera de las horas activas.

Firmas de registro comunes:

-   `cron: scheduler disabled; jobs will not run automatically` → cron está deshabilitado.
-   `heartbeat skipped` con `reason=quiet-hours` → fuera de las horas activas configuradas.
-   `requests-in-flight` → carril principal ocupado; el despertar del heartbeat se pospuso.
-   `unknown accountId` → la cuenta objetivo de entrega del heartbeat no existe.

Páginas profundas:

-   [/gateway/troubleshooting#cron-and-heartbeat-delivery](../gateway/troubleshooting.md#cron-and-heartbeat-delivery)
-   [/automation/troubleshooting](../automation/troubleshooting.md)
-   [/gateway/heartbeat](../gateway/heartbeat.md)

```bash
openclaw status
openclaw gateway status
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw logs --follow
```

La buena salida se ve así:

-   El nodo aparece como conectado y emparejado para el rol `node`.
-   Existe la capacidad para el comando que estás invocando.
-   El estado de permiso está concedido para la herramienta.

Firmas de registro comunes:

-   `NODE_BACKGROUND_UNAVAILABLE` → lleva la aplicación del nodo al primer plano.
-   `*_PERMISSION_REQUIRED` → el permiso del sistema operativo fue denegado/faltante.
-   `SYSTEM_RUN_DENIED: approval required` → la aprobación de ejecución está pendiente.
-   `SYSTEM_RUN_DENIED: allowlist miss` → el comando no está en la lista blanca de ejecución.

Páginas profundas:

-   [/gateway/troubleshooting#node-paired-tool-fails](../gateway/troubleshooting.md#node-paired-tool-fails)
-   [/nodes/troubleshooting](../nodes/troubleshooting.md)
-   [/tools/exec-approvals](../tools/exec-approvals.md)

```bash
openclaw status
openclaw gateway status
openclaw browser status
openclaw logs --follow
openclaw doctor
```

La buena salida se ve así:

-   El estado del navegador muestra `running: true` y un perfil/navegador elegido.
-   El perfil `openclaw` inicia o el relé `chrome` tiene una pestaña adjunta.

Firmas de registro comunes:

-   `Failed to start Chrome CDP on port` → falló el lanzamiento del navegador local.
-   `browser.executablePath not found` → la ruta del binario configurada es incorrecta.
-   `Chrome extension relay is running, but no tab is connected` → la extensión no está adjunta.
-   `Browser attachOnly is enabled ... not reachable` → el perfil de solo adjuntar no tiene un objetivo CDP activo.

Páginas profundas:

-   [/gateway/troubleshooting#browser-tool-fails](../gateway/troubleshooting.md#browser-tool-fails)
-   [/tools/browser-linux-troubleshooting](../tools/browser-linux-troubleshooting.md)
-   [/tools/chrome-extension](../tools/chrome-extension.md)

[Ayuda](../help.md)[Preguntas Frecuentes](./faq.md)