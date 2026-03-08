

  Protocolos y APIs

  
# Completaciones de Chat de OpenAI

El Gateway de OpenClaw puede servir un pequeño endpoint de Completaciones de Chat compatible con OpenAI. Este endpoint está **deshabilitado por defecto**. Habilítalo primero en la configuración.

-   `POST /v1/chat/completions`
-   Mismo puerto que el Gateway (WS + HTTP multiplexado): `http://<gateway-host>:/v1/chat/completions`

Internamente, las solicitudes se ejecutan como una ejecución normal de agente del Gateway (misma ruta de código que `openclaw agent`), por lo que el enrutamiento/permisos/configuración coinciden con tu Gateway.

## Autenticación

Utiliza la configuración de autenticación del Gateway. Envía un token bearer:

-   `Authorization: Bearer `

Notas:

-   Cuando `gateway.auth.mode="token"`, usa `gateway.auth.token` (o `OPENCLAW_GATEWAY_TOKEN`).
-   Cuando `gateway.auth.mode="password"`, usa `gateway.auth.password` (o `OPENCLAW_GATEWAY_PASSWORD`).
-   Si `gateway.auth.rateLimit` está configurado y ocurren demasiados fallos de autenticación, el endpoint devuelve `429` con `Retry-After`.

## Límite de seguridad (importante)

Trata este endpoint como una superficie de **acceso completo de operador** para la instancia del gateway.

-   La autenticación HTTP bearer aquí no es un modelo de alcance estrecho por usuario.
-   Un token/contraseña válido del Gateway para este endpoint debe tratarse como una credencial de propietario/operador.
-   Las solicitudes pasan por la misma ruta de agente del plano de control que las acciones de operador confiables.
-   No hay un límite de herramienta separado para no propietarios/por usuario en este endpoint; una vez que un llamante pasa la autenticación del Gateway aquí, OpenClaw trata a ese llamante como un operador confiable para este gateway.
-   Si la política del agente objetivo permite herramientas sensibles, este endpoint puede usarlas.
-   Mantén este endpoint solo en loopback/tailnet/ingreso privado; no lo expongas directamente a la internet pública.

Consulta [Seguridad](./security.md) y [Acceso remoto](./remote.md).

## Eligiendo un agente

No se requieren cabeceras personalizadas: codifica el id del agente en el campo `model` de OpenAI:

-   `model: "openclaw:"` (ejemplo: `"openclaw:main"`, `"openclaw:beta"`)
-   `model: "agent:"` (alias)

O apunta a un agente específico de OpenClaw por cabecera:

-   `x-openclaw-agent-id: ` (por defecto: `main`)

Avanzado:

-   `x-openclaw-session-key: ` para controlar completamente el enrutamiento de sesión.

## Habilitando el endpoint

Establece `gateway.http.endpoints.chatCompletions.enabled` en `true`:

```json
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: true },
      },
    },
  },
}
```

## Deshabilitando el endpoint

Establece `gateway.http.endpoints.chatCompletions.enabled` en `false`:

```json
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: false },
      },
    },
  },
}
```

## Comportamiento de sesión

Por defecto, el endpoint es **sin estado por solicitud** (se genera una nueva clave de sesión en cada llamada). Si la solicitud incluye una cadena `user` de OpenAI, el Gateway deriva una clave de sesión estable a partir de ella, por lo que las llamadas repetidas pueden compartir una sesión de agente.

## Streaming (SSE)

Establece `stream: true` para recibir Eventos Enviados por el Servidor (SSE):

-   `Content-Type: text/event-stream`
-   Cada línea de evento es `data: `
-   El stream termina con `data: [DONE]`

## Ejemplos

Sin streaming:

```bash
curl -sS http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "messages": [{"role":"user","content":"hi"}]
  }'
```

Con streaming:

```bash
curl -N http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "stream": true,
    "messages": [{"role":"user","content":"hi"}]
  }'
```

[Protocolo Bridge](./bridge-protocol.md)[API OpenResponses](./openresponses-http-api.md)