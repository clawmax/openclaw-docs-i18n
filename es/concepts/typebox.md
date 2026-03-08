

  Conceptos internos

  
# TypeBox

Última actualización: 2026-01-10 TypeBox es una biblioteca de esquemas con prioridad en TypeScript. La usamos para definir el **protocolo WebSocket de Gateway** (handshake, solicitud/respuesta, eventos del servidor). Esos esquemas impulsan la **validación en tiempo de ejecución**, la **exportación de JSON Schema** y la **generación de código Swift** para la aplicación macOS. Una única fuente de verdad; todo lo demás se genera. Si quieres el contexto de protocolo de alto nivel, comienza con [Arquitectura de Gateway](./architecture.md).

## Modelo mental (30 segundos)

Cada mensaje de Gateway WS es uno de tres tipos de trama:

-   **Solicitud**: `{ type: "req", id, method, params }`
-   **Respuesta**: `{ type: "res", id, ok, payload | error }`
-   **Evento**: `{ type: "event", event, payload, seq?, stateVersion? }`

La primera trama **debe** ser una solicitud `connect`. Después de eso, los clientes pueden llamar métodos (ej. `health`, `send`, `chat.send`) y suscribirse a eventos (ej. `presence`, `tick`, `agent`). Flujo de conexión (mínimo):

```
Cliente                    Gateway
  |---- req:connect -------->|
  |<---- res:hello-ok --------|
  |<---- event:tick ----------|
  |---- req:health ---------->|
  |<---- res:health ----------|
```

Métodos + eventos comunes:

| Categoría | Ejemplos | Notas |
| --- | --- | --- |
| Núcleo | `connect`, `health`, `status` | `connect` debe ser el primero |
| Mensajería | `send`, `poll`, `agent`, `agent.wait` | los efectos secundarios necesitan `idempotencyKey` |
| Chat | `chat.history`, `chat.send`, `chat.abort`, `chat.inject` | WebChat usa estos |
| Sesiones | `sessions.list`, `sessions.patch`, `sessions.delete` | administración de sesiones |
| Nodos | `node.list`, `node.invoke`, `node.pair.*` | acciones de Gateway WS + nodos |
| Eventos | `tick`, `presence`, `agent`, `chat`, `health`, `shutdown` | push del servidor |

La lista autoritativa reside en `src/gateway/server.ts` (`METHODS`, `EVENTS`).

## Dónde viven los esquemas

-   Fuente: `src/gateway/protocol/schema.ts`
-   Validadores en tiempo de ejecución (AJV): `src/gateway/protocol/index.ts`
-   Handshake del servidor + despacho de métodos: `src/gateway/server.ts`
-   Cliente Node: `src/gateway/client.ts`
-   JSON Schema generado: `dist/protocol.schema.json`
-   Modelos Swift generados: `apps/macos/Sources/OpenClawProtocol/GatewayModels.swift`

## Pipeline actual

-   `pnpm protocol:gen`
    -   escribe JSON Schema (draft‑07) en `dist/protocol.schema.json`
-   `pnpm protocol:gen:swift`
    -   genera modelos Swift de gateway
-   `pnpm protocol:check`
    -   ejecuta ambos generadores y verifica que la salida esté confirmada

## Cómo se usan los esquemas en tiempo de ejecución

-   **Lado del servidor**: cada trama entrante se valida con AJV. El handshake solo acepta una solicitud `connect` cuyos parámetros coincidan con `ConnectParams`.
-   **Lado del cliente**: el cliente JS valida las tramas de evento y respuesta antes de usarlas.
-   **Superficie de métodos**: el Gateway anuncia los `methods` y `events` admitidos en `hello-ok`.

## Ejemplos de tramas

Connect (primer mensaje):

```json
{
  "type": "req",
  "id": "c1",
  "method": "connect",
  "params": {
    "minProtocol": 2,
    "maxProtocol": 2,
    "client": {
      "id": "openclaw-macos",
      "displayName": "macos",
      "version": "1.0.0",
      "platform": "macos 15.1",
      "mode": "ui",
      "instanceId": "A1B2"
    }
  }
}
```

Respuesta Hello-ok:

```json
{
  "type": "res",
  "id": "c1",
  "ok": true,
  "payload": {
    "type": "hello-ok",
    "protocol": 2,
    "server": { "version": "dev", "connId": "ws-1" },
    "features": { "methods": ["health"], "events": ["tick"] },
    "snapshot": {
      "presence": [],
      "health": {},
      "stateVersion": { "presence": 0, "health": 0 },
      "uptimeMs": 0
    },
    "policy": { "maxPayload": 1048576, "maxBufferedBytes": 1048576, "tickIntervalMs": 30000 }
  }
}
```

Solicitud + respuesta:

```json
{ "type": "req", "id": "r1", "method": "health" }
```

```json
{ "type": "res", "id": "r1", "ok": true, "payload": { "ok": true } }
```

Evento:

```json
{ "type": "event", "event": "tick", "payload": { "ts": 1730000000 }, "seq": 12 }
```

## Cliente mínimo (Node.js)

Flujo útil más pequeño: connect + health.

```typescript
import { WebSocket } from "ws";

const ws = new WebSocket("ws://127.0.0.1:18789");

ws.on("open", () => {
  ws.send(
    JSON.stringify({
      type: "req",
      id: "c1",
      method: "connect",
      params: {
        minProtocol: 3,
        maxProtocol: 3,
        client: {
          id: "cli",
          displayName: "example",
          version: "dev",
          platform: "node",
          mode: "cli",
        },
      },
    }),
  );
});

ws.on("message", (data) => {
  const msg = JSON.parse(String(data));
  if (msg.type === "res" && msg.id === "c1" && msg.ok) {
    ws.send(JSON.stringify({ type: "req", id: "h1", method: "health" }));
  }
  if (msg.type === "res" && msg.id === "h1") {
    console.log("health:", msg.payload);
    ws.close();
  }
});
```

## Ejemplo práctico: agregar un método de extremo a extremo

Ejemplo: agregar una nueva solicitud `system.echo` que devuelva `{ ok: true, text }`.

1.  **Esquema (fuente de verdad)**

Agregar a `src/gateway/protocol/schema.ts`:

```bash
export const SystemEchoParamsSchema = Type.Object(
  { text: NonEmptyString },
  { additionalProperties: false },
);

export const SystemEchoResultSchema = Type.Object(
  { ok: Type.Boolean(), text: NonEmptyString },
  { additionalProperties: false },
);
```

Agregar ambos a `ProtocolSchemas` y exportar tipos:

```yaml
SystemEchoParams: SystemEchoParamsSchema,
  SystemEchoResult: SystemEchoResultSchema,
```

```bash
export type SystemEchoParams = Static<typeof SystemEchoParamsSchema>;
export type SystemEchoResult = Static<typeof SystemEchoResultSchema>;
```

2.  **Validación**

En `src/gateway/protocol/index.ts`, exportar un validador AJV:

```bash
export const validateSystemEchoParams = ajv.compile<SystemEchoParams>(SystemEchoParamsSchema);
```

3.  **Comportamiento del servidor**

Agregar un manejador en `src/gateway/server-methods/system.ts`:

```bash
export const systemHandlers: GatewayRequestHandlers = {
  "system.echo": ({ params, respond }) => {
    const text = String(params.text ?? "");
    respond(true, { ok: true, text });
  },
};
```

Registrarlo en `src/gateway/server-methods.ts` (ya fusiona `systemHandlers`), luego agregar `"system.echo"` a `METHODS` en `src/gateway/server.ts`.

4.  **Regenerar**

```bash
pnpm protocol:check
```

5.  **Pruebas + documentación**

Agregar una prueba de servidor en `src/gateway/server.*.test.ts` y anotar el método en la documentación.

## Comportamiento de la generación de código Swift

El generador Swift emite:

-   Enumeración `GatewayFrame` con casos `req`, `res`, `event` y `unknown`
-   Estructuras/enumeraciones de carga útil fuertemente tipadas
-   Valores `ErrorCode` y `GATEWAY_PROTOCOL_VERSION`

Los tipos de trama desconocidos se conservan como cargas útiles sin procesar para compatibilidad futura.

## Control de versiones + compatibilidad

-   `PROTOCOL_VERSION` reside en `src/gateway/protocol/schema.ts`.
-   Los clientes envían `minProtocol` + `maxProtocol`; el servidor rechaza desajustes.
-   Los modelos Swift mantienen tipos de trama desconocidos para evitar romper clientes antiguos.

## Patrones y convenciones de esquemas

-   La mayoría de los objetos usan `additionalProperties: false` para cargas útiles estrictas.
-   `NonEmptyString` es el valor por defecto para IDs y nombres de métodos/eventos.
-   El `GatewayFrame` de nivel superior usa un **discriminador** en `type`.
-   Los métodos con efectos secundarios generalmente requieren una `idempotencyKey` en los parámetros (ejemplo: `send`, `poll`, `agent`, `chat.send`).
-   `agent` acepta `internalEvents` opcionales para contexto de orquestación generado en tiempo de ejecución (por ejemplo, transferencia de finalización de subtareas/cron); trata esto como superficie de API interna.

## JSON Schema en vivo

El JSON Schema generado está en el repositorio en `dist/protocol.schema.json`. El archivo raw publicado está típicamente disponible en:

-   [https://raw.githubusercontent.com/openclaw/openclaw/main/dist/protocol.schema.json](https://raw.githubusercontent.com/openclaw/openclaw/main/dist/protocol.schema.json)

## Cuando cambias los esquemas

1.  Actualiza los esquemas TypeBox.
2.  Ejecuta `pnpm protocol:check`.
3.  Confirma el esquema regenerado + los modelos Swift.

[Fecha y Hora](../date-time.md)[Formato Markdown](./markdown-formatting.md)