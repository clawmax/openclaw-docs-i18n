

  Seguridad y sandboxing

  
# Seguridad

> \[!WARNING\] **Modelo de confianza de asistente personal:** esta guía asume un límite de operador confiable por gateway (modelo de usuario único/asistente personal). OpenClaw **no** es un límite de seguridad multiinquilino hostil para múltiples usuarios adversarios compartiendo un agente/gateway. Si necesitas operación de confianza mixta o con usuarios adversarios, divide los límites de confianza (gateway separado + credenciales, idealmente usuarios/hosts de SO separados).

## Primero el alcance: modelo de seguridad de asistente personal

La guía de seguridad de OpenClaw asume un despliegue de **asistente personal**: un límite de operador confiable, potencialmente muchos agentes.

-   Postura de seguridad soportada: un usuario/límite de confianza por gateway (preferible un usuario/host/VPS de SO por límite).
-   No es un límite de seguridad soportado: un gateway/agente compartido usado por usuarios mutuamente no confiables o adversarios.
-   Si se requiere aislamiento de usuarios adversarios, divide por límite de confianza (gateway separado + credenciales, e idealmente usuarios/hosts de SO separados).
-   Si múltiples usuarios no confiables pueden enviar mensajes a un agente con herramientas habilitadas, trátalos como compartiendo la misma autoridad de herramientas delegada para ese agente.

Esta página explica el endurecimiento **dentro de ese modelo**. No afirma aislamiento multiinquilino hostil en un gateway compartido.

## Comprobación rápida: auditoría de seguridad de openclaw

Ver también: [Verificación Formal (Modelos de Seguridad)](../security/formal-verification.md) Ejecuta esto regularmente (especialmente después de cambiar la configuración o exponer superficies de red):

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
openclaw security audit --json
```

Marca errores comunes (exposición de autenticación del Gateway, exposición de control del navegador, listas de permitidos elevadas, permisos del sistema de archivos). OpenClaw es tanto un producto como un experimento: estás conectando el comportamiento de modelos de frontera en superficies de mensajería reales y herramientas reales. **No hay una configuración "perfectamente segura".** El objetivo es ser deliberado sobre:

-   quién puede hablar con tu bot
-   dónde se permite que el bot actúe
-   qué puede tocar el bot

Comienza con el acceso más pequeño que aún funcione, luego amplíalo a medida que ganes confianza.

## Supuesto de despliegue (importante)

OpenClaw asume que el host y el límite de configuración son confiables:

-   Si alguien puede modificar el estado/configuración del host del Gateway (`~/.openclaw`, incluyendo `openclaw.json`), trátalo como un operador confiable.
-   Ejecutar un Gateway para múltiples operadores mutuamente no confiables/adversarios **no es una configuración recomendada**.
-   Para equipos de confianza mixta, divide los límites de confianza con gateways separados (o como mínimo usuarios/hosts de SO separados).
-   OpenClaw puede ejecutar múltiples instancias de gateway en una máquina, pero las operaciones recomendadas favorecen una separación limpia de límites de confianza.
-   Configuración por defecto recomendada: un usuario por máquina/host (o VPS), un gateway para ese usuario, y uno o más agentes en ese gateway.
-   Si múltiples usuarios quieren OpenClaw, usa un VPS/host por usuario.

### Consecuencia práctica (límite de confianza del operador)

Dentro de una instancia de Gateway, el acceso de operador autenticado es un rol de plano de control confiable, no un rol de inquilino por usuario.

-   Los operadores con acceso de lectura/plano de control pueden inspeccionar metadatos/historial de sesión del gateway por diseño.
-   Los identificadores de sesión (`sessionKey`, IDs de sesión, etiquetas) son selectores de enrutamiento, no tokens de autorización.
-   Ejemplo: esperar aislamiento por operador para métodos como `sessions.list`, `sessions.preview`, o `chat.history` está fuera de este modelo.
-   Si necesitas aislamiento de usuarios adversarios, ejecuta gateways separados por límite de confianza.
-   Múltiples gateways en una máquina son técnicamente posibles, pero no es la línea base recomendada para aislamiento multi-usuario.

## Modelo de asistente personal (no es un bus multiinquilino)

OpenClaw está diseñado como un modelo de seguridad de asistente personal: un límite de operador confiable, potencialmente muchos agentes.

-   Si varias personas pueden enviar mensajes a un agente con herramientas habilitadas, cada una de ellas puede dirigir ese mismo conjunto de permisos.
-   El aislamiento de sesión/memoria por usuario ayuda a la privacidad, pero no convierte a un agente compartido en autorización de host por usuario.
-   Si los usuarios pueden ser adversarios entre sí, ejecuta gateways separados (o usuarios/hosts de SO separados) por límite de confianza.

### Espacio de trabajo de Slack compartido: riesgo real

Si "todos en Slack pueden enviar mensajes al bot", el riesgo central es la autoridad de herramientas delegada:

-   cualquier remitente permitido puede inducir llamadas a herramientas (`exec`, navegador, herramientas de red/archivos) dentro de la política del agente;
-   la inyección de contenido/prompt de un remitente puede causar acciones que afecten el estado compartido, dispositivos o salidas;
-   si un agente compartido tiene credenciales/archivos sensibles, cualquier remitente permitido puede potencialmente impulsar la exfiltración mediante el uso de herramientas.

Usa agentes/gateways separados con herramientas mínimas para flujos de trabajo de equipo; mantén los agentes de datos personales privados.

### Agente compartido en una empresa: patrón aceptable

Esto es aceptable cuando todos los que usan ese agente están en el mismo límite de confianza (por ejemplo, un equipo de empresa) y el agente está estrictamente limitado al ámbito empresarial.

-   ejecútalo en una máquina/VM/contenedor dedicado;
-   usa un usuario de SO dedicado + navegador/perfil/cuentas dedicados para ese tiempo de ejecución;
-   no inicies sesión en ese tiempo de ejecución en cuentas personales de Apple/Google o perfiles personales de navegador/gestor de contraseñas.

Si mezclas identidades personales y empresariales en el mismo tiempo de ejecución, colapsas la separación y aumentas el riesgo de exposición de datos personales.

## Concepto de confianza del Gateway y nodo

Trata al Gateway y al nodo como un dominio de confianza de operador, con roles diferentes:

-   **Gateway** es el plano de control y superficie de política (`gateway.auth`, política de herramientas, enrutamiento).
-   **Nodo** es la superficie de ejecución remota emparejada con ese Gateway (comandos, acciones de dispositivo, capacidades locales del host).
-   Un llamador autenticado en el Gateway es confiable en el alcance del Gateway. Después del emparejamiento, las acciones del nodo son acciones de operador confiables en ese nodo.
-   `sessionKey` es selección de enrutamiento/contexto, no autenticación por usuario.
-   Las aprobaciones de ejecución (lista de permitidos + preguntar) son barreras de seguridad para la intención del operador, no aislamiento multiinquilino hostil.

Si necesitas aislamiento de usuarios hostiles, divide los límites de confianza por usuario/host de SO y ejecuta gateways separados.

## Matriz de límites de confianza

Usa esto como el modelo rápido al evaluar riesgos:

| Límite o control | Qué significa | Lectura errónea común |
| --- | --- | --- |
| `gateway.auth` (token/contraseña/autenticación de dispositivo) | Autentica a los llamadores a las APIs del gateway | "Necesita firmas por mensaje en cada trama para ser seguro" |
| `sessionKey` | Clave de enrutamiento para selección de contexto/sesión | "La clave de sesión es un límite de autenticación de usuario" |
| Barreras de seguridad de prompt/contenido | Reduce el riesgo de abuso del modelo | "La inyección de prompt por sí sola prueba una omisión de autenticación" |
| `canvas.eval` / evaluación del navegador | Capacidad intencional del operador cuando está habilitada | "Cualquier primitiva de evaluación JS es automáticamente una vulnerabilidad en este modelo de confianza" |
| Shell `!` de TUI local | Ejecución local explícita desencadenada por el operador | "El comando de conveniencia de shell local es inyección remota" |
| Emparejamiento de nodo y comandos de nodo | Ejecución remota a nivel de operador en dispositivos emparejados | "El control remoto de dispositivos debe tratarse como acceso de usuario no confiable por defecto" |

## No son vulnerabilidades por diseño

Estos patrones son comúnmente reportados y usualmente se cierran sin acción a menos que se muestre una omisión real de límite:

-   Cadenas de solo inyección de prompt sin una omisión de política/autenticación/sandbox.
-   Afirmaciones que asumen operación multiinquilino hostil en un host/configuración compartido.
-   Afirmaciones que clasifican el acceso normal de lectura del operador (por ejemplo `sessions.list`/`sessions.preview`/`chat.history`) como IDOR en una configuración de gateway compartido.
-   Hallazgos de despliegue solo en localhost (por ejemplo HSTS en gateway solo de loopback).
-   Hallazgos de firma de webhook entrante de Discord para rutas entrantes que no existen en este repositorio.
-   Hallazgos de "falta de autorización por usuario" que tratan `sessionKey` como un token de autenticación.

## Lista de verificación previa para investigadores

Antes de abrir un GHSA, verifica todos estos puntos:

1.  La reproducción aún funciona en el último `main` o la última versión.
2.  El reporte incluye la ruta de código exacta (`archivo`, función, rango de líneas) y la versión/commit probada.
3.  El impacto cruza un límite de confianza documentado (no solo inyección de prompt).
4.  La afirmación no está listada en [Fuera de Alcance](https://github.com/openclaw/openclaw/blob/main/SECURITY.md#out-of-scope).
5.  Se verificaron los avisos existentes para duplicados (reutiliza el GHSA canónico cuando sea aplicable).
6.  Los supuestos de despliegue son explícitos (loopback/local vs expuesto, operadores confiables vs no confiables).

## Línea base endurecida en 60 segundos

Usa esta línea base primero, luego vuelve a habilitar herramientas selectivamente por agente confiable:

```json
{
  gateway: {
    mode: "local",
    bind: "loopback",
    auth: { mode: "token", token: "replace-with-long-random-token" },
  },
  session: {
    dmScope: "per-channel-peer",
  },
  tools: {
    profile: "messaging",
    deny: ["group:automation", "group:runtime", "group:fs", "sessions_spawn", "sessions_send"],
    fs: { workspaceOnly: true },
    exec: { security: "deny", ask: "always" },
    elevated: { enabled: false },
  },
  channels: {
    whatsapp: { dmPolicy: "pairing", groups: { "*": { requireMention: true } } },
  },
}
```

Esto mantiene el Gateway solo local, aísla los DMs y deshabilita las herramientas de plano de control/tiempo de ejecución por defecto.

## Regla rápida para bandeja de entrada compartida

Si más de una persona puede enviar DMs a tu bot:

-   Configura `session.dmScope: "per-channel-peer"` (o `"per-account-channel-peer"` para canales multi-cuenta).
-   Mantén `dmPolicy: "pairing"` o listas de permitidos estrictas.
-   Nunca combines DMs compartidos con acceso amplio a herramientas.
-   Esto endurece las bandejas de entrada cooperativas/compartidas, pero no está diseñado como aislamiento de co-inquilinos hostiles cuando los usuarios comparten acceso de escritura al host/configuración.

### Qué comprueba la auditoría (alto nivel)

-   **Acceso entrante** (políticas de DM, políticas de grupo, listas de permitidos): ¿pueden extraños activar el bot?
-   **Radio de explosión de herramientas** (herramientas elevadas + salas abiertas): ¿podría la inyección de prompt convertirse en acciones de shell/archivos/red?
-   **Exposición de red** (enlace/autenticación del Gateway, Tailscale Serve/Funnel, tokens de autenticación débiles/cortos).
-   **Exposición de control del navegador** (nodos remotos, puertos de relay, endpoints CDP remotos).
-   **Higiene de disco local** (permisos, enlaces simbólicos, inclusiones de configuración, rutas de "carpeta sincronizada").
-   **Plugins** (existen extensiones sin una lista de permitidos explícita).
-   **Deriva/mala configuración de política** (configuración de Docker sandbox configurada pero modo sandbox desactivado; patrones inefectivos de `gateway.nodes.denyCommands` porque la coincidencia es solo por nombre de comando exacto (por ejemplo `system.run`) y no inspecciona el texto del shell; entradas peligrosas de `gateway.nodes.allowCommands`; perfil global `tools.profile="minimal"` anulado por perfiles por agente; herramientas de plugins de extensión accesibles bajo política de herramientas permisiva).
-   **Deriva de expectativas de tiempo de ejecución** (por ejemplo `tools.exec.host="sandbox"` mientras el modo sandbox está desactivado, lo que ejecuta directamente en el host del gateway).
-   **Higiene de modelos** (advertir cuando los modelos configurados parecen legados; no es un bloqueo duro).

Si ejecutas `--deep`, OpenClaw también intenta una sonda en vivo del Gateway con el mejor esfuerzo.

## Mapa de almacenamiento de credenciales

Usa esto al auditar el acceso o decidir qué respaldar:

-   **WhatsApp**: `~/.openclaw/credentials/whatsapp//creds.json`
-   **Token de bot de Telegram**: config/env o `channels.telegram.tokenFile`
-   **Token de bot de Discord**: config/env o SecretRef (proveedores env/archivo/exec)
-   **Tokens de Slack**: config/env (`channels.slack.*`)
-   **Listas de permitidos de emparejamiento**:
    -   `~/.openclaw/credentials/-allowFrom.json` (cuenta por defecto)
    -   `~/.openclaw/credentials/--allowFrom.json` (cuentas no por defecto)
-   **Perfiles de autenticación de modelo**: `~/.openclaw/agents//agent/auth-profiles.json`
-   **Carga de secretos respaldada en archivo (opcional)**: `~/.openclaw/secrets.json`
-   **Importación OAuth legada**: `~/.openclaw/credentials/oauth.json`

## Lista de verificación de auditoría de seguridad

Cuando la auditoría imprime hallazgos, trata esto como un orden de prioridad:

1.  **Cualquier cosa "abierta" + herramientas habilitadas**: bloquea primero los DMs/grupos (emparejamiento/listas de permitidos), luego ajusta la política de herramientas/sandboxing.
2.  **Exposición de red pública** (enlace LAN, Funnel, autenticación faltante): corrige inmediatamente.
3.  **Exposición remota de control del navegador**: trátalo como acceso de operador (solo tailnet, empareja nodos deliberadamente, evita exposición pública).
4.  **Permisos**: asegúrate de que el estado/configuración/credenciales/autenticación no sean legibles por grupo/mundo.
5.  **Plugins/extensiones**: solo carga lo que confíes explícitamente.
6.  **Elección de modelo**: prefiere modelos modernos, endurecidos por instrucciones para cualquier bot con herramientas.

## Glosario de auditoría de seguridad

Valores `checkId` de alta señal que probablemente verás en despliegues reales (no exhaustivo):

| `checkId` | Severidad | Por qué importa | Clave/ruta de corrección primaria | Auto-corrección |
| --- | --- | --- | --- | --- |
| `fs.state_dir.perms_world_writable` | crítico | Otros usuarios/procesos pueden modificar el estado completo de OpenClaw | permisos del sistema de archivos en `~/.openclaw` | sí |
| `fs.config.perms_writable` | crítico | Otros pueden cambiar autenticación/política de herramientas/configuración | permisos del sistema de archivos en `~/.openclaw/openclaw.json` | sí |
| `fs.config.perms_world_readable` | crítico | La configuración puede exponer tokens/ajustes | permisos del sistema de archivos en el archivo de configuración | sí |
| `gateway.bind_no_auth` | crítico | Enlace remoto sin secreto compartido | `gateway.bind`, `gateway.auth.*` | no |
| `gateway.loopback_no_auth` | crítico | Loopback con proxy inverso puede volverse no autenticado | `gateway.auth.*`, configuración del proxy | no |
| `gateway.http.no_auth` | advertencia/crítico | APIs HTTP del Gateway accesibles con `auth.mode="none"` | `gateway.auth.mode`, `gateway.http.endpoints.*` | no |
| `gateway.tools_invoke_http.dangerous_allow` | advertencia/crítico | Rehabilita herramientas peligrosas sobre API HTTP | `gateway.tools.allow` | no |
| `gateway.nodes.allow_commands_dangerous` | advertencia/crítico | Habilita comandos de nodo de alto impacto (cámara/pantalla/contactos/calendario/SMS) | `gateway.nodes.allowCommands` | no |
| `gateway.tailscale_funnel` | crítico | Exposición a internet pública | `gateway.tailscale.mode` | no |
| `gateway.control_ui.allowed_origins_required` | crítico | Control UI no loopback sin lista de permitidos de origen del navegador explícita | `gateway.controlUi.allowedOrigins` | no |
| `gateway.control_ui.host_header_origin_fallback` | advertencia/crítico | Habilita la fallback de origen por cabecera Host (degradación del endurecimiento de rebinding DNS) | `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback` | no |
| `gateway.control_ui.insecure_auth` | advertencia | Interruptor de compatibilidad de autenticación insegura habilitado | `gateway.controlUi.allowInsecureAuth` | no |
| `gateway.control_ui.device_auth_disabled` | crítico | Desactiva la verificación de identidad del dispositivo | `gateway.controlUi.dangerouslyDisableDeviceAuth` | no |
| `gateway.real_ip_fallback_enabled` | advertencia/crítico | Confiar en la fallback `X-Real-IP` puede habilitar suplantación de IP de origen mediante mala configuración del proxy | `gateway.allowRealIpFallback`, `gateway.trustedProxies` | no |
| `discovery.mdns_full_mode` | advertencia/crítico | El modo completo mDNS anuncia metadatos `cliPath`/`sshPort` en la red local | `discovery.mdns.mode`, `gateway.bind` | no |
| `config.insecure_or_dangerous_flags` | advertencia | Cualquier bandera insegura/peligrosa de depuración habilitada | múltiples claves (ver detalle del hallazgo) | no |
| `hooks.token_too_short` | advertencia | Fuerza bruta más fácil en entrada de hooks | `hooks.token` | no |
| `hooks.request_session_key_enabled` | advertencia/crítico | Llamador externo puede elegir sessionKey | `hooks.allowRequestSessionKey` | no |
| `hooks.request_session_key_prefixes_missing` | advertencia/crítico | Sin límite en las formas de clave de sesión externa | `hooks.allowedSessionKeyPrefixes` | no |
| `logging.redact_off` | advertencia | Valores sensibles se filtran a registros/estado | `logging.redactSensitive` | sí |
| `sandbox.docker_config_mode_off` | advertencia | Configuración Docker sandbox presente pero inactiva | `agents.*.sandbox.mode` | no |
| `sandbox.dangerous_network_mode` | crítico | Red Docker sandbox usa modo `host` o `container:*` de unión de espacio de nombres | `agents.*.sandbox.docker.network` | no |
| `tools.exec.host_sandbox_no_sandbox_defaults` | advertencia | `exec host=sandbox` se resuelve a ejecución en host cuando sandbox está desactivado | `tools.exec.host`, `agents.defaults.sandbox.mode` | no |
| `tools.exec.host_sandbox_no_sandbox_agents` | advertencia | `exec host=sandbox` por agente se resuelve a ejecución en host cuando sandbox está desactivado | `agents.list[].tools.exec.host`, `agents.list[].sandbox.mode` | no |
| `tools.exec.safe_bins_interpreter_unprofiled` | advertencia | Bins de intérprete/tiempo de ejecución en `safeBins` sin perfiles explícitos amplían el riesgo de ejecución | `tools.exec.safeBins`, `tools.exec.safeBinProfiles`, `agents.list[].tools.exec.*` | no |
| `skills.workspace.symlink_escape` | advertencia | `skills/**/SKILL.md` del espacio de trabajo se resuelve fuera de la raíz del espacio de trabajo (deriva de cadena de enlaces simbólicos) | estado del sistema de archivos `skills/**` del espacio de trabajo | no |
| `security.exposure.open_groups_with_elevated` | crítico | Grupos abiertos + herramientas elevadas crean rutas de inyección de prompt de alto impacto | `channels.*.groupPolicy`, `tools.elevated.*` | no |
| `security.exposure.open_groups_with_runtime_or_fs` | crítico/advertencia | Grupos abiertos pueden alcanzar herramientas de comando/archivos sin guardias de sandbox/espacio de trabajo | `channels.*.groupPolicy`, `tools.profile/deny`, `tools.fs.workspaceOnly`, `agents.*.sandbox.mode` | no |
| `security.trust_model.multi_user_heuristic` | advertencia | La configuración parece multi-usuario mientras el modelo de confianza del gateway es de asistente personal | divide límites de confianza, o endurecimiento de usuario compartido (`sandbox.mode`, alcance de denegación/herramientas de espacio de trabajo) | no |
| `tools.profile_minimal_overridden` | advertencia | Las anulaciones del agente omiten el perfil mínimo global | `agents.list[].tools.profile` | no |
| `plugins.tools_reachable_permissive_policy` | advertencia | Herramientas de extensión accesibles en contextos permisivos | `tools.profile` + permitir/denegar herramientas | no |
| `models.small_params` | crítico/info | Modelos pequeños + superficies de herramientas no seguras aumentan el riesgo de inyección | elección de modelo + política de herramientas/sandbox | no |

## Control UI sobre HTTP

El Control UI necesita un **contexto seguro** (HTTPS o localhost) para generar identidad del dispositivo. `gateway.controlUi.allowInsecureAuth` **no** omite las comprobaciones de contexto-seguro, identidad-del-dispositivo o emparejamiento-de-dispositivo. Prefiere HTTPS (Tailscale Serve) o abre la UI en `127.0.0.1`. Solo para escenarios de emergencia, `gateway.controlUi.dangerouslyDisableDeviceAuth` desactiva las comprobaciones de identidad del dispositivo por completo. Esta es una degradación severa de seguridad; manténlo desactivado a menos que estés depurando activamente y puedas revertir rápidamente. `openclaw security audit` advierte cuando esta configuración está habilitada.

## Resumen de banderas inseguras o peligrosas

`openclaw security audit` incluye `config.insecure_or_dangerous_flags` cuando se habilitan interruptores de depuración inseguros/peligrosos conocidos. Esa comprobación actualmente agrega:

-   `gateway.controlUi.allowInsecureAuth=true`
-   `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback=true`
-   `gateway.controlUi.dangerouslyDisableDeviceAuth=true`
-   `hooks.gmail.allowUnsafeExternalContent=true`
-   `hooks.mappings[].allowUnsafeExternalContent=true`
-   `tools.exec.applyPatch.workspaceOnly=false`

Claves de configuración `dangerous*` / `dangerously*` completas definidas en el esquema de configuración de OpenClaw:

-   `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback`
-   `gateway.controlUi.dangerouslyDisableDeviceAuth`
-   `browser.ssrfPolicy.dangerouslyAllowPrivateNetwork`
-   `channels.discord.dangerouslyAllowNameMatching`
-   `channels.discord.accounts..dangerouslyAllowNameMatching`
-   `channels.slack.dangerouslyAllowNameMatching`
-   `channels.slack.accounts..dangerouslyAllowNameMatching`
-   `channels.googlechat.dangerouslyAllowNameMatching`
-   `channels.googlechat.accounts..dangerouslyAllowNameMatching`
-   `channels.msteams.dangerouslyAllowNameMatching`
-   `channels.irc.dangerouslyAllowNameMatching` (canal de extensión)
-   `channels.irc.accounts..dangerouslyAllowNameMatching` (canal de extensión)
-   `channels.mattermost.dangerouslyAllowNameMatching` (canal de extensión)
-   `channels.mattermost.accounts..dangerouslyAllowNameMatching` (canal de extensión)
-   `agents.defaults.sandbox.docker.dangerouslyAllowReservedContainerTargets`
-   `agents.defaults.sandbox.docker.dangerouslyAllowExternalBindSources`
-   `agents.defaults.sandbox.docker.dangerouslyAllowContainerNamespaceJoin`
-   `agents.list[].sandbox.docker.dangerouslyAllowReservedContainerTargets`
-   `agents.list[].sandbox.docker.dangerouslyAllowExternalBindSources`
-   `agents.list[].sandbox.docker.dangerouslyAllowContainerNamespaceJoin`

## Configuración de Proxy Inverso

Si ejecutas el Gateway detrás de un proxy inverso (nginx, Caddy, Traefik, etc.), debes configurar `gateway.trustedProxies` para una detección adecuada de la IP del cliente. Cuando el Gateway detecta cabeceras de proxy desde una dirección que **no** está en `trustedProxies`, **no** tratará las conexiones como clientes locales. Si la autenticación del gateway está deshabilitada, esas conexiones son rechazadas. Esto evita la omisión de autenticación donde las conexiones con proxy aparecerían de otro modo como provenientes de localhost y recibirían confianza automática.

```
gateway:
  trustedProxies:
    - "127.0.0.1" # si tu proxy se ejecuta en localhost
  # Opcional. Por defecto false.
  # Solo habilita si tu proxy no puede proporcionar X-Forwarded-For.
  allowRealIpFallback: false
  auth:
    mode: password
    password: ${OPENCLAW_GATEWAY_PASSWORD}
```

Cuando `trustedProxies` está configurado, el Gateway usa `X-Forwarded-For` para determinar la IP del cliente. `X-Real-IP` es ignorado por defecto a menos que `gateway.allowRealIpFallback: true` esté explícitamente configurado. Comportamiento bueno del proxy inverso (sobrescribir cabeceras de reenvío entrantes):

```bash
proxy_set_header X-Forwarded-For $remote_addr;
proxy_set_header X-Real-IP $remote_addr;
```

Comportamiento malo del proxy inverso (añadir/preservar cabeceras de reenvío no confiables):

```bash
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```

## Notas sobre HSTS y origen

-   El gateway de OpenClaw es primero local/loopback. Si terminas TLS en un proxy inverso, configura HSTS en el dominio HTTPS que enfrenta al proxy allí.
-   Si el gateway mismo termina HTTPS, puedes configurar `gateway.http.securityHeaders.strictTransportSecurity` para emitir la cabecera HSTS desde las respuestas de OpenClaw.
-   La guía detallada de despliegue está en [Autenticación de Proxy Confiable](./trusted-proxy-auth.md#tls-termination-and-hsts).
-   Para despliegues de Control UI no loopback, `gateway.controlUi.allowedOrigins` es requerido por defecto.
-   `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback=true` habilita el modo de fallback de origen por cabecera Host; trátalo como una política peligrosa seleccionada por el operador.
-   Trata el comportamiento de rebinding DNS y cabecera de host del proxy como preocupaciones de endurecimiento de despliegue; mantén `trustedProxies` ajustado y evita exponer el gateway directamente a internet pública.

## Los registros de sesión local viven en disco

OpenClaw almacena transcripciones de sesión en disco bajo `~/.openclaw/agents//sessions/*.jsonl`. Esto es requerido para la continuidad de sesión y (opcionalmente) la indexación de memoria de sesión, pero también significa **cualquier proceso/usuario con acceso al sistema de archivos puede leer esos registros**. Trata el acceso al disco como el límite de confianza y bloquea los permisos en `~/.openclaw` (ver la sección de auditoría abajo). Si necesitas un aislamiento más fuerte entre agentes, ejecútalos bajo usuarios de SO separados o hosts separados.

## Ejecución de nodo (system.run)

Si un nodo macOS está emparejado, el Gateway puede invocar `system.run` en ese nodo. Esto es **ejecución remota de código** en el Mac:

-   Requiere emparejamiento de nodo (aprobación + token).
-   Controlado en el Mac mediante **Configuración → Aprobaciones de ejecución** (seguridad + preguntar + lista de permitidos).
-   Si no quieres ejecución remota, configura la seguridad en **denegar** y elimina el emparejamiento de nodo para ese Mac.

## Habilidades dinámicas (observador / nodos remotos)

OpenClaw puede actualizar la lista de habilidades a mitad de sesión:

-   **Observador de habilidades**: los cambios a `SKILL.md` pueden actualizar la instantánea de habilidades en el siguiente turno del agente.
-   **Nodos remotos**: conectar un nodo macOS puede hacer que las habilidades solo para macOS sean elegibles (basado en sondeo de bins).

Trata las carpetas de habilidades como **código confiable** y restringe quién puede modificarlas.

## El Modelo de Amenazas

Tu asistente de IA puede:

-   Ejecutar comandos de shell arbitrarios
-   Leer/escribir archivos
-   Acceder a servicios de red
-   Enviar mensajes a cualquiera (si le das acceso a WhatsApp)

Las personas que te envían mensajes pueden:

-   Intentar engañar a tu IA para que haga cosas malas
-   Ingeniería social para acceder a tus datos
-   Sondearte para obtener detalles de infraestructura

## Concepto central: control de acceso antes que inteligencia

La mayoría de los fallos aquí no son exploits sofisticados — son "alguien envió un mensaje al bot y el bot hizo lo que pidió". La postura de OpenClaw:

-   **Identidad primero:** decide quién puede hablar con el bot (emparejamiento DM / listas de permitidos / "abierto" explícito).
-   **Alcance después