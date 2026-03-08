

  Protocolos y APIs

  
# API de Invocación de Herramientas

El Gateway de OpenClaw expone un endpoint HTTP simple para invocar una sola herramienta directamente. Siempre está habilitado, pero protegido por la autenticación del Gateway y las políticas de herramientas.

-   `POST /tools/invoke`
-   Mismo puerto que el Gateway (WS + HTTP multiplexado): `http://<gateway-host>:/tools/invoke`

El tamaño máximo de carga útil por defecto es de 2 MB.

## Autenticación

Utiliza la configuración de autenticación del Gateway. Envía un token bearer:

-   `Authorization: Bearer `

Notas:

-   Cuando `gateway.auth.mode="token"`, usa `gateway.auth.token` (o `OPENCLAW_GATEWAY_TOKEN`).
-   Cuando `gateway.auth.mode="password"`, usa `gateway.auth.password` (o `OPENCLAW_GATEWAY_PASSWORD`).
-   Si `gateway.auth.rateLimit` está configurado y ocurren demasiados fallos de autenticación, el endpoint devuelve `429` con `Retry-After`.

## Cuerpo de la solicitud

```json
{
  "tool": "sessions_list",
  "action": "json",
  "args": {},
  "sessionKey": "main",
  "dryRun": false
}
```

Campos:

-   `tool` (string, requerido): nombre de la herramienta a invocar.
-   `action` (string, opcional): se mapea en args si el esquema de la herramienta soporta `action` y el payload de args lo omitió.
-   `args` (objeto, opcional): argumentos específicos de la herramienta.
-   `sessionKey` (string, opcional): clave de sesión objetivo. Si se omite o es `"main"`, el Gateway usa la clave de sesión principal configurada (respeta `session.mainKey` y el agente por defecto, o `global` en el ámbito global).
-   `dryRun` (booleano, opcional): reservado para uso futuro; actualmente ignorado.

## Comportamiento de políticas y enrutamiento

La disponibilidad de herramientas se filtra a través de la misma cadena de políticas utilizada por los agentes del Gateway:

-   `tools.profile` / `tools.byProvider.profile`
-   `tools.allow` / `tools.byProvider.allow`
-   `agents..tools.allow` / `agents..tools.byProvider.allow`
-   políticas de grupo (si la clave de sesión se asigna a un grupo o canal)
-   política de subagente (al invocar con una clave de sesión de subagente)

Si una herramienta no está permitida por la política, el endpoint devuelve **404**. El HTTP del Gateway también aplica una lista de denegación estricta por defecto (incluso si la política de sesión permite la herramienta):

-   `sessions_spawn`
-   `sessions_send`
-   `gateway`
-   `whatsapp_login`

Puedes personalizar esta lista de denegación a través de `gateway.tools`:

```json
{
  gateway: {
    tools: {
      // Herramientas adicionales para bloquear sobre HTTP /tools/invoke
      deny: ["browser"],
      // Eliminar herramientas de la lista de denegación por defecto
      allow: ["gateway"],
    },
  },
}
```

Para ayudar a que las políticas de grupo resuelvan el contexto, puedes configurar opcionalmente:

-   `x-openclaw-message-channel: ` (ejemplo: `slack`, `telegram`)
-   `x-openclaw-account-id: ` (cuando existen múltiples cuentas)

## Respuestas

-   `200` → `{ ok: true, result }`
-   `400` → `{ ok: false, error: { type, message } }` (solicitud inválida o error de entrada de herramienta)
-   `401` → no autorizado
-   `429` → límite de tasa de autenticación (`Retry-After` establecido)
-   `404` → herramienta no disponible (no encontrada o no en la lista de permitidos)
-   `405` → método no permitido
-   `500` → `{ ok: false, error: { type, message } }` (error inesperado de ejecución de herramienta; mensaje sanitizado)

## Ejemplo

```bash
curl -sS http://127.0.0.1:18789/tools/invoke \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
    "tool": "sessions_list",
    "action": "json",
    "args": {}
  }'
```

[API OpenResponses](./openresponses-http-api.md)[Backends CLI](./cli-backends.md)

---