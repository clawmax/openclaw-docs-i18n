title: "Guía de Solución de Problemas y Soluciones para OpenClaw Gateway"
description: "Resuelve problemas comunes del gateway de OpenClaw: servicio no en ejecución, sin respuestas, errores 429, conectividad del panel de control y problemas de flujo de mensajes con comandos paso a paso."
keywords: ["solución de problemas openclaw", "gateway no en ejecución", "error 429 de anthropic", "conectividad del panel de control", "problemas de flujo de mensajes", "emparejamiento de canales", "latido cron", "falla de herramienta del navegador"]
---

  Configuración y operaciones

  
# Solución de Problemas

Esta página es el manual de procedimientos detallado. Comienza en [/help/troubleshooting](../help/troubleshooting.md) si quieres el flujo de triaje rápido primero.

## Escalera de comandos

Ejecuta estos primero, en este orden:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Señales saludables esperadas:

-   `openclaw gateway status` muestra `Runtime: running` y `RPC probe: ok`.
-   `openclaw doctor` no reporta problemas de configuración/servicio bloqueantes.
-   `openclaw channels status --probe` muestra canales conectados/listos.

## Uso extra de Anthropic 429 requerido para contexto largo

Usa esto cuando los registros/errores incluyan: `HTTP 429: rate_limit_error: Extra usage is required for long context requests`.

```bash
openclaw logs --follow
openclaw models status
openclaw config get agents.defaults.models
```

Busca:

-   El modelo Anthropic Opus/Sonnet seleccionado tiene `params.context1m: true`.
-   La credencial actual de Anthropic no es elegible para uso de contexto largo.
-   Las solicitudes fallan solo en sesiones largas/ejecuciones de modelo que necesitan la ruta beta de 1M.

Opciones de solución:

1.  Deshabilita `context1m` para ese modelo para volver a la ventana de contexto normal.
2.  Usa una clave API de Anthropic con facturación, o habilita Uso Extra de Anthropic en la cuenta de suscripción.
3.  Configura modelos de respaldo para que las ejecuciones continúen cuando las solicitudes de contexto largo de Anthropic sean rechazadas.

Relacionado:

-   [/providers/anthropic](../providers/anthropic.md)
-   [/reference/token-use](../reference/token-use.md)
-   [/help/faq#why-am-i-seeing-http-429-ratelimiterror-from-anthropic](../help/faq.md#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)

## Sin respuestas

Si los canales están activos pero nada responde, verifica el enrutamiento y la política antes de reconectar nada.

```bash
openclaw status
openclaw channels status --probe
openclaw pairing list --channel <channel> [--account <id>]
openclaw config get channels
openclaw logs --follow
```

Busca:

-   Emparejamiento pendiente para remitentes de DM.
-   Control de menciones en grupo (`requireMention`, `mentionPatterns`).
-   Desajustes en listas de permitidos de canal/grupo.

Firmas comunes:

-   `drop guild message (mention required` → mensaje de grupo ignorado hasta la mención.
-   `pairing request` → el remitente necesita aprobación.
-   `blocked` / `allowlist` → el remitente/canal fue filtrado por la política.

Relacionado:

-   [/channels/troubleshooting](../channels/troubleshooting.md)
-   [/channels/pairing](../channels/pairing.md)
-   [/channels/groups](../channels/groups.md)

## Conectividad de la interfaz de usuario de control del panel

Cuando el panel de control/interfaz de usuario de control no se conecta, valida la URL, el modo de autenticación y las suposiciones de contexto seguro.

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
openclaw doctor
openclaw gateway status --json
```

Busca:

-   URL de sondeo y URL del panel de control correctas.
-   Desajuste de modo de autenticación/token entre el cliente y el gateway.
-   Uso de HTTP donde se requiere identidad del dispositivo.

Firmas comunes:

-   `device identity required` → contexto no seguro o falta autenticación del dispositivo.
-   `device nonce required` / `device nonce mismatch` → el cliente no está completando el flujo de autenticación de dispositivo basado en desafío (`connect.challenge` + `device.nonce`).
-   `device signature invalid` / `device signature expired` → el cliente firmó el payload incorrecto (o marca de tiempo obsoleta) para el handshake actual.
-   `unauthorized` / bucle de reconexión → desajuste de token/contraseña.
-   `gateway connect failed:` → objetivo de host/puerto/url incorrecto.

Verificación de migración a autenticación de dispositivo v2:

```bash
openclaw --version
openclaw doctor
openclaw gateway status
```

Si los registros muestran errores de nonce/firma, actualiza el cliente que se conecta y verifica que:

1.  espere por `connect.challenge`
2.  firme el payload vinculado al desafío
3.  envíe `connect.params.device.nonce` con el mismo nonce del desafío

Relacionado:

-   [/web/control-ui](../web/control-ui.md)
-   [/gateway/authentication](./authentication.md)
-   [/gateway/remote](./remote.md)

## Servicio de gateway no en ejecución

Usa esto cuando el servicio está instalado pero el proceso no permanece activo.

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
openclaw doctor
openclaw gateway status --deep
```

Busca:

-   `Runtime: stopped` con indicios de salida.
-   Desajuste de configuración del servicio (`Config (cli)` vs `Config (service)`).
-   Conflictos de puerto/escucha.

Firmas comunes:

-   `Gateway start blocked: set gateway.mode=local` → el modo de gateway local no está habilitado. Solución: establece `gateway.mode="local"` en tu configuración (o ejecuta `openclaw configure`). Si estás ejecutando OpenClaw vía Podman usando el usuario dedicado `openclaw`, la configuración reside en `~openclaw/.openclaw/openclaw.json`.
-   `refusing to bind gateway ... without auth` → enlace no de bucle invertido sin token/contraseña.
-   `another gateway instance is already listening` / `EADDRINUSE` → conflicto de puerto.

Relacionado:

-   [/gateway/background-process](./background-process.md)
-   [/gateway/configuration](./configuration.md)
-   [/gateway/doctor](./doctor.md)

## Canal conectado pero mensajes no fluyen

Si el estado del canal es conectado pero el flujo de mensajes está muerto, enfócate en la política, permisos y reglas de entrega específicas del canal.

```bash
openclaw channels status --probe
openclaw pairing list --channel <channel> [--account <id>]
openclaw status --deep
openclaw logs --follow
openclaw config get channels
```

Busca:

-   Política de DM (`pairing`, `allowlist`, `open`, `disabled`).
-   Lista de permitidos de grupo y requisitos de mención.
-   Permisos/alcances de API de canal faltantes.

Firmas comunes:

-   `mention required` → mensaje ignorado por la política de mención de grupo.
-   `pairing` / rastros de aprobación pendiente → el remitente no está aprobado.
-   `missing_scope`, `not_in_channel`, `Forbidden`, `401/403` → problema de autenticación/permisos del canal.

Relacionado:

-   [/channels/troubleshooting](../channels/troubleshooting.md)
-   [/channels/whatsapp](../channels/whatsapp.md)
-   [/channels/telegram](../channels/telegram.md)
-   [/channels/discord](../channels/discord.md)

## Entrega de cron y latido

Si cron o el latido no se ejecutaron o no se entregaron, verifica primero el estado del programador, luego el objetivo de entrega.

```bash
openclaw cron status
openclaw cron list
openclaw cron runs --id <jobId> --limit 20
openclaw system heartbeat last
openclaw logs --follow
```

Busca:

-   Cron habilitado y próxima activación presente.
-   Estado del historial de ejecución de trabajos (`ok`, `skipped`, `error`).
-   Razones de omisión del latido (`quiet-hours`, `requests-in-flight`, `alerts-disabled`).

Firmas comunes:

-   `cron: scheduler disabled; jobs will not run automatically` → cron deshabilitado.
-   `cron: timer tick failed` → el tick del programador falló; verifica errores de archivo/registro/tiempo de ejecución.
-   `heartbeat skipped` con `reason=quiet-hours` → fuera de la ventana de horas activas.
-   `heartbeat: unknown accountId` → id de cuenta inválida para el objetivo de entrega del latido.
-   `heartbeat skipped` con `reason=dm-blocked` → el objetivo del latido se resolvió a un destino de tipo DM mientras `agents.defaults.heartbeat.directPolicy` (o la anulación por agente) está establecido en `block`.

Relacionado:

-   [/automation/troubleshooting](../automation/troubleshooting.md)
-   [/automation/cron-jobs](../automation/cron-jobs.md)
-   [/gateway/heartbeat](./heartbeat.md)

## Herramienta de nodo emparejado falla

Si un nodo está emparejado pero las herramientas fallan, aísla el estado de primer plano, permisos y aprobación.

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
openclaw logs --follow
openclaw status
```

Busca:

-   Nodo en línea con las capacidades esperadas.
-   Concesiones de permisos del SO para cámara/micrófono/ubicación/pantalla.
-   Aprobaciones de ejecución y estado de lista de permitidos.

Firmas comunes:

-   `NODE_BACKGROUND_UNAVAILABLE` → la aplicación del nodo debe estar en primer plano.
-   `*_PERMISSION_REQUIRED` / `LOCATION_PERMISSION_REQUIRED` → falta permiso del SO.
-   `SYSTEM_RUN_DENIED: approval required` → aprobación de ejecución pendiente.
-   `SYSTEM_RUN_DENIED: allowlist miss` → comando bloqueado por la lista de permitidos.

Relacionado:

-   [/nodes/troubleshooting](../nodes/troubleshooting.md)
-   [/nodes/index](../nodes/index.md)
-   [/tools/exec-approvals](../tools/exec-approvals.md)

## Herramienta del navegador falla

Usa esto cuando las acciones de la herramienta del navegador fallan aunque el gateway en sí esté saludable.

```bash
openclaw browser status
openclaw browser start --browser-profile openclaw
openclaw browser profiles
openclaw logs --follow
openclaw doctor
```

Busca:

-   Ruta ejecutable del navegador válida.
-   Accesibilidad del perfil CDP.
-   Conexión de pestaña de relé de extensión para `profile="chrome"`.

Firmas comunes:

-   `Failed to start Chrome CDP on port` → falló el lanzamiento del proceso del navegador.
-   `browser.executablePath not found` → la ruta configurada es inválida.
-   `Chrome extension relay is running, but no tab is connected` → el relé de extensión no está conectado.
-   `Browser attachOnly is enabled ... not reachable` → el perfil de solo conexión no tiene un objetivo accesible.

Relacionado:

-   [/tools/browser-linux-troubleshooting](../tools/browser-linux-troubleshooting.md)
-   [/tools/chrome-extension](../tools/chrome-extension.md)
-   [/tools/browser](../tools/browser.md)

## Si actualizaste y algo se rompió repentinamente

La mayoría de las rupturas posteriores a la actualización son desviaciones de configuración o valores predeterminados más estrictos que ahora se aplican.

### 1) El comportamiento de anulación de autenticación y URL cambió

```bash
openclaw gateway status
openclaw config get gateway.mode
openclaw config get gateway.remote.url
openclaw config get gateway.auth.mode
```

Qué verificar:

-   Si `gateway.mode=remote`, las llamadas CLI pueden estar apuntando a remoto mientras tu servicio local está bien.
-   Las llamadas explícitas `--url` no recurren a las credenciales almacenadas.

Firmas comunes:

-   `gateway connect failed:` → objetivo de URL incorrecto.
-   `unauthorized` → punto final accesible pero autenticación incorrecta.

### 2) Las salvaguardas de enlace y autenticación son más estrictas

```bash
openclaw config get gateway.bind
openclaw config get gateway.auth.token
openclaw gateway status
openclaw logs --follow
```

Qué verificar:

-   Los enlaces no de bucle invertido (`lan`, `tailnet`, `custom`) necesitan autenticación configurada.
-   Las claves antiguas como `gateway.token` no reemplazan `gateway.auth.token`.

Firmas comunes:

-   `refusing to bind gateway ... without auth` → desajuste de enlace+autenticación.
-   `RPC probe: failed` mientras el tiempo de ejecución está en marcha → gateway vivo pero inaccesible con la autenticación/url actual.

### 3) El estado de emparejamiento e identidad del dispositivo cambió

```bash
openclaw devices list
openclaw pairing list --channel <channel> [--account <id>]
openclaw logs --follow
openclaw doctor
```

Qué verificar:

-   Aprobaciones de dispositivos pendientes para el panel de control/nodos.
-   Aprobaciones de emparejamiento de DM pendientes después de cambios de política o identidad.

Firmas comunes:

-   `device identity required` → autenticación del dispositivo no satisfecha.
-   `pairing required` → el remitente/dispositivo debe ser aprobado.

Si la configuración del servicio y el tiempo de ejecución aún no concuerdan después de las verificaciones, reinstala los metadatos del servicio desde el mismo directorio de perfil/estado:

```bash
openclaw gateway install --force
openclaw gateway restart
```

Relacionado:

-   [/gateway/pairing](./pairing.md)
-   [/gateway/authentication](./authentication.md)
-   [/gateway/background-process](./background-process.md)

[Múltiples Gateways](./multiple-gateways.md)[Seguridad](./security.md)

---