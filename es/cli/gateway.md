

  Comandos CLI

  
# gateway

El Gateway es el servidor WebSocket de OpenClaw (canales, nodos, sesiones, hooks). Los subcomandos en esta página se encuentran bajo `openclaw gateway …`. Documentación relacionada:

-   [/gateway/bonjour](../gateway/bonjour.md)
-   [/gateway/discovery](../gateway/discovery.md)
-   [/gateway/configuration](../gateway/configuration.md)

## Ejecutar el Gateway

Ejecuta un proceso Gateway local:

```bash
openclaw gateway
```

Alias en primer plano:

```bash
openclaw gateway run
```

Notas:

-   Por defecto, el Gateway se niega a iniciar a menos que `gateway.mode=local` esté configurado en `~/.openclaw/openclaw.json`. Usa `--allow-unconfigured` para ejecuciones ad-hoc/de desarrollo.
-   Enlazar más allá del loopback sin autenticación está bloqueado (medida de seguridad).
-   `SIGUSR1` desencadena un reinicio dentro del proceso cuando está autorizado (`commands.restart` está habilitado por defecto; establece `commands.restart: false` para bloquear el reinicio manual, mientras que la aplicación/actualización de la herramienta/configuración del gateway siguen permitidas).
-   Los manejadores `SIGINT`/`SIGTERM` detienen el proceso del gateway, pero no restauran ningún estado personalizado de la terminal. Si envuelves la CLI con una TUI o entrada en modo raw, restaura la terminal antes de salir.

### Opciones

-   `--port `: Puerto WebSocket (el valor por defecto viene de la configuración/entorno; usualmente `18789`).
-   `--bind <loopback|lan|tailnet|auto|custom>`: modo de enlace del listener.
-   `--auth <token|password>`: anulación del modo de autenticación.
-   `--token `: anulación del token (también establece `OPENCLAW_GATEWAY_TOKEN` para el proceso).
-   `--password `: anulación de la contraseña (también establece `OPENCLAW_GATEWAY_PASSWORD` para el proceso).
-   `--tailscale <off|serve|funnel>`: expone el Gateway a través de Tailscale.
-   `--tailscale-reset-on-exit`: restablece la configuración de serve/funnel de Tailscale al apagar.
-   `--allow-unconfigured`: permite que el gateway inicie sin `gateway.mode=local` en la configuración.
-   `--dev`: crea una configuración de desarrollo + espacio de trabajo si faltan (omite BOOTSTRAP.md).
-   `--reset`: restablece la configuración de desarrollo + credenciales + sesiones + espacio de trabajo (requiere `--dev`).
-   `--force`: mata cualquier listener existente en el puerto seleccionado antes de iniciar.
-   `--verbose`: registros detallados.
-   `--claude-cli-logs`: solo muestra los registros de claude-cli en la consola (y habilita su stdout/stderr).
-   `--ws-log <auto|full|compact>`: estilo de registro de websocket (por defecto `auto`).
-   `--compact`: alias para `--ws-log compact`.
-   `--raw-stream`: registra eventos crudos del flujo del modelo en jsonl.
-   `--raw-stream-path `: ruta del archivo jsonl del flujo crudo.

## Consultar un Gateway en ejecución

Todos los comandos de consulta usan RPC WebSocket. Modos de salida:

-   Por defecto: legible para humanos (coloreado en TTY).
-   `--json`: JSON legible por máquina (sin estilo/animación).
-   `--no-color` (o `NO_COLOR=1`): deshabilita ANSI manteniendo el diseño humano.

Opciones compartidas (donde se admiten):

-   `--url `: URL WebSocket del Gateway.
-   `--token `: Token del Gateway.
-   `--password `: Contraseña del Gateway.
-   `--timeout `: tiempo de espera/presupuesto (varía por comando).
-   `--expect-final`: espera una respuesta "final" (llamadas de agente).

Nota: cuando estableces `--url`, la CLI no recurre a credenciales de configuración o entorno. Pasa `--token` o `--password` explícitamente. La falta de credenciales explícitas es un error.

### gateway health

```bash
openclaw gateway health --url ws://127.0.0.1:18789
```

### gateway status

`gateway status` muestra el servicio Gateway (launchd/systemd/schtasks) más una sonda RPC opcional.

```bash
openclaw gateway status
openclaw gateway status --json
```

Opciones:

-   `--url `: anula la URL de la sonda.
-   `--token `: autenticación por token para la sonda.
-   `--password `: autenticación por contraseña para la sonda.
-   `--timeout `: tiempo de espera de la sonda (por defecto `10000`).
-   `--no-probe`: omite la sonda RPC (solo vista del servicio).
-   `--deep`: escanea también servicios a nivel del sistema.

Notas:

-   `gateway status` resuelve las SecretRefs de autenticación configuradas para la autenticación de la sonda cuando es posible.
-   Si una SecretRef de autenticación requerida no está resuelta en esta ruta de comando, la autenticación de la sonda puede fallar; pasa `--token`/`--password` explícitamente o resuelve primero la fuente del secreto.

### gateway probe

`gateway probe` es el comando "depurar todo". Siempre sondea:

-   tu gateway remoto configurado (si está configurado), y
-   localhost (loopback) **incluso si hay un remoto configurado**.

Si se puede alcanzar múltiples gateways, los imprime todos. Se admiten múltiples gateways cuando usas perfiles/puertos aislados (ej., un bot de rescate), pero la mayoría de las instalaciones aún ejecutan un solo gateway.

```bash
openclaw gateway probe
openclaw gateway probe --json
```

#### Remoto sobre SSH (paridad con la app de Mac)

El modo "Remoto sobre SSH" de la aplicación macOS usa un reenvío de puerto local para que el gateway remoto (que puede estar enlazado solo a loopback) sea alcanzable en `ws://127.0.0.1:`. Equivalente en CLI:

```bash
openclaw gateway probe --ssh user@gateway-host
```

Opciones:

-   `--ssh `: `user@host` o `user@host:port` (el puerto por defecto es `22`).
-   `--ssh-identity `: archivo de identidad.
-   `--ssh-auto`: elige el primer host de gateway descubierto como objetivo SSH (solo LAN/WAB).

Configuración (opcional, usada como valores por defecto):

-   `gateway.remote.sshTarget`
-   `gateway.remote.sshIdentity`

### gateway call &lt;method&gt;

Ayudante de RPC de bajo nivel.

```bash
openclaw gateway call status
openclaw gateway call logs.tail --params '{"sinceMs": 60000}'
```

## Gestionar el servicio Gateway

```bash
openclaw gateway install
openclaw gateway start
openclaw gateway stop
openclaw gateway restart
openclaw gateway uninstall
```

Notas:

-   `gateway install` admite `--port`, `--runtime`, `--token`, `--force`, `--json`.
-   Cuando la autenticación por token requiere un token y `gateway.auth.token` está gestionado por SecretRef, `gateway install` valida que la SecretRef sea resoluble pero no persiste el token resuelto en los metadatos del entorno del servicio.
-   Si la autenticación por token requiere un token y la SecretRef del token configurado no está resuelta, la instalación falla cerrada en lugar de persistir texto plano de respaldo.
-   En modo de autenticación inferido, `OPENCLAW_GATEWAY_PASSWORD`/`CLAWDBOT_GATEWAY_PASSWORD` solo para shell no relaja los requisitos de token de instalación; usa configuración duradera (`gateway.auth.password` o `env` de configuración) al instalar un servicio gestionado.
-   Si tanto `gateway.auth.token` como `gateway.auth.password` están configurados y `gateway.auth.mode` no está establecido, la instalación se bloquea hasta que el modo se establece explícitamente.
-   Los comandos de ciclo de vida aceptan `--json` para scripting.

## Descubrir gateways (Bonjour)

`gateway discover` escanea en busca de balizas de Gateway (`_openclaw-gw._tcp`).

-   Multicast DNS-SD: `local.`
-   Unicast DNS-SD (Wide-Area Bonjour): elige un dominio (ejemplo: `openclaw.internal.`) y configura split DNS + un servidor DNS; consulta [/gateway/bonjour](../gateway/bonjour.md)

Solo los gateways con descubrimiento Bonjour habilitado (por defecto) anuncian la baliza. Los registros de descubrimiento de área amplia incluyen (TXT):

-   `role` (sugerencia del rol del gateway)
-   `transport` (sugerencia de transporte, ej. `gateway`)
-   `gatewayPort` (puerto WebSocket, usualmente `18789`)
-   `sshPort` (puerto SSH; por defecto `22` si no está presente)
-   `tailnetDns` (nombre de host MagicDNS, cuando está disponible)
-   `gatewayTls` / `gatewayTlsSha256` (TLS habilitado + huella digital del certificado)
-   `cliPath` (sugerencia opcional para instalaciones remotas)

### gateway discover

```bash
openclaw gateway discover
```

Opciones:

-   `--timeout `: tiempo de espera por comando (navegar/resolver); por defecto `2000`.
-   `--json`: salida legible por máquina (también deshabilita estilo/animación).

Ejemplos:

```bash
openclaw gateway discover --timeout 4000
openclaw gateway discover --json | jq '.beacons[].wsUrl'
```

[doctor](./doctor.md)[health](./health.md)

---