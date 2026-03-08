

  Configuración y operaciones

  
# Herramientas Exec y Process en Segundo Plano

OpenClaw ejecuta comandos de shell a través de la herramienta `exec` y mantiene tareas de larga duración en memoria. La herramienta `process` gestiona esas sesiones en segundo plano.

## Herramienta exec

Parámetros clave:

-   `command` (obligatorio)
-   `yieldMs` (por defecto 10000): pasa a segundo plano automáticamente después de este retraso
-   `background` (bool): pasa a segundo plano inmediatamente
-   `timeout` (segundos, por defecto 1800): mata el proceso después de este tiempo de espera
-   `elevated` (bool): ejecuta en el host si el modo elevado está habilitado/permitido
-   ¿Necesita un TTY real? Establece `pty: true`.
-   `workdir`, `env`

Comportamiento:

-   Las ejecuciones en primer plano devuelven la salida directamente.
-   Cuando pasa a segundo plano (explícito o por tiempo de espera), la herramienta devuelve `status: "running"` + `sessionId` y una cola corta.
-   La salida se mantiene en memoria hasta que la sesión es sondeada o borrada.
-   Si la herramienta `process` no está permitida, `exec` se ejecuta de forma sincrónica e ignora `yieldMs`/`background`.
-   Los comandos exec generados reciben `OPENCLAW_SHELL=exec` para reglas de shell/perfil conscientes del contexto.

## Puenteo de procesos hijos

Al generar procesos de larga duración fuera de las herramientas exec/process (por ejemplo, reinicios de CLI o ayudantes del gateway), adjunta el ayudante de puenteo de procesos hijos para que las señales de terminación se reenvíen y los oyentes se desvinculen al salir/error. Esto evita procesos huérfanos en systemd y mantiene un comportamiento de apagado consistente en todas las plataformas. Anulaciones de entorno:

-   `PI_BASH_YIELD_MS`: retraso de paso a segundo plano por defecto (ms)
-   `PI_BASH_MAX_OUTPUT_CHARS`: límite de salida en memoria (caracteres)
-   `OPENCLAW_BASH_PENDING_MAX_OUTPUT_CHARS`: límite de salida pendiente por flujo stdout/stderr (caracteres)
-   `PI_BASH_JOB_TTL_MS`: TTL para sesiones finalizadas (ms, limitado a 1m–3h)

Configuración (preferida):

-   `tools.exec.backgroundMs` (por defecto 10000)
-   `tools.exec.timeoutSec` (por defecto 1800)
-   `tools.exec.cleanupMs` (por defecto 1800000)
-   `tools.exec.notifyOnExit` (por defecto true): pone en cola un evento del sistema + solicita latido cuando un exec en segundo plano finaliza.
-   `tools.exec.notifyOnExitEmptySuccess` (por defecto false): cuando es true, también pone en cola eventos de finalización para ejecuciones en segundo plano exitosas que no produjeron salida.

## Herramienta process

Acciones:

-   `list`: sesiones en ejecución + finalizadas
-   `poll`: drena la nueva salida para una sesión (también informa el estado de salida)
-   `log`: lee la salida agregada (admite `offset` + `limit`)
-   `write`: envía stdin (`data`, opcional `eof`)
-   `kill`: termina una sesión en segundo plano
-   `clear`: elimina una sesión finalizada de la memoria
-   `remove`: mata si está en ejecución, de lo contrario la borra si está finalizada

Notas:

-   Solo las sesiones en segundo plano se listan/persisten en memoria.
-   Las sesiones se pierden al reiniciar el proceso (sin persistencia en disco).
-   Los registros de sesión solo se guardan en el historial del chat si ejecutas `process poll/log` y el resultado de la herramienta se registra.
-   `process` tiene alcance por agente; solo ve las sesiones iniciadas por ese agente.
-   `process list` incluye un `name` derivado (verbo del comando + objetivo) para escaneos rápidos.
-   `process log` usa `offset`/`limit` basado en líneas.
-   Cuando se omiten tanto `offset` como `limit`, devuelve las últimas 200 líneas e incluye una sugerencia de paginación.
-   Cuando se proporciona `offset` y se omite `limit`, devuelve desde `offset` hasta el final (no limitado a 200).

## Ejemplos

Ejecuta una tarea larga y sondea más tarde:

```json
{ "tool": "exec", "command": "sleep 5 && echo done", "yieldMs": 1000 }
```

```json
{ "tool": "process", "action": "poll", "sessionId": "<id>" }
```

Inicia inmediatamente en segundo plano:

```json
{ "tool": "exec", "command": "npm run build", "background": true }
```

Envía stdin:

```json
{ "tool": "process", "action": "write", "sessionId": "<id>", "data": "y\n" }
```

[Bloqueo de Gateway](./gateway-lock.md)[Múltiples Gateways](./multiple-gateways.md)