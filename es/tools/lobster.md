

  Herramientas integradas

  
# Lobster

Lobster es un shell de flujo de trabajo que permite a OpenClaw ejecutar secuencias de herramientas multi-paso como una única operación determinista con puntos de control de aprobación explícitos.

## Gancho

Tu asistente puede construir las herramientas que se gestionan a sí mismas. Pide un flujo de trabajo, y 30 minutos después tienes una CLI más tuberías que se ejecutan como una llamada. Lobster es la pieza faltante: tuberías deterministas, aprobaciones explícitas y estado reanudable.

## Por qué

Hoy, los flujos de trabajo complejos requieren muchas llamadas de herramientas de ida y vuelta. Cada llamada cuesta tokens, y el LLM tiene que orquestar cada paso. Lobster mueve esa orquestación a un entorno de ejecución tipado:

-   **Una llamada en lugar de muchas**: OpenClaw ejecuta una llamada a la herramienta Lobster y obtiene un resultado estructurado.
-   **Aprobaciones integradas**: Los efectos secundarios (enviar correo, publicar comentario) detienen el flujo de trabajo hasta que sean aprobados explícitamente.
-   **Reanudable**: Los flujos de trabajo detenidos devuelven un token; aprueba y reanuda sin volver a ejecutar todo.

## ¿Por qué un DSL en lugar de programas simples?

Lobster es intencionalmente pequeño. El objetivo no es "un nuevo lenguaje", es una especificación de tubería predecible y amigable para IA con aprobaciones y tokens de reanudación de primera clase.

-   **Aprobar/reanudar está integrado**: Un programa normal puede solicitar a un humano, pero no puede *pausar y reanudar* con un token duradero sin que inventes ese entorno de ejecución tú mismo.
-   **Determinismo + auditabilidad**: Las tuberías son datos, por lo que son fáciles de registrar, comparar, reproducir y revisar.
-   **Superficie restringida para IA**: Una gramática pequeña + tuberías JSON reduce las rutas de código "creativas" y hace que la validación sea realista.
-   **Política de seguridad integrada**: Los tiempos de espera, límites de salida, comprobaciones de sandbox y listas de permitidos son aplicados por el entorno de ejecución, no por cada script.
-   **Sigue siendo programable**: Cada paso puede llamar a cualquier CLI o script. Si quieres JS/TS, genera archivos `.lobster` desde código.

## Cómo funciona

OpenClaw inicia la CLI local `lobster` en **modo herramienta** y analiza un sobre JSON desde stdout. Si la tubería se pausa para aprobación, la herramienta devuelve un `resumeToken` para que puedas continuar más tarde.

## Patrón: CLI pequeña + tuberías JSON + aprobaciones

Construye comandos diminutos que hablen JSON, luego encadénalos en una única llamada Lobster. (Nombres de comandos de ejemplo a continuación — sustituye por los tuyos.)

```bash
inbox list --json
inbox categorize --json
inbox apply --json
```

```json
{
  "action": "run",
  "pipeline": "exec --json --shell 'inbox list --json' | exec --stdin json --shell 'inbox categorize --json' | exec --stdin json --shell 'inbox apply --json' | approve --preview-from-stdin --limit 5 --prompt 'Apply changes?'",
  "timeoutMs": 30000
}
```

Si la tubería solicita aprobación, reanuda con el token:

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

La IA desencadena el flujo de trabajo; Lobster ejecuta los pasos. Las compuertas de aprobación mantienen los efectos secundarios explícitos y auditables. Ejemplo: mapear elementos de entrada en llamadas a herramientas:

```
gog.gmail.search --query 'newer_than:1d' \
  | openclaw.invoke --tool message --action send --each --item-key message --args-json '{"provider":"telegram","to":"..."}'
```

## Pasos LLM solo JSON (llm-task)

Para flujos de trabajo que necesitan un **paso LLM estructurado**, habilita la herramienta de plugin opcional `llm-task` y llámala desde Lobster. Esto mantiene el flujo de trabajo determinista mientras aún te permite clasificar/resumir/redactar con un modelo. Habilita la herramienta:

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  },
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": { "allow": ["llm-task"] }
      }
    ]
  }
}
```

Úsala en una tubería:

```
openclaw.invoke --tool llm-task --action json --args-json '{
  "prompt": "Given the input email, return intent and draft.",
  "input": { "subject": "Hello", "body": "Can you help?" },
  "schema": {
    "type": "object",
    "properties": {
      "intent": { "type": "string" },
      "draft": { "type": "string" }
    },
    "required": ["intent", "draft"],
    "additionalProperties": false
  }
}'
```

Consulta [LLM Task](./llm-task.md) para detalles y opciones de configuración.

## Archivos de flujo de trabajo (.lobster)

Lobster puede ejecutar archivos de flujo de trabajo YAML/JSON con campos `name`, `args`, `steps`, `env`, `condition` y `approval`. En las llamadas a herramientas de OpenClaw, establece `pipeline` en la ruta del archivo.

```yaml
name: inbox-triage
args:
  tag:
    default: "family"
steps:
  - id: collect
    command: inbox list --json
  - id: categorize
    command: inbox categorize --json
    stdin: $collect.stdout
  - id: approve
    command: inbox apply --approve
    stdin: $categorize.stdout
    approval: required
  - id: execute
    command: inbox apply --execute
    stdin: $categorize.stdout
    condition: $approve.approved
```

Notas:

-   `stdin: $step.stdout` y `stdin: $step.json` pasan la salida de un paso anterior.
-   `condition` (o `when`) puede condicionar pasos a `$step.approved`.

## Instalar Lobster

Instala la CLI de Lobster en el **mismo host** que ejecuta el Gateway de OpenClaw (consulta el [repositorio de Lobster](https://github.com/openclaw/lobster)), y asegúrate de que `lobster` esté en `PATH`.

## Habilitar la herramienta

Lobster es una herramienta de plugin **opcional** (no habilitada por defecto). Recomendado (aditivo, seguro):

```json
{
  "tools": {
    "alsoAllow": ["lobster"]
  }
}
```

O por agente:

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": {
          "alsoAllow": ["lobster"]
        }
      }
    ]
  }
}
```

Evita usar `tools.allow: ["lobster"]` a menos que pretendas ejecutar en modo de lista de permitidos restrictiva. Nota: las listas de permitidos son opcionales para plugins opcionales. Si tu lista de permitidos solo nombra herramientas de plugin (como `lobster`), OpenClaw mantiene las herramientas principales habilitadas. Para restringir las herramientas principales, incluye también las herramientas o grupos principales que quieras en la lista de permitidos.

## Ejemplo: Triaje de correo

Sin Lobster:

```
Usuario: "Revisa mi correo y redacta respuestas"
→ openclaw llama a gmail.list
→ LLM resume
→ Usuario: "redacta respuestas para #2 y #5"
→ LLM redacta
→ Usuario: "envía #2"
→ openclaw llama a gmail.send
(repetir diariamente, sin memoria de lo que se trió)
```

Con Lobster:

```json
{
  "action": "run",
  "pipeline": "email.triage --limit 20",
  "timeoutMs": 30000
}
```

Devuelve un sobre JSON (truncado):

```json
{
  "ok": true,
  "status": "needs_approval",
  "output": [{ "summary": "5 need replies, 2 need action" }],
  "requiresApproval": {
    "type": "approval_request",
    "prompt": "Send 2 draft replies?",
    "items": [],
    "resumeToken": "..."
  }
}
```

Usuario aprueba → reanudar:

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

Un flujo de trabajo. Determinista. Seguro.

## Parámetros de la herramienta

### run

Ejecuta una tubería en modo herramienta.

```json
{
  "action": "run",
  "pipeline": "gog.gmail.search --query 'newer_than:1d' | email.triage",
  "cwd": "workspace",
  "timeoutMs": 30000,
  "maxStdoutBytes": 512000
}
```

Ejecuta un archivo de flujo de trabajo con argumentos:

```json
{
  "action": "run",
  "pipeline": "/path/to/inbox-triage.lobster",
  "argsJson": "{\"tag\":\"family\"}"
}
```

### resume

Continúa un flujo de trabajo detenido después de la aprobación.

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

### Entradas opcionales

-   `cwd`: Directorio de trabajo relativo para la tubería (debe permanecer dentro del directorio de trabajo actual del proceso).
-   `timeoutMs`: Mata el subproceso si excede esta duración (por defecto: 20000).
-   `maxStdoutBytes`: Mata el subproceso si stdout excede este tamaño (por defecto: 512000).
-   `argsJson`: Cadena JSON pasada a `lobster run --args-json` (solo archivos de flujo de trabajo).

## Sobre de salida

Lobster devuelve un sobre JSON con uno de tres estados:

-   `ok` → finalizado con éxito
-   `needs_approval` → pausado; se requiere `requiresApproval.resumeToken` para reanudar
-   `cancelled` → denegado o cancelado explícitamente

La herramienta muestra el sobre tanto en `content` (JSON formateado) como en `details` (objeto crudo).

## Aprobaciones

Si `requiresApproval` está presente, inspecciona el mensaje y decide:

-   `approve: true` → reanuda y continúa los efectos secundarios
-   `approve: false` → cancela y finaliza el flujo de trabajo

Usa `approve --preview-from-stdin --limit N` para adjuntar una vista previa JSON a las solicitudes de aprobación sin pegamento personalizado jq/heredoc. Los tokens de reanudación ahora son compactos: Lobster almacena el estado de reanudación del flujo de trabajo bajo su directorio de estado y devuelve una pequeña clave token.

## OpenProse

OpenProse combina bien con Lobster: usa `/prose` para orquestar preparación multi-agente, luego ejecuta una tubería Lobster para aprobaciones deterministas. Si un programa Prose necesita Lobster, permite la herramienta `lobster` para subagentes a través de `tools.subagents.tools`. Consulta [OpenProse](../prose.md).

## Seguridad

-   **Solo subproceso local** — sin llamadas de red desde el plugin en sí.
-   **Sin secretos** — Lobster no gestiona OAuth; llama a herramientas de OpenClaw que sí lo hacen.
-   **Consciente del sandbox** — deshabilitado cuando el contexto de la herramienta está en sandbox.
-   **Endurecido** — nombre de ejecutable fijo (`lobster`) en `PATH`; se aplican tiempos de espera y límites de salida.

## Solución de problemas

-   **`lobster subprocess timed out`** → aumenta `timeoutMs`, o divide una tubería larga.
-   **`lobster output exceeded maxStdoutBytes`** → aumenta `maxStdoutBytes` o reduce el tamaño de la salida.
-   **`lobster returned invalid JSON`** → asegúrate de que la tubería se ejecute en modo herramienta e imprima solo JSON.
-   **`lobster failed (code …)`** → ejecuta la misma tubería en una terminal para inspeccionar stderr.

## Aprende más

-   [Plugins](./plugin.md)
-   [Creación de herramientas de plugin](../plugins/agent-tools.md)

## Estudio de caso: flujos de trabajo comunitarios

Un ejemplo público: una CLI de "segundo cerebro" + tuberías Lobster que gestionan tres bóvedas Markdown (personal, pareja, compartida). La CLI emite JSON para estadísticas, listados de bandeja de entrada y escaneos obsoletos; Lobster encadena esos comandos en flujos de trabajo como `weekly-review`, `inbox-triage`, `memory-consolidation` y `shared-task-sync`, cada uno con compuertas de aprobación. La IA maneja el juicio (categorización) cuando está disponible y recurre a reglas deterministas cuando no lo está.

-   Hilo: [https://x.com/plattenschieber/status/2014508656335770033](https://x.com/plattenschieber/status/2014508656335770033)
-   Repositorio: [https://github.com/bloomedai/brain-cli](https://github.com/bloomedai/brain-cli)

[LLM Task](./llm-task.md)[Detección de bucle de herramientas](./loop-detection.md)