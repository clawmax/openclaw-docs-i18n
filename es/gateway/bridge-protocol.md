

  Protocolos y APIs

  
# Protocolo Bridge

El protocolo Bridge es un transporte de nodos **heredado** (TCP JSONL). Los nuevos clientes de nodo deben usar el protocolo unificado Gateway WebSocket en su lugar. Si estás construyendo un operador o cliente de nodo, usa el [protocolo Gateway](./protocol.md). **Nota:** Las compilaciones actuales de OpenClaw ya no incluyen el listener TCP bridge; este documento se mantiene como referencia histórica. Las claves de configuración heredadas `bridge.*` ya no forman parte del esquema de configuración.

## Por qué tenemos ambos

-   **Límite de seguridad**: el bridge expone una pequeña lista de permitidos en lugar de toda la superficie de la API del gateway.
-   **Emparejamiento + identidad del nodo**: la admisión de nodos es propiedad del gateway y está vinculada a un token por nodo.
-   **UX de descubrimiento**: los nodos pueden descubrir gateways vía Bonjour en la LAN, o conectarse directamente a través de una tailnet.
-   **WS de bucle local**: el plano de control WS completo permanece local a menos que se tunelice vía SSH.

## Transporte

-   TCP, un objeto JSON por línea (JSONL).
-   TLS opcional (cuando `bridge.tls.enabled` es verdadero).
-   El puerto de escucha heredado por defecto era `18790` (las compilaciones actuales no inician un TCP bridge).

Cuando TLS está habilitado, los registros TXT de descubrimiento incluyen `bridgeTls=1` más `bridgeTlsSha256` como una pista no secreta. Ten en cuenta que los registros TXT de Bonjour/mDNS no están autenticados; los clientes no deben tratar la huella digital anunciada como un pin autoritativo sin la intención explícita del usuario u otra verificación fuera de banda.

## Handshake + emparejamiento

1.  El cliente envía `hello` con metadatos del nodo + token (si ya está emparejado).
2.  Si no está emparejado, el gateway responde `error` (`NOT_PAIRED`/`UNAUTHORIZED`).
3.  El cliente envía `pair-request`.
4.  El gateway espera la aprobación, luego envía `pair-ok` y `hello-ok`.

`hello-ok` devuelve `serverName` y puede incluir `canvasHostUrl`.

## Tramas

Cliente → Gateway:

-   `req` / `res`: RPC del gateway con alcance (chat, sesiones, configuración, salud, voicewake, skills.bins)
-   `event`: señales del nodo (transcripción de voz, solicitud de agente, suscripción a chat, ciclo de vida de ejecución)

Gateway → Cliente:

-   `invoke` / `invoke-res`: comandos del nodo (`canvas.*`, `camera.*`, `screen.record`, `location.get`, `sms.send`)
-   `event`: actualizaciones de chat para sesiones suscritas
-   `ping` / `pong`: keepalive

La aplicación heredada de la lista de permitidos residía en `src/gateway/server-bridge.ts` (eliminada).

## Eventos del ciclo de vida de ejecución

Los nodos pueden emitir eventos `exec.finished` o `exec.denied` para mostrar la actividad de `system.run`. Estos se asignan a eventos del sistema en el gateway. (Los nodos heredados aún pueden emitir `exec.started`). Campos del payload (todos opcionales a menos que se indique):

-   `sessionKey` (requerido): sesión del agente que recibirá el evento del sistema.
-   `runId`: id único de ejecución para agrupar.
-   `command`: cadena de comando cruda o formateada.
-   `exitCode`, `timedOut`, `success`, `output`: detalles de finalización (solo para finished).
-   `reason`: motivo de denegación (solo para denied).

## Uso de Tailnet

-   Vincula el bridge a una IP de tailnet: `bridge.bind: "tailnet"` en `~/.openclaw/openclaw.json`.
-   Los clientes se conectan vía nombre MagicDNS o IP de tailnet.
-   Bonjour **no** cruza redes; usa host/puerto manual o DNS‑SD de área amplia cuando sea necesario.

## Control de versiones

Bridge es actualmente **v1 implícita** (sin negociación mín./máx.). Se espera compatibilidad hacia atrás; añade un campo de versión del protocolo bridge antes de cualquier cambio incompatible.

[Protocolo Gateway](./protocol.md)[Completaciones de Chat de OpenAI](./openai-http-api.md)

---