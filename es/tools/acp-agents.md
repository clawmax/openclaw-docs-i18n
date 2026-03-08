

  Coordinación de agentes

  
# Agentes ACP

Las sesiones del [Protocolo de Agente Cliente (ACP)](https://agentclientprotocol.com/) permiten a OpenClaw ejecutar entornos de codificación externos (por ejemplo Pi, Claude Code, Codex, OpenCode y Gemini CLI) a través de un plugin backend ACP. Si le pides a OpenClaw en lenguaje natural que "ejecute esto en Codex" o "inicie Claude Code en un hilo", OpenClaw debería enrutar esa solicitud al tiempo de ejecución ACP (no al tiempo de ejecución nativo de sub-agentes).

## Flujo rápido para operadores

Usa esto cuando quieras un manual de ejecución `/acp` práctico:

1.  Generar una sesión:
    -   `/acp spawn codex --mode persistent --thread auto`
2.  Trabajar en el hilo vinculado (o apuntar explícitamente a esa clave de sesión).
3.  Verificar el estado del tiempo de ejecución:
    -   `/acp status`
4.  Ajustar las opciones de tiempo de ejecución según sea necesario:
    -   `/acp model <provider/model>`
    -   `/acp permissions `
    -   `/acp timeout `
5.  Guiar una sesión activa sin reemplazar el contexto:
    -   `/acp steer tighten logging and continue`
6.  Detener el trabajo:
    -   `/acp cancel` (detener el turno actual), o
    -   `/acp close` (cerrar sesión + eliminar vinculaciones)

## Inicio rápido para humanos

Ejemplos de solicitudes naturales:

-   "Inicia una sesión persistente de Codex en un hilo aquí y mantenla enfocada."
-   "Ejecuta esto como una sesión ACP única de Claude Code y resume el resultado."
-   "Usa Gemini CLI para esta tarea en un hilo, luego mantén los seguimientos en ese mismo hilo."

Lo que OpenClaw debería hacer:

1.  Elegir `runtime: "acp"`.
2.  Resolver el entorno objetivo solicitado (`agentId`, por ejemplo `codex`).
3.  Si se solicita vinculación de hilo y el canal actual lo admite, vincular la sesión ACP al hilo.
4.  Enrutar los mensajes de seguimiento en el hilo a esa misma sesión ACP hasta que se pierda el enfoque/se cierre/caduque.

## ACP versus sub-agentes

Usa ACP cuando quieras un entorno de ejecución externo. Usa sub-agentes cuando quieras ejecuciones delegadas nativas de OpenClaw.

| Área | Sesión ACP | Ejecución de sub-agente |
| --- | --- | --- |
| Tiempo de ejecución | Plugin backend ACP (por ejemplo acpx) | Tiempo de ejecución nativo de sub-agentes de OpenClaw |
| Clave de sesión | `agent::acp:` | `agent::subagent:` |
| Comandos principales | `/acp ...` | `/subagents ...` |
| Herramienta de generación | `sessions_spawn` con `runtime:"acp"` | `sessions_spawn` (tiempo de ejecución por defecto) |

Ver también [Sub-agentes](./subagents.md).

## Sesiones vinculadas a hilos (independientes del canal)

Cuando las vinculaciones de hilos están habilitadas para un adaptador de canal, las sesiones ACP pueden vincularse a hilos:

-   OpenClaw vincula un hilo a una sesión ACP objetivo.
-   Los mensajes de seguimiento en ese hilo se enrutan a la sesión ACP vinculada.
-   La salida ACP se entrega al mismo hilo.
-   La pérdida de enfoque/cierre/archivado/caducidad por inactividad o antigüedad máxima elimina la vinculación.

El soporte de vinculación de hilos es específico del adaptador. Si el adaptador de canal activo no admite vinculaciones de hilos, OpenClaw devuelve un mensaje claro de no admitido/no disponible. Banderas de características requeridas para ACP vinculado a hilos:

-   `acp.enabled=true`
-   `acp.dispatch.enabled` está activado por defecto (establecer `false` para pausar el despacho ACP)
-   Bandera de generación de hilos ACP del adaptador de canal habilitada (específica del adaptador)
    -   Discord: `channels.discord.threadBindings.spawnAcpSessions=true`
    -   Telegram: `channels.telegram.threadBindings.spawnAcpSessions=true`

### Canales que admiten hilos

-   Cualquier adaptador de canal que exponga capacidad de vinculación de sesión/hilo.
-   Soporte incorporado actual:
    -   Hilos/canales de Discord
    -   Temas de Telegram (temas de foro en grupos/supergrupos y temas de DM)
-   Los canales de plugin pueden agregar soporte a través de la misma interfaz de vinculación.

## Configuración específica del canal

Para flujos de trabajo no efímeros, configura vinculaciones ACP persistentes en entradas de nivel superior `bindings[]`.

### Modelo de vinculación

-   `bindings[].type="acp"` marca una vinculación de conversación ACP persistente.
-   `bindings[].match` identifica la conversación objetivo:
    -   Canal o hilo de Discord: `match.channel="discord"` + `match.peer.id=""`
    -   Tema de foro de Telegram: `match.channel="telegram"` + `match.peer.id=":topic:"`
-   `bindings[].agentId` es el id del agente OpenClaw propietario.
-   Las anulaciones ACP opcionales se encuentran bajo `bindings[].acp`:
    -   `mode` (`persistent` o `oneshot`)
    -   `label`
    -   `cwd`
    -   `backend`

### Valores predeterminados de tiempo de ejecución por agente

Usa `agents.list[].runtime` para definir valores predeterminados ACP una vez por agente:

-   `agents.list[].runtime.type="acp"`
-   `agents.list[].runtime.acp.agent` (id del entorno, por ejemplo `codex` o `claude`)
-   `agents.list[].runtime.acp.backend`
-   `agents.list[].runtime.acp.mode`
-   `agents.list[].runtime.acp.cwd`

Precedencia de anulación para sesiones ACP vinculadas:

1.  `bindings[].acp.*`
2.  `agents.list[].runtime.acp.*`
3.  Valores predeterminados globales de ACP (por ejemplo `acp.backend`)

Ejemplo:

```json
{
  agents: {
    list: [
      {
        id: "codex",
        runtime: {
          type: "acp",
          acp: {
            agent: "codex",
            backend: "acpx",
            mode: "persistent",
            cwd: "/workspace/openclaw",
          },
        },
      },
      {
        id: "claude",
        runtime: {
          type: "acp",
          acp: { agent: "claude", backend: "acpx", mode: "persistent" },
        },
      },
    ],
  },
  bindings: [
    {
      type: "acp",
      agentId: "codex",
      match: {
        channel: "discord",
        accountId: "default",
        peer: { kind: "channel", id: "222222222222222222" },
      },
      acp: { label: "codex-main" },
    },
    {
      type: "acp",
      agentId: "claude",
      match: {
        channel: "telegram",
        accountId: "default",
        peer: { kind: "group", id: "-1001234567890:topic:42" },
      },
      acp: { cwd: "/workspace/repo-b" },
    },
    {
      type: "route",
      agentId: "main",
      match: { channel: "discord", accountId: "default" },
    },
    {
      type: "route",
      agentId: "main",
      match: { channel: "telegram", accountId: "default" },
    },
  ],
  channels: {
    discord: {
      guilds: {
        "111111111111111111": {
          channels: {
            "222222222222222222": { requireMention: false },
          },
        },
      },
    },
    telegram: {
      groups: {
        "-1001234567890": {
          topics: { "42": { requireMention: false } },
        },
      },
    },
  },
}
```

Comportamiento:

-   OpenClaw asegura que la sesión ACP configurada exista antes de usarla.
-   Los mensajes en ese canal o tema se enrutan a la sesión ACP configurada.
-   En conversaciones vinculadas, `/new` y `/reset` reinician la misma clave de sesión ACP en su lugar.
-   Las vinculaciones de tiempo de ejecución temporales (por ejemplo creadas por flujos de enfoque de hilo) aún se aplican donde estén presentes.

## Iniciar sesiones ACP (interfaces)

### Desde sessions\_spawn

Usa `runtime: "acp"` para iniciar una sesión ACP desde un turno de agente o una llamada a herramienta.

```json
{
  "task": "Open the repo and summarize failing tests",
  "runtime": "acp",
  "agentId": "codex",
  "thread": true,
  "mode": "session"
}
```

Notas:

-   `runtime` por defecto es `subagent`, así que establece `runtime: "acp"` explícitamente para sesiones ACP.
-   Si se omite `agentId`, OpenClaw usa `acp.defaultAgent` cuando está configurado.
-   `mode: "session"` requiere `thread: true` para mantener una conversación vinculada persistente.

Detalles de la interfaz:

-   `task` (requerido): prompt inicial enviado a la sesión ACP.
-   `runtime` (requerido para ACP): debe ser `"acp"`.
-   `agentId` (opcional): id del entorno objetivo ACP. Recurre a `acp.defaultAgent` si está configurado.
-   `thread` (opcional, por defecto `false`): solicitar flujo de vinculación de hilo donde sea compatible.
-   `mode` (opcional): `run` (única) o `session` (persistente).
    -   el valor por defecto es `run`
    -   si `thread: true` y se omite mode, OpenClaw puede usar un comportamiento persistente predeterminado según la ruta de tiempo de ejecución
    -   `mode: "session"` requiere `thread: true`
-   `cwd` (opcional): directorio de trabajo de tiempo de ejecución solicitado (validado por política de backend/tiempo de ejecución).
-   `label` (opcional): etiqueta orientada al operador usada en texto de sesión/banner.
-   `streamTo` (opcional): `"parent"` transmite resúmenes de progreso de la ejecución ACP inicial de vuelta a la sesión del solicitante como eventos del sistema.
    -   Cuando está disponible, las respuestas aceptadas incluyen `streamLogPath` que apunta a un registro JSONL con alcance de sesión (`.acp-stream.jsonl`) que puedes seguir para el historial completo de retransmisión.

## Compatibilidad con sandbox

Las sesiones ACP actualmente se ejecutan en el tiempo de ejecución del host, no dentro del sandbox de OpenClaw. Limitaciones actuales:

-   Si la sesión del solicitante está en sandbox, los spawns ACP están bloqueados.
    -   Error: `Sandboxed sessions cannot spawn ACP sessions because runtime="acp" runs on the host. Use runtime="subagent" from sandboxed sessions.`
-   `sessions_spawn` con `runtime: "acp"` no admite `sandbox: "require"`.
    -   Error: `sessions_spawn sandbox="require" is unsupported for runtime="acp" because ACP sessions run outside the sandbox. Use runtime="subagent" or sandbox="inherit".`

Usa `runtime: "subagent"` cuando necesites ejecución forzada por sandbox.

### Desde el comando /acp

Usa `/acp spawn` para control explícito del operador desde el chat cuando sea necesario.

```bash
/acp spawn codex --mode persistent --thread auto
/acp spawn codex --mode oneshot --thread off
/acp spawn codex --thread here
```

Banderas clave:

-   `--mode persistent|oneshot`
-   `--thread auto|here|off`
-   `--cwd <absolute-path>`
-   `--label `

Ver [Comandos de barra](./slash-commands.md).

## Resolución del objetivo de sesión

La mayoría de las acciones `/acp` aceptan un objetivo de sesión opcional (`session-key`, `session-id`, o `session-label`). Orden de resolución:

1.  Argumento de objetivo explícito (o `--session` para `/acp steer`)
    -   intenta clave
    -   luego id de sesión con formato UUID
    -   luego etiqueta
2.  Vinculación de hilo actual (si esta conversación/hilo está vinculado a una sesión ACP)
3.  Recurso de sesión del solicitante actual

Si no se resuelve ningún objetivo, OpenClaw devuelve un error claro (`Unable to resolve session target: ...`).

## Modos de generación de hilos

`/acp spawn` admite `--thread auto|here|off`.

| Modo | Comportamiento |
| --- | --- |
| `auto` | En un hilo activo: vincular ese hilo. Fuera de un hilo: crear/vincular un hilo hijo cuando sea compatible. |
| `here` | Requerir hilo activo actual; fallar si no está en uno. |
| `off` | Sin vinculación. La sesión comienza sin vincular. |

Notas:

-   En superficies sin vinculación de hilos, el comportamiento predeterminado es efectivamente `off`.
-   La generación vinculada a hilos requiere soporte de política de canal:
    -   Discord: `channels.discord.threadBindings.spawnAcpSessions=true`
    -   Telegram: `channels.telegram.threadBindings.spawnAcpSessions=true`

## Controles ACP

Familia de comandos disponible:

-   `/acp spawn`
-   `/acp cancel`
-   `/acp steer`
-   `/acp close`
-   `/acp status`
-   `/acp set-mode`
-   `/acp set`
-   `/acp cwd`
-   `/acp permissions`
-   `/acp timeout`
-   `/acp model`
-   `/acp reset-options`
-   `/acp sessions`
-   `/acp doctor`
-   `/acp install`

`/acp status` muestra las opciones efectivas de tiempo de ejecución y, cuando están disponibles, tanto los identificadores de sesión a nivel de tiempo de ejecución como a nivel de backend. Algunos controles dependen de las capacidades del backend. Si un backend no admite un control, OpenClaw devuelve un error claro de control no admitido.

## Recetario de comandos ACP

| Comando | Qué hace | Ejemplo |
| --- | --- | --- |
| `/acp spawn` | Crear sesión ACP; vinculación de hilo opcional. | `/acp spawn codex --mode persistent --thread auto --cwd /repo` |
| `/acp cancel` | Cancelar turno en curso para la sesión objetivo. | `/acp cancel agent:codex:acp:` |
| `/acp steer` | Enviar instrucción de guía a la sesión en ejecución. | `/acp steer --session support inbox prioritize failing tests` |
| `/acp close` | Cerrar sesión y desvincular objetivos de hilo. | `/acp close` |
| `/acp status` | Mostrar backend, modo, estado, opciones de tiempo de ejecución, capacidades. | `/acp status` |
| `/acp set-mode` | Establecer modo de tiempo de ejecución para la sesión objetivo. | `/acp set-mode plan` |
| `/acp set` | Escritura genérica de opción de configuración de tiempo de ejecución. | `/acp set model openai/gpt-5.2` |
| `/acp cwd` | Establecer anulación del directorio de trabajo de tiempo de ejecución. | `/acp cwd /Users/user/Projects/repo` |
| `/acp permissions` | Establecer perfil de política de aprobación. | `/acp permissions strict` |
| `/acp timeout` | Establecer tiempo de espera de tiempo de ejecución (segundos). | `/acp timeout 120` |
| `/acp model` | Establecer anulación de modelo de tiempo de ejecución. | `/acp model anthropic/claude-opus-4-5` |
| `/acp reset-options` | Eliminar anulaciones de opciones de tiempo de ejecución de la sesión. | `/acp reset-options` |
| `/acp sessions` | Listar sesiones ACP recientes del almacén. | `/acp sessions` |
| `/acp doctor` | Salud del backend, capacidades, correcciones accionables. | `/acp doctor` |
| `/acp install` | Imprimir pasos de instalación y habilitación deterministas. | `/acp install` |

## Mapeo de opciones de tiempo de ejecución

`/acp` tiene comandos de conveniencia y un establecedor genérico. Operaciones equivalentes:

-   `/acp model ` se mapea a la clave de configuración de tiempo de ejecución `model`.
-   `/acp permissions ` se mapea a la clave de configuración de tiempo de ejecución `approval_policy`.
-   `/acp timeout ` se mapea a la clave de configuración de tiempo de ejecución `timeout`.
-   `/acp cwd ` actualiza la anulación de cwd de tiempo de ejecución directamente.
-   `/acp set  ` es la ruta genérica.
    -   Caso especial: `key=cwd` usa la ruta de anulación de cwd.
-   `/acp reset-options` borra todas las anulaciones de tiempo de ejecución para la sesión objetivo.

## Soporte de entorno acpx (actual)

Alias de entorno incorporados actuales de acpx:

-   `pi`
-   `claude`
-   `codex`
-   `opencode`
-   `gemini`
-   `kimi`

Cuando OpenClaw usa el backend acpx, prefiere estos valores para `agentId` a menos que tu configuración de acpx defina alias de agente personalizados. El uso directo de la CLI acpx también puede apuntar a adaptadores arbitrarios a través de `--agent `, pero esa escotilla de escape cruda es una característica de la CLI acpx (no la ruta normal de `agentId` de OpenClaw).

## Configuración requerida

Línea base ACP central:

```json
{
  acp: {
    enabled: true,
    // Opcional. Por defecto es true; establecer false para pausar el despacho ACP manteniendo los controles /acp.
    dispatch: { enabled: true },
    backend: "acpx",
    defaultAgent: "codex",
    allowedAgents: ["pi", "claude", "codex", "opencode", "gemini", "kimi"],
    maxConcurrentSessions: 8,
    stream: {
      coalesceIdleMs: 300,
      maxChunkChars: 1200,
    },
    runtime: {
      ttlMinutes: 120,
    },
  },
}
```

La configuración de vinculación de hilos es específica del adaptador de canal. Ejemplo para Discord:

```json
{
  session: {
    threadBindings: {
      enabled: true,
      idleHours: 24,
      maxAgeHours: 0,
    },
  },
  channels: {
    discord: {
      threadBindings: {
        enabled: true,
        spawnAcpSessions: true,
      },
    },
  },
}
```

Si la generación ACP vinculada a hilos no funciona, verifica primero la bandera de característica del adaptador:

-   Discord: `channels.discord.threadBindings.spawnAcpSessions=true`

Ver [Referencia de Configuración](../gateway/configuration-reference.md).

## Configuración del plugin para backend acpx

Instalar y habilitar plugin:

```bash
openclaw plugins install acpx
openclaw config set plugins.entries.acpx.enabled true
```

Instalación local en espacio de trabajo durante desarrollo:

```bash
openclaw plugins install ./extensions/acpx
```

Luego verificar la salud del backend:

```bash
/acp doctor
```

### Configuración de comando y versión de acpx

Por defecto, el plugin acpx (publicado como `@openclaw/acpx`) usa el binario fijado local al plugin:

1.  El comando por defecto es `extensions/acpx/node_modules/.bin/acpx`.
2.  La versión esperada por defecto es la fijación de la extensión.
3.  El inicio registra el backend ACP inmediatamente como no listo.
4.  Un trabajo de verificación en segundo plano verifica `acpx --version`.
5.  Si falta el binario local al plugin o no coincide, ejecuta: `npm install --omit=dev --no-save acpx@` y vuelve a verificar.

Puedes anular comando/versión en la configuración del plugin:

```json
{
  "plugins": {
    "entries": {
      "acpx": {
        "enabled": true,
        "config": {
          "command": "../acpx/dist/cli.js",
          "expectedVersion": "any"
        }
      }
    }
  }
}
```

Notas:

-   `command` acepta una ruta absoluta, ruta relativa o nombre de comando (`acpx`).
-   Las rutas relativas se resuelven desde el directorio del espacio de trabajo de OpenClaw.
-   `expectedVersion: "any"` desactiva la coincidencia estricta de versiones.
-   Cuando `command` apunta a un binario/ruta personalizado, la auto-instalación local al plugin está deshabilitada.
-   El inicio de OpenClaw permanece no bloqueante mientras se ejecuta la verificación de salud del backend.

Ver [Plugins](./plugin.md).

## Configuración de permisos

Las sesiones ACP se ejecutan de forma no interactiva — no hay TTY para aprobar o denegar solicitudes de permisos de escritura de archivos y ejecución de shell. El plugin acpx proporciona dos claves de configuración que controlan cómo se manejan los permisos:

### permissionMode

Controla qué operaciones puede realizar el agente del entorno sin solicitar confirmación.

| Valor | Comportamiento |
| --- | --- |
| `approve-all` | Aprobar automáticamente todas las escrituras de archivos y comandos de shell. |
| `approve-reads` | Aprobar automáticamente solo lecturas; escrituras y ejecuciones requieren solicitudes. |
| `deny-all` | Denegar todas las solicitudes de permiso. |

### nonInteractivePermissions

Controla qué sucede cuando se mostraría una solicitud de permiso pero no hay TTY interactivo disponible (que siempre es el caso para sesiones ACP).

| Valor | Comportamiento |
| --- | --- |
| `fail` | Abortar la sesión con `AcpRuntimeError`. **(por defecto)** |
| `deny` | Denegar silenciosamente el permiso y continuar (degradación elegante). |

### Configuración

Establecer a través de la configuración del plugin:

```bash
openclaw config set plugins.entries.acpx.config.permissionMode approve-all
openclaw config set plugins.entries.acpx.config.nonInteractivePermissions fail
```

Reiniciar la puerta de enlace después de cambiar estos valores.

> **Importante:** OpenClaw actualmente tiene por defecto `permissionMode=approve-reads` y `nonInteractivePermissions=fail`. En sesiones ACP no interactivas, cualquier escritura o ejecución que active una solicitud de permiso puede fallar con `AcpRuntimeError: Permission prompt unavailable in non-interactive mode`. Si necesitas restringir permisos, establece `nonInteractivePermissions` en `deny` para que las sesiones se degraden elegantemente en lugar de fallar.

## Solución de problemas

| Síntoma | Causa probable | Solución |
| --- | --- | --- |
| `ACP runtime backend is not configured` | Plugin backend faltante o deshabilitado. | Instalar y habilitar plugin backend, luego ejecutar `/acp doctor`. |
| `ACP is disabled by policy (acp.enabled=false)` | ACP globalmente deshabilitado. | Establecer `acp.enabled=true`. |
| `ACP dispatch is disabled by policy (acp.dispatch.enabled=false)` | Despacho desde mensajes normales de hilo deshabilitado. | Establecer `acp.dispatch.enabled=true`. |
| `ACP agent "" is not allowed by policy` | Agente no en lista de permitidos. | Usar `agentId` permitido o actualizar `acp.allowedAgents`. |
| `Unable to resolve session target: ...` | Token de clave/id/etiqueta incorrecto. | Ejecutar `/acp sessions`, copiar clave/etiqueta exacta, reintentar. |
| `--thread here requires running /acp spawn inside an active ... thread` | `--thread here` usado fuera de un contexto de hilo. | Moverse al hilo objetivo o usar `--thread auto`/`off`. |
| `Only <user-id> can rebind this thread.` | Otro usuario posee la vinculación del hilo. | Revincular como propietario o usar un hilo diferente. |
| `Thread bindings are unavailable for .` | El adaptador carece de capacidad de vinculación de hilos. | Usar `--thread off` o moverse a adaptador/canal compatible. |
| `Sandboxed sessions cannot spawn ACP sessions ...` | El tiempo de ejecución ACP está en el lado del host; la sesión del solicitante está en sandbox. | Usar `runtime="subagent"` desde sesiones en sandbox, o ejecutar spawn ACP desde una sesión no sandbox. |
| `sessions_spawn sandbox="require" is unsupported for runtime="acp" ...` | `sandbox="require"` solicitado para tiempo de ejecución ACP. | Usar `runtime="subagent"` para sandbox requerido, o usar ACP con `sandbox="inherit"` desde una sesión no sandbox. |
| Falta metadato ACP para sesión vinculada | Metadato de sesión ACP obsoleto/eliminado. | Recrear con `/acp spawn`, luego revincular/enfocar hilo. |
| `AcpRuntimeError: Permission prompt unavailable in non-interactive mode` | `permissionMode` bloquea escrituras/ejecuciones en sesión ACP no interactiva. | Establecer `plugins.entries.acpx.config.permissionMode` en `approve-all` y reiniciar puerta de enlace. Ver [Configuración de permisos](#permission-configuration). |
| La sesión ACP falla temprano con poca salida | Las solicitudes de permiso están bloqueadas por `permissionMode`/`nonInteractivePermissions`. | Verificar registros de la puerta de enlace para `AcpRuntimeError`. Para permisos completos, establecer `permissionMode=approve-all`; para degradación elegante, establecer `nonInteractivePermissions=deny`. |
| La sesión ACP se detiene indefinidamente después de completar el trabajo | El proceso del entorno terminó pero la sesión ACP no reportó finalización. | Monitorear con `ps aux \| grep acpx`; matar procesos obsoletos manualmente. |

[Sub-Agents](./subagents.md)[Multi-Agent Sandbox & Tools](./multi-agent-sandbox-tools.md)