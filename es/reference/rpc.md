

  RPC y API

  
# Adaptadores RPC

OpenClaw integra CLIs externos a través de JSON-RPC. Actualmente se utilizan dos patrones.

## Patrón A: Demonio HTTP (signal-cli)

-   `signal-cli` se ejecuta como un demonio con JSON-RPC sobre HTTP.
-   El flujo de eventos es SSE (`/api/v1/events`).
-   Sonda de salud: `/api/v1/check`.
-   OpenClaw gestiona el ciclo de vida cuando `channels.signal.autoStart=true`.

Consulta [Signal](../channels/signal.md) para la configuración y los endpoints.

## Patrón B: Proceso hijo stdio (legado: imsg)

> **Nota:** Para nuevas configuraciones de iMessage, usa [BlueBubbles](../channels/bluebubbles.md) en su lugar.

-   OpenClaw genera `imsg rpc` como un proceso hijo (integración legada de iMessage).
-   JSON-RPC es delimitado por líneas sobre stdin/stdout (un objeto JSON por línea).
-   No requiere puerto TCP ni demonio.

Métodos principales utilizados:

-   `watch.subscribe` → notificaciones (`method: "message"`)
-   `watch.unsubscribe`
-   `send`
-   `chats.list` (sonda/diagnóstico)

Consulta [iMessage](../channels/imessage.md) para la configuración legada y direccionamiento (se prefiere `chat_id`).

## Directrices para adaptadores

-   La puerta de enlace posee el proceso (inicio/detención vinculado al ciclo de vida del proveedor).
-   Mantén los clientes RPC resilientes: tiempos de espera, reinicio al salir.
-   Prefiere IDs estables (ej., `chat_id`) sobre cadenas de texto para mostrar.

[webhooks](../cli/webhooks.md)[Base de Datos de Modelos de Dispositivo](./device-models.md)

---