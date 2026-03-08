title: "Gestión de Colas para Ejecuciones de Agente de Respuesta Automática en OpenClaw"
description: "Aprende cómo OpenClaw serializa los mensajes entrantes usando una cola para prevenir colisiones, gestionar la concurrencia y configurar modos y opciones de cola."
keywords: ["cola de mensajes", "respuesta automática", "concurrencia", "gestión de sesiones", "modos de cola", "mensajes entrantes", "ejecuciones de agente", "serialización"]
---

  Mensajes y entrega

  
# Cola de Comandos

Serializamos las ejecuciones de respuesta automática entrantes (todos los canales) a través de una pequeña cola en proceso para evitar que múltiples ejecuciones de agente colisionen, permitiendo al mismo tiempo un paralelismo seguro entre sesiones.

## Por qué

-   Las ejecuciones de respuesta automática pueden ser costosas (llamadas a LLM) y pueden colisionar cuando llegan múltiples mensajes entrantes muy seguidos.
-   La serialización evita la competencia por recursos compartidos (archivos de sesión, registros, stdin de la CLI) y reduce la posibilidad de límites de tasa (rate limits) aguas arriba.

## Cómo funciona

-   Una cola FIFO consciente de carriles drena cada carril con un límite de concurrencia configurable (por defecto 1 para carriles no configurados; el principal por defecto es 4, el subagente es 8).
-   `runEmbeddedPiAgent` encola por **clave de sesión** (carril `session:`) para garantizar solo una ejecución activa por sesión.
-   Cada ejecución de sesión se encola luego en un **carril global** (`main` por defecto) para que el paralelismo total esté limitado por `agents.defaults.maxConcurrent`.
-   Cuando el registro detallado (verbose logging) está habilitado, las ejecuciones encoladas emiten un breve aviso si esperaron más de ~2s antes de comenzar.
-   Los indicadores de escritura (typing indicators) aún se activan inmediatamente al encolar (cuando el canal lo soporta) para que la experiencia del usuario no cambie mientras esperamos nuestro turno.

## Modos de cola (por canal)

Los mensajes entrantes pueden dirigir (steer) la ejecución actual, esperar un turno de seguimiento (followup), o hacer ambas cosas:

-   `steer`: inyecta inmediatamente en la ejecución actual (cancela las llamadas a herramientas pendientes después del siguiente límite de herramienta). Si no está en modo streaming, recurre a followup.
-   `followup`: encola para el siguiente turno del agente después de que termine la ejecución actual.
-   `collect`: combina todos los mensajes encolados en un **único** turno de seguimiento (por defecto). Si los mensajes apuntan a diferentes canales/hilos, se drenan individualmente para preservar el enrutamiento.
-   `steer-backlog` (también `steer+backlog`): dirige (steer) ahora **y** preserva el mensaje para un turno de seguimiento.
-   `interrupt` (heredado): aborta la ejecución activa para esa sesión, luego ejecuta el mensaje más nuevo.
-   `queue` (alias heredado): igual que `steer`.

Steer-backlog significa que puedes obtener una respuesta de seguimiento después de la ejecución dirigida, por lo que las superficies de streaming pueden parecer duplicados. Prefiere `collect`/`steer` si quieres una respuesta por mensaje entrante. Envía `/queue collect` como un comando independiente (por sesión) o configura `messages.queue.byChannel.discord: "collect"`. Valores por defecto (cuando no están configurados):

-   Todas las superficies → `collect`

Configura globalmente o por canal a través de `messages.queue`:

```json
{
  messages: {
    queue: {
      mode: "collect",
      debounceMs: 1000,
      cap: 20,
      drop: "summarize",
      byChannel: { discord: "collect" },
    },
  },
}
```

## Opciones de cola

Las opciones se aplican a `followup`, `collect` y `steer-backlog` (y a `steer` cuando recurre a followup):

-   `debounceMs`: espera por silencio antes de comenzar un turno de seguimiento (previene "continuar, continuar").
-   `cap`: máximo de mensajes encolados por sesión.
-   `drop`: política de desbordamiento (`old`, `new`, `summarize`).

Summarize mantiene una breve lista con viñetas de los mensajes descartados y la inyecta como un prompt de seguimiento sintético. Valores por defecto: `debounceMs: 1000`, `cap: 20`, `drop: summarize`.

## Anulaciones por sesión

-   Envía `/queue ` como un comando independiente para almacenar el modo para la sesión actual.
-   Las opciones se pueden combinar: `/queue collect debounce:2s cap:25 drop:summarize`
-   `/queue default` o `/queue reset` borra la anulación de sesión.

## Alcance y garantías

-   Se aplica a las ejecuciones de agente de respuesta automática en todos los canales entrantes que usan la canalización de respuesta del gateway (WhatsApp web, Telegram, Slack, Discord, Signal, iMessage, webchat, etc.).
-   El carril por defecto (`main`) es a nivel de proceso para entrantes + latidos principales (heartbeats); configura `agents.defaults.maxConcurrent` para permitir múltiples sesiones en paralelo.
-   Pueden existir carriles adicionales (ej. `cron`, `subagent`) para que los trabajos en segundo plano puedan ejecutarse en paralelo sin bloquear las respuestas entrantes.
-   Los carriles por sesión garantizan que solo una ejecución de agente toque una sesión dada a la vez.
-   Sin dependencias externas o hilos de trabajo en segundo plano; TypeScript puro + promesas.

## Resolución de problemas

-   Si los comandos parecen atascados, habilita los registros detallados (verbose logs) y busca líneas "queued for …ms" para confirmar que la cola se está drenando.
-   Si necesitas profundidad de cola, habilita los registros detallados y observa las líneas de tiempo de cola.

[Política de Reintento](./retry.md)

---