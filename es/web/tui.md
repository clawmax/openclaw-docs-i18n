

  Interfaces web

  
# TUI

## Inicio rápido

1.  Inicia el Gateway.

```bash
openclaw gateway
```

2.  Abre la TUI.

```bash
openclaw tui
```

3.  Escribe un mensaje y presiona Enter.

Gateway remoto:

```bash
openclaw tui --url ws://<host>:<port> --token <gateway-token>
```

Usa `--password` si tu Gateway usa autenticación por contraseña.

## Lo que ves

-   Encabezado: URL de conexión, agente actual, sesión actual.
-   Registro de chat: mensajes del usuario, respuestas del asistente, avisos del sistema, tarjetas de herramientas.
-   Línea de estado: estado de conexión/ejecución (conectando, ejecutando, transmitiendo, inactivo, error).
-   Pie de página: estado de conexión + agente + sesión + modelo + pensar/verbose/razonamiento + conteo de tokens + entregar.
-   Entrada: editor de texto con autocompletado.

## Modelo mental: agentes + sesiones

-   Los agentes son identificadores únicos (ej. `main`, `research`). El Gateway expone la lista.
-   Las sesiones pertenecen al agente actual.
-   Las claves de sesión se almacenan como `agent::`.
    -   Si escribes `/session main`, la TUI lo expande a `agent::main`.
    -   Si escribes `/session agent:other:main`, cambias a esa sesión de agente explícitamente.
-   Alcance de la sesión:
    -   `per-sender` (predeterminado): cada agente tiene muchas sesiones.
    -   `global`: la TUI siempre usa la sesión `global` (el selector puede estar vacío).
-   El agente actual + la sesión actual siempre son visibles en el pie de página.

## Envío + entrega

-   Los mensajes se envían al Gateway; la entrega a los proveedores está desactivada por defecto.
-   Activa la entrega:
    -   `/deliver on`
    -   o el panel de Configuración
    -   o inicia con `openclaw tui --deliver`

## Selectores + superposiciones

-   Selector de modelo: lista los modelos disponibles y establece la anulación de sesión.
-   Selector de agente: elige un agente diferente.
-   Selector de sesión: muestra solo las sesiones para el agente actual.
-   Configuración: alterna entrega, expansión de salida de herramientas y visibilidad del pensamiento.

## Atajos de teclado

-   Enter: enviar mensaje
-   Esc: abortar ejecución activa
-   Ctrl+C: borrar entrada (presiona dos veces para salir)
-   Ctrl+D: salir
-   Ctrl+L: selector de modelo
-   Ctrl+G: selector de agente
-   Ctrl+P: selector de sesión
-   Ctrl+O: alternar expansión de salida de herramientas
-   Ctrl+T: alternar visibilidad del pensamiento (recarga el historial)

## Comandos de barra

Núcleo:

-   `/help`
-   `/status`
-   `/agent ` (o `/agents`)
-   `/session ` (o `/sessions`)
-   `/model <provider/model>` (o `/models`)

Controles de sesión:

-   `/think <off|minimal|low|medium|high>`
-   `/verbose <on|full|off>`
-   `/reasoning <on|off|stream>`
-   `/usage <off|tokens|full>`
-   `/elevated <on|off|ask|full>` (alias: `/elev`)
-   `/activation <mention|always>`
-   `/deliver <on|off>`

Ciclo de vida de la sesión:

-   `/new` o `/reset` (reinicia la sesión)
-   `/abort` (aborta la ejecución activa)
-   `/settings`
-   `/exit`

Otros comandos de barra del Gateway (por ejemplo, `/context`) se reenvían al Gateway y se muestran como salida del sistema. Consulta [Comandos de barra](../tools/slash-commands.md).

## Comandos de shell local

-   Prefija una línea con `!` para ejecutar un comando de shell local en el host de la TUI.
-   La TUI solicita permiso una vez por sesión para permitir la ejecución local; rechazar mantiene `!` deshabilitado para la sesión.
-   Los comandos se ejecutan en un shell nuevo y no interactivo en el directorio de trabajo de la TUI (sin `cd`/env persistentes).
-   Los comandos de shell local reciben `OPENCLAW_SHELL=tui-local` en su entorno.
-   Un `!` solo se envía como un mensaje normal; los espacios iniciales no activan la ejecución local.

## Salida de herramientas

-   Las llamadas a herramientas se muestran como tarjetas con argumentos + resultados.
-   Ctrl+O alterna entre vistas contraída/expandida.
-   Mientras las herramientas se ejecutan, las actualizaciones parciales se transmiten a la misma tarjeta.

## Historial + transmisión

-   Al conectar, la TUI carga el historial más reciente (200 mensajes por defecto).
-   Las respuestas en transmisión se actualizan en el lugar hasta que se finalizan.
-   La TUI también escucha eventos de herramientas del agente para tarjetas de herramientas más ricas.

## Detalles de conexión

-   La TUI se registra en el Gateway como `mode: "tui"`.
-   Las reconexiones muestran un mensaje del sistema; los huecos en eventos se muestran en el registro.

## Opciones

-   `--url `: URL WebSocket del Gateway (por defecto la configuración o `ws://127.0.0.1:`)
-   `--token `: Token del Gateway (si es requerido)
-   `--password `: Contraseña del Gateway (si es requerida)
-   `--session `: Clave de sesión (por defecto: `main`, o `global` cuando el alcance es global)
-   `--deliver`: Entregar respuestas del asistente al proveedor (desactivado por defecto)
-   `--thinking `: Anular el nivel de pensamiento para los envíos
-   `--timeout-ms `: Tiempo de espera del agente en ms (por defecto `agents.defaults.timeoutSeconds`)

Nota: cuando configuras `--url`, la TUI no recurre a credenciales de configuración o entorno. Pasa `--token` o `--password` explícitamente. La falta de credenciales explícitas es un error.

## Solución de problemas

Sin salida después de enviar un mensaje:

-   Ejecuta `/status` en la TUI para confirmar que el Gateway está conectado e inactivo/ocupado.
-   Revisa los registros del Gateway: `openclaw logs --follow`.
-   Confirma que el agente puede ejecutarse: `openclaw status` y `openclaw models status`.
-   Si esperas mensajes en un canal de chat, activa la entrega (`/deliver on` o `--deliver`).
-   `--history-limit `: Entradas del historial a cargar (200 por defecto)

## Solución de problemas de conexión

-   `desconectado`: asegúrate de que el Gateway esté ejecutándose y que tu `--url/--token/--password` sean correctos.
-   Sin agentes en el selector: verifica `openclaw agents list` y tu configuración de enrutamiento.
-   Selector de sesión vacío: podrías estar en alcance global o no tener sesiones aún.

[WebChat](./webchat.md)

---