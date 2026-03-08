

  Fundamentos

  
# Contexto

"Contexto" es **todo lo que OpenClaw envía al modelo para una ejecución**. Está limitado por la **ventana de contexto** del modelo (límite de tokens). Modelo mental para principiantes:

-   **Prompt del sistema** (construido por OpenClaw): reglas, herramientas, lista de habilidades, tiempo/tiempo de ejecución, y archivos del espacio de trabajo inyectados.
-   **Historial de conversación**: tus mensajes + los mensajes del asistente para esta sesión.
-   **Llamadas/resultados de herramientas + archivos adjuntos**: salida de comandos, lecturas de archivos, imágenes/audio, etc.

El contexto *no es lo mismo* que la "memoria": la memoria puede almacenarse en disco y recargarse más tarde; el contexto es lo que está dentro de la ventana actual del modelo.

## Inicio rápido (inspeccionar contexto)

-   `/status` → vista rápida de "¿qué tan llena está mi ventana?" + configuración de la sesión.
-   `/context list` → qué está inyectado + tamaños aproximados (por archivo + totales).
-   `/context detail` → desglose más profundo: por archivo, tamaños de esquemas por herramienta, tamaños de entradas por habilidad, y tamaño del prompt del sistema.
-   `/usage tokens` → añade un pie de página de uso por respuesta a las respuestas normales.
-   `/compact` → resume el historial antiguo en una entrada compacta para liberar espacio en la ventana.

Ver también: [Comandos de barra](../tools/slash-commands.md), [Uso y costos de tokens](../reference/token-use.md), [Compactación](./compaction.md).

## Ejemplo de salida

Los valores varían según el modelo, proveedor, política de herramientas y lo que haya en tu espacio de trabajo.

### /context list

```
🧠 Context breakdown
Workspace: <workspaceDir>
Bootstrap max/file: 20,000 chars
Sandbox: mode=non-main sandboxed=false
System prompt (run): 38,412 chars (~9,603 tok) (Project Context 23,901 chars (~5,976 tok))

Injected workspace files:
- AGENTS.md: OK | raw 1,742 chars (~436 tok) | injected 1,742 chars (~436 tok)
- SOUL.md: OK | raw 912 chars (~228 tok) | injected 912 chars (~228 tok)
- TOOLS.md: TRUNCATED | raw 54,210 chars (~13,553 tok) | injected 20,962 chars (~5,241 tok)
- IDENTITY.md: OK | raw 211 chars (~53 tok) | injected 211 chars (~53 tok)
- USER.md: OK | raw 388 chars (~97 tok) | injected 388 chars (~97 tok)
- HEARTBEAT.md: MISSING | raw 0 | injected 0
- BOOTSTRAP.md: OK | raw 0 chars (~0 tok) | injected 0 chars (~0 tok)

Skills list (system prompt text): 2,184 chars (~546 tok) (12 skills)
Tools: read, edit, write, exec, process, browser, message, sessions_send, …
Tool list (system prompt text): 1,032 chars (~258 tok)
Tool schemas (JSON): 31,988 chars (~7,997 tok) (counts toward context; not shown as text)
Tools: (same as above)

Session tokens (cached): 14,250 total / ctx=32,000
```

### /context detail

```
🧠 Context breakdown (detailed)
…
Top skills (prompt entry size):
- frontend-design: 412 chars (~103 tok)
- oracle: 401 chars (~101 tok)
… (+10 more skills)

Top tools (schema size):
- browser: 9,812 chars (~2,453 tok)
- exec: 6,240 chars (~1,560 tok)
… (+N more tools)
```

## Qué cuenta hacia la ventana de contexto

Todo lo que el modelo recibe cuenta, incluyendo:

-   Prompt del sistema (todas las secciones).
-   Historial de conversación.
-   Llamadas a herramientas + resultados de herramientas.
-   Archivos adjuntos/transcripciones (imágenes/audio/archivos).
-   Resúmenes de compactación y artefactos de poda.
-   "Envolturas" del proveedor o encabezados ocultos (no visibles, pero contados).

## Cómo OpenClaw construye el prompt del sistema

El prompt del sistema es **propiedad de OpenClaw** y se reconstruye en cada ejecución. Incluye:

-   Lista de herramientas + descripciones breves.
-   Lista de habilidades (solo metadatos; ver más abajo).
-   Ubicación del espacio de trabajo.
-   Hora (UTC + hora del usuario convertida si está configurada).
-   Metadatos de tiempo de ejecución (host/SO/modelo/pensamiento).
-   Archivos de arranque del espacio de trabajo inyectados bajo **Contexto del Proyecto**.

Desglose completo: [Prompt del Sistema](./system-prompt.md).

## Archivos del espacio de trabajo inyectados (Contexto del Proyecto)

Por defecto, OpenClaw inyecta un conjunto fijo de archivos del espacio de trabajo (si están presentes):

-   `AGENTS.md`
-   `SOUL.md`
-   `TOOLS.md`
-   `IDENTITY.md`
-   `USER.md`
-   `HEARTBEAT.md`
-   `BOOTSTRAP.md` (solo en la primera ejecución)

Los archivos grandes se truncan por archivo usando `agents.defaults.bootstrapMaxChars` (por defecto `20000` caracteres). OpenClaw también aplica un límite total de inyección de arranque entre todos los archivos con `agents.defaults.bootstrapTotalMaxChars` (por defecto `150000` caracteres). `/context` muestra los tamaños **crudo vs inyectado** y si ocurrió truncamiento. Cuando ocurre truncamiento, el tiempo de ejecución puede inyectar un bloque de advertencia en el prompt bajo Contexto del Proyecto. Configura esto con `agents.defaults.bootstrapPromptTruncationWarning` (`off`, `once`, `always`; por defecto `once`).

## Habilidades: qué se inyecta vs qué se carga bajo demanda

El prompt del sistema incluye una **lista compacta de habilidades** (nombre + descripción + ubicación). Esta lista tiene un costo real. Las instrucciones de las habilidades *no* se incluyen por defecto. Se espera que el modelo `lea` el `SKILL.md` de la habilidad **solo cuando sea necesario**.

## Herramientas: hay dos costos

Las herramientas afectan el contexto de dos maneras:

1.  **Texto de la lista de herramientas** en el prompt del sistema (lo que ves como "Herramientas").
2.  **Esquemas de herramientas** (JSON). Estos se envían al modelo para que pueda llamar a las herramientas. Cuentan hacia el contexto aunque no los veas como texto plano.

`/context detail` desglosa los esquemas de herramientas más grandes para que puedas ver qué domina.

## Comandos, directivas y "atajos en línea"

Los comandos de barra son manejados por la Pasarela. Hay algunos comportamientos diferentes:

-   **Comandos independientes**: un mensaje que es solo `/...` se ejecuta como un comando.
-   **Directivas**: `/think`, `/verbose`, `/reasoning`, `/elevated`, `/model`, `/queue` se eliminan antes de que el modelo vea el mensaje.
    -   Los mensajes que son solo directivas persisten la configuración de la sesión.
    -   Las directivas en línea en un mensaje normal actúan como sugerencias por mensaje.
-   **Atajos en línea** (solo remitentes permitidos): ciertos tokens `/...` dentro de un mensaje normal pueden ejecutarse inmediatamente (ejemplo: "oye /status"), y se eliminan antes de que el modelo vea el texto restante.

Detalles: [Comandos de barra](../tools/slash-commands.md).

## Sesiones, compactación y poda (qué persiste)

Lo que persiste entre mensajes depende del mecanismo:

-   **Historial normal** persiste en la transcripción de la sesión hasta que es compactado/podado por la política.
-   **Compactación** persiste un resumen en la transcripción y mantiene intactos los mensajes recientes.
-   **Poda** elimina resultados antiguos de herramientas del prompt *en memoria* para una ejecución, pero no reescribe la transcripción.

Documentación: [Sesión](./session.md), [Compactación](./compaction.md), [Poda de sesión](./session-pruning.md). Por defecto, OpenClaw usa el motor de contexto integrado `legacy` para el ensamblaje y la compactación. Si instalas un plugin que proporcione `kind: "context-engine"` y lo seleccionas con `plugins.slots.contextEngine`, OpenClaw delega el ensamblaje del contexto, `/compact`, y los ganchos relacionados del ciclo de vida del contexto del subagente a ese motor en su lugar.

## Qué reporta realmente /context

`/context` prefiere el último reporte del prompt del sistema **construido en ejecución** cuando está disponible:

-   `System prompt (run)` = capturado desde la última ejecución embebida (con capacidad de herramientas) y persistido en el almacén de sesiones.
-   `System prompt (estimate)` = calculado sobre la marcha cuando no existe un reporte de ejecución (o cuando se ejecuta a través de un backend CLI que no genera el reporte).

En cualquier caso, reporta tamaños y los principales contribuyentes; **no** vuelca el prompt del sistema completo ni los esquemas de herramientas.

[Prompt del Sistema](./system-prompt.md)[Espacio de Trabajo del Agente](./agent-workspace.md)