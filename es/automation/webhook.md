

  Automatización

  
# Webhooks

La pasarela puede exponer un pequeño endpoint HTTP de webhook para disparadores externos.

## Habilitar

```json
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    // Opcional: restringir el enrutamiento explícito de `agentId` a esta lista permitida.
    // Omite o incluye "*" para permitir cualquier agente.
    // Establece [] para denegar todo enrutamiento explícito de `agentId`.
    allowedAgentIds: ["hooks", "main"],
  },
}
```

Notas:

-   `hooks.token` es obligatorio cuando `hooks.enabled=true`.
-   `hooks.path` tiene como valor predeterminado `/hooks`.

## Autenticación

Cada solicitud debe incluir el token del webhook. Se prefieren los encabezados:

-   `Authorization: Bearer ` (recomendado)
-   `x-openclaw-token: `
-   Los tokens en la cadena de consulta son rechazados (`?token=...` devuelve `400`).

## Endpoints

### POST /hooks/wake

Carga útil:

```json
{ "text": "System line", "mode": "now" }
```

-   `text` **obligatorio** (string): La descripción del evento (ej., “Nuevo correo recibido”).
-   `mode` opcional (`now` | `next-heartbeat`): Si activar un latido inmediato (predeterminado `now`) o esperar a la siguiente comprobación periódica.

Efecto:

-   Encola un evento del sistema para la sesión **principal**
-   Si `mode=now`, activa un latido inmediato

### POST /hooks/agent

Carga útil:

```json
{
  "message": "Run this",
  "name": "Email",
  "agentId": "hooks",
  "sessionKey": "hook:email:msg-123",
  "wakeMode": "now",
  "deliver": true,
  "channel": "last",
  "to": "+15551234567",
  "model": "openai/gpt-5.2-mini",
  "thinking": "low",
  "timeoutSeconds": 120
}
```

-   `message` **obligatorio** (string): El mensaje o prompt para que el agente lo procese.
-   `name` opcional (string): Nombre legible para humanos del webhook (ej., “GitHub”), usado como prefijo en los resúmenes de sesión.
-   `agentId` opcional (string): Enruta este webhook a un agente específico. Los IDs desconocidos vuelven al agente predeterminado. Cuando se establece, el webhook se ejecuta usando el espacio de trabajo y configuración del agente resuelto.
-   `sessionKey` opcional (string): La clave usada para identificar la sesión del agente. Por defecto este campo es rechazado a menos que `hooks.allowRequestSessionKey=true`.
-   `wakeMode` opcional (`now` | `next-heartbeat`): Si activar un latido inmediato (predeterminado `now`) o esperar a la siguiente comprobación periódica.
-   `deliver` opcional (boolean): Si es `true`, la respuesta del agente se enviará al canal de mensajería. Por defecto es `true`. Las respuestas que son solo acuses de recibo de latido se omiten automáticamente.
-   `channel` opcional (string): El canal de mensajería para la entrega. Uno de: `last`, `whatsapp`, `telegram`, `discord`, `slack`, `mattermost` (plugin), `signal`, `imessage`, `msteams`. Por defecto es `last`.
-   `to` opcional (string): El identificador del destinatario para el canal (ej., número de teléfono para WhatsApp/Signal, ID de chat para Telegram, ID de canal para Discord/Slack/Mattermost (plugin), ID de conversación para MS Teams). Por defecto es el último destinatario en la sesión principal.
-   `model` opcional (string): Anulación del modelo (ej., `anthropic/claude-3-5-sonnet` o un alias). Debe estar en la lista de modelos permitidos si está restringida.
-   `thinking` opcional (string): Anulación del nivel de pensamiento (ej., `low`, `medium`, `high`).
-   `timeoutSeconds` opcional (number): Duración máxima para la ejecución del agente en segundos.

Efecto:

-   Ejecuta un turno de agente **aislado** (su propia clave de sesión)
-   Siempre publica un resumen en la sesión **principal**
-   Si `wakeMode=now`, activa un latido inmediato

## Política de clave de sesión (cambio incompatible)

Las anulaciones de `sessionKey` en la carga útil de `/hooks/agent` están deshabilitadas por defecto.

-   Recomendado: establecer una `hooks.defaultSessionKey` fija y mantener las anulaciones por solicitud desactivadas.
-   Opcional: permitir anulaciones por solicitud solo cuando sea necesario, y restringir los prefijos.

Configuración recomendada:

```json
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
  },
}
```

Configuración de compatibilidad (comportamiento heredado):

```json
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    allowRequestSessionKey: true,
    allowedSessionKeyPrefixes: ["hook:"], // muy recomendado
  },
}
```

### POST /hooks/&lt;nombre&gt; (mapeado)

Los nombres de webhook personalizados se resuelven mediante `hooks.mappings` (ver configuración). Un mapeo puede convertir cargas útiles arbitrarias en acciones `wake` o `agent`, con plantillas o transformaciones de código opcionales. Opciones de mapeo (resumen):

-   `hooks.presets: ["gmail"]` habilita el mapeo incorporado de Gmail.
-   `hooks.mappings` te permite definir `match`, `action` y plantillas en la configuración.
-   `hooks.transformsDir` + `transform.module` carga un módulo JS/TS para lógica personalizada.
    -   `hooks.transformsDir` (si se establece) debe permanecer dentro del directorio raíz de transformaciones bajo tu directorio de configuración de OpenClaw (típicamente `~/.openclaw/hooks/transforms`).
    -   `transform.module` debe resolverse dentro del directorio de transformaciones efectivo (las rutas de escape/traversal son rechazadas).
-   Usa `match.source` para mantener un endpoint de ingesta genérico (enrutamiento basado en carga útil).
-   Las transformaciones TS requieren un cargador TS (ej. `bun` o `tsx`) o un `.js` precompilado en tiempo de ejecución.
-   Establece `deliver: true` + `channel`/`to` en los mapeos para enrutar respuestas a una superficie de chat (`channel` por defecto es `last` y vuelve a WhatsApp).
-   `agentId` enruta el webhook a un agente específico; los IDs desconocidos vuelven al agente predeterminado.
-   `hooks.allowedAgentIds` restringe el enrutamiento explícito de `agentId`. Omítelo (o incluye `*`) para permitir cualquier agente. Establece `[]` para denegar el enrutamiento explícito de `agentId`.
-   `hooks.defaultSessionKey` establece la sesión predeterminada para las ejecuciones de agentes de webhook cuando no se proporciona una clave explícita.
-   `hooks.allowRequestSessionKey` controla si las cargas útiles de `/hooks/agent` pueden establecer `sessionKey` (predeterminado: `false`).
-   `hooks.allowedSessionKeyPrefixes` restringe opcionalmente los valores explícitos de `sessionKey` provenientes de cargas útiles de solicitud y mapeos.
-   `allowUnsafeExternalContent: true` desactiva el envoltorio de seguridad de contenido externo para ese webhook (peligroso; solo para fuentes internas confiables).
-   `openclaw webhooks gmail setup` escribe la configuración `hooks.gmail` para `openclaw webhooks gmail run`. Consulta [Gmail Pub/Sub](./gmail-pubsub.md) para el flujo completo de seguimiento de Gmail.

## Respuestas

-   `200` para `/hooks/wake`
-   `200` para `/hooks/agent` (ejecución asíncrona aceptada)
-   `401` en fallo de autenticación
-   `429` después de fallos de autenticación repetidos del mismo cliente (verifica `Retry-After`)
-   `400` en carga útil inválida
-   `413` en cargas útiles demasiado grandes

## Ejemplos

```bash
curl -X POST http://127.0.0.1:18789/hooks/wake \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"text":"New email received","mode":"now"}'
```

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","wakeMode":"next-heartbeat"}'
```

### Usar un modelo diferente

Añade `model` a la carga útil del agente (o al mapeo) para anular el modelo para esa ejecución:

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","model":"openai/gpt-5.2-mini"}'
```

Si aplicas `agents.defaults.models`, asegúrate de que el modelo de anulación esté incluido allí.

```bash
curl -X POST http://127.0.0.1:18789/hooks/gmail \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"source":"gmail","messages":[{"from":"Ada","subject":"Hello","snippet":"Hi"}]}'
```

## Seguridad

-   Mantén los endpoints de webhook detrás de loopback, tailnet o un proxy inverso confiable.
-   Usa un token de webhook dedicado; no reutilices tokens de autenticación de la pasarela.
-   Los fallos de autenticación repetidos son limitados por tasa por dirección de cliente para ralentizar intentos de fuerza bruta.
-   Si usas enrutamiento multi-agente, establece `hooks.allowedAgentIds` para limitar la selección explícita de `agentId`.
-   Mantén `hooks.allowRequestSessionKey=false` a menos que requieras sesiones seleccionadas por el llamante.
-   Si habilitas `sessionKey` por solicitud, restringe `hooks.allowedSessionKeyPrefixes` (por ejemplo, `["hook:"]`).
-   Evita incluir cargas útiles sensibles en bruto en los registros de webhooks.
-   Las cargas útiles de webhook se tratan como no confiables y se envuelven con límites de seguridad por defecto. Si debes desactivar esto para un webhook específico, establece `allowUnsafeExternalContent: true` en el mapeo de ese webhook (peligroso).

[Solución de Problemas de Automatización](./troubleshooting.md)[Gmail PubSub](./gmail-pubsub.md)