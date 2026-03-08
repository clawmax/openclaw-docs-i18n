

  Herramientas integradas

  
# Herramienta Exec

Ejecuta comandos de shell en el workspace. Soporta ejecución en primer plano + segundo plano mediante `process`. Si `process` no está permitido, `exec` se ejecuta de forma síncrona e ignora `yieldMs`/`background`. Las sesiones en segundo plano están limitadas por agente; `process` solo ve sesiones del mismo agente.

## Parámetros

-   `command` (obligatorio)
-   `workdir` (por defecto al directorio de trabajo actual)
-   `env` (anulaciones clave/valor)
-   `yieldMs` (por defecto 10000): pasa automáticamente a segundo plano después del retraso
-   `background` (bool): pasa a segundo plano inmediatamente
-   `timeout` (segundos, por defecto 1800): termina al expirar
-   `pty` (bool): ejecuta en un pseudo-terminal cuando esté disponible (CLIs solo TTY, agentes de codificación, UIs de terminal)
-   `host` (`sandbox | gateway | node`): dónde ejecutar
-   `security` (`deny | allowlist | full`): modo de aplicación para `gateway`/`node`
-   `ask` (`off | on-miss | always`): solicitudes de aprobación para `gateway`/`node`
-   `node` (string): id/nombre del nodo para `host=node`
-   `elevated` (bool): solicita modo elevado (host gateway); `security=full` solo se fuerza cuando elevated se resuelve a `full`

Notas:

-   `host` por defecto es `sandbox`.
-   `elevated` se ignora cuando el sandboxing está desactivado (exec ya se ejecuta en el host).
-   Las aprobaciones de `gateway`/`node` se controlan mediante `~/.openclaw/exec-approvals.json`.
-   `node` requiere un nodo emparejado (aplicación companion o host de nodo headless).
-   Si hay múltiples nodos disponibles, configura `exec.node` o `tools.exec.node` para seleccionar uno.
-   En hosts no Windows, exec usa `SHELL` cuando está configurado; si `SHELL` es `fish`, prefiere `bash` (o `sh`) desde `PATH` para evitar scripts incompatibles con fish, luego recurre a `SHELL` si ninguno existe.
-   En hosts Windows, exec prefiere la detección de PowerShell 7 (`pwsh`) (Program Files, ProgramW6432, luego PATH), luego recurre a Windows PowerShell 5.1.
-   La ejecución en host (`gateway`/`node`) rechaza `env.PATH` y anulaciones de cargador (`LD_*`/`DYLD_*`) para prevenir secuestro de binarios o inyección de código.
-   OpenClaw establece `OPENCLAW_SHELL=exec` en el entorno del comando generado (incluyendo ejecución PTY y sandbox) para que las reglas del shell/perfil puedan detectar el contexto de la herramienta exec.
-   Importante: el sandboxing está **desactivado por defecto**. Si el sandboxing está desactivado y se configura/solicita explícitamente `host=sandbox`, exec ahora falla cerrado en lugar de ejecutarse silenciosamente en el host gateway. Activa el sandboxing o usa `host=gateway` con aprobaciones.
-   Las comprobaciones previas de scripts (para errores comunes de sintaxis de shell en Python/Node) solo inspeccionan archivos dentro del límite efectivo de `workdir`. Si la ruta de un script se resuelve fuera de `workdir`, se omite la comprobación previa para ese archivo.

## Configuración

-   `tools.exec.notifyOnExit` (por defecto: true): cuando es true, las sesiones exec en segundo plano encolan un evento del sistema y solicitan un heartbeat al salir.
-   `tools.exec.approvalRunningNoticeMs` (por defecto: 10000): emite un único aviso "running" cuando un exec sujeto a aprobación se ejecuta más tiempo que este (0 lo desactiva).
-   `tools.exec.host` (por defecto: `sandbox`)
-   `tools.exec.security` (por defecto: `deny` para sandbox, `allowlist` para gateway + node cuando no está configurado)
-   `tools.exec.ask` (por defecto: `on-miss`)
-   `tools.exec.node` (por defecto: sin configurar)
-   `tools.exec.pathPrepend`: lista de directorios para anteponer a `PATH` para las ejecuciones de exec (solo gateway + sandbox).
-   `tools.exec.safeBins`: binarios seguros solo para stdin que pueden ejecutarse sin entradas explícitas en la lista de permitidos. Para detalles de comportamiento, consulta [Binarios seguros](./exec-approvals.md#safe-bins-stdin-only).
-   `tools.exec.safeBinTrustedDirs`: directorios explícitos adicionales confiables para las comprobaciones de ruta de `safeBins`. Las entradas de `PATH` nunca son automáticamente confiables. Los valores por defecto integrados son `/bin` y `/usr/bin`.
-   `tools.exec.safeBinProfiles`: política de argv personalizada opcional por binario seguro (`minPositional`, `maxPositional`, `allowedValueFlags`, `deniedFlags`).

Ejemplo:

```json
{
  tools: {
    exec: {
      pathPrepend: ["~/bin", "/opt/oss/bin"],
    },
  },
}
```

### Manejo de PATH

-   `host=gateway`: fusiona tu `PATH` del shell de inicio de sesión en el entorno de exec. Las anulaciones de `env.PATH` son rechazadas para la ejecución en host. El daemon en sí aún se ejecuta con un `PATH` mínimo:
    -   macOS: `/opt/homebrew/bin`, `/usr/local/bin`, `/usr/bin`, `/bin`
    -   Linux: `/usr/local/bin`, `/usr/bin`, `/bin`
-   `host=sandbox`: ejecuta `sh -lc` (shell de inicio de sesión) dentro del contenedor, por lo que `/etc/profile` puede restablecer `PATH`. OpenClaw antepone `env.PATH` después del sourcing del perfil mediante una variable de entorno interna (sin interpolación de shell); `tools.exec.pathPrepend` también se aplica aquí.
-   `host=node`: solo las anulaciones de entorno no bloqueadas que pases se envían al nodo. Las anulaciones de `env.PATH` son rechazadas para la ejecución en host e ignoradas por los hosts de nodo. Si necesitas entradas PATH adicionales en un nodo, configura el entorno del servicio host del nodo (systemd/launchd) o instala herramientas en ubicaciones estándar.

Vinculación de nodo por agente (usa el índice de la lista de agentes en la configuración):

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

UI de control: la pestaña Nodes incluye un pequeño panel "Exec node binding" para las mismas configuraciones.

## Anulaciones de sesión (/exec)

Usa `/exec` para establecer valores por defecto **por sesión** para `host`, `security`, `ask` y `node`. Envía `/exec` sin argumentos para mostrar los valores actuales. Ejemplo:

```bash
/exec host=gateway security=allowlist ask=on-miss node=mac-1
```

## Modelo de autorización

`/exec` solo se respeta para **remitentes autorizados** (listas de permitidos de canal/emparejamiento más `commands.useAccessGroups`). Actualiza **solo el estado de la sesión** y no escribe configuración. Para desactivar exec de forma dura, deniégala mediante la política de herramientas (`tools.deny: ["exec"]` o por agente). Las aprobaciones de host aún se aplican a menos que configures explícitamente `security=full` y `ask=off`.

## Aprobaciones de Exec (aplicación companion / host de nodo)

Los agentes en sandbox pueden requerir aprobación por solicitud antes de que `exec` se ejecute en el gateway o host de nodo. Consulta [Aprobaciones de Exec](./exec-approvals.md) para conocer la política, lista de permitidos y flujo de UI. Cuando se requieren aprobaciones, la herramienta exec regresa inmediatamente con `status: "approval-pending"` y un id de aprobación. Una vez aprobado (o denegado / agotado el tiempo), el Gateway emite eventos del sistema (`Exec finished` / `Exec denied`). Si el comando aún se está ejecutando después de `tools.exec.approvalRunningNoticeMs`, se emite un único aviso `Exec running`.

## Lista de permitidos + binarios seguros

La aplicación manual de la lista de permitidos coincide **solo con rutas de binarios resueltas** (sin coincidencias de nombre base). Cuando `security=allowlist`, los comandos de shell se permiten automáticamente solo si cada segmento de la pipeline está en la lista de permitidos o es un binario seguro. El encadenamiento (`;`, `&&`, `||`) y las redirecciones son rechazados en modo allowlist a menos que cada segmento de primer nivel cumpla con la lista de permitidos (incluyendo binarios seguros). Las redirecciones siguen sin ser compatibles. `autoAllowSkills` es una ruta de conveniencia separada en las aprobaciones de exec. No es lo mismo que las entradas manuales de ruta en la lista de permitidos. Para una confianza explícita estricta, mantén `autoAllowSkills` desactivado. Usa los dos controles para trabajos diferentes:

-   `tools.exec.safeBins`: pequeños filtros de flujo solo para stdin.
-   `tools.exec.safeBinTrustedDirs`: directorios confiables extra explícitos para rutas ejecutables de binarios seguros.
-   `tools.exec.safeBinProfiles`: política de argv explícita para binarios seguros personalizados.
-   allowlist: confianza explícita para rutas ejecutables.

No trates `safeBins` como una lista de permitidos genérica, y no agregues binarios de intérprete/runtime (por ejemplo `python3`, `node`, `ruby`, `bash`). Si los necesitas, usa entradas explícitas en la lista de permitidos y mantén las solicitudes de aprobación activadas. `openclaw security audit` advierte cuando faltan perfiles explícitos para entradas `safeBins` de intérprete/runtime, y `openclaw doctor --fix` puede generar entradas personalizadas faltantes de `safeBinProfiles`. Para detalles completos de política y ejemplos, consulta [Aprobaciones de Exec](./exec-approvals.md#safe-bins-stdin-only) y [Binarios seguros versus lista de permitidos](./exec-approvals.md#safe-bins-versus-allowlist).

## Ejemplos

Primer plano:

```json
{ "tool": "exec", "command": "ls -la" }
```

Segundo plano + sondeo:

```json
{"tool":"exec","command":"npm run build","yieldMs":1000}
{"tool":"process","action":"poll","sessionId":"<id>"}
```

Enviar teclas (estilo tmux):

```json
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Enter"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["C-c"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Up","Up","Enter"]}
```

Enviar (solo envía CR):

```json
{ "tool": "process", "action": "submit", "sessionId": "<id>" }
```

Pegar (delimitado por defecto):

```json
{ "tool": "process", "action": "paste", "sessionId": "<id>", "text": "line1\nline2\n" }
```

## apply\_patch (experimental)

`apply_patch` es una subherramienta de `exec` para ediciones estructuradas de múltiples archivos. Actívala explícitamente:

```json
{
  tools: {
    exec: {
      applyPatch: { enabled: true, workspaceOnly: true, allowModels: ["gpt-5.2"] },
    },
  },
}
```

Notas:

-   Solo disponible para modelos OpenAI/OpenAI Codex.
-   La política de herramientas aún se aplica; `allow: ["exec"]` permite implícitamente `apply_patch`.
-   La configuración reside bajo `tools.exec.applyPatch`.
-   `tools.exec.applyPatch.workspaceOnly` por defecto es `true` (contenido en el workspace). Configúralo en `false` solo si intencionalmente quieres que `apply_patch` escriba/elimine fuera del directorio del workspace.

[Modo Elevado](./elevated.md)[Aprobaciones de Exec](./exec-approvals.md)