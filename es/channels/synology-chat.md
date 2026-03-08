

  Plataformas de mensajería

  
# Synology Chat

Estado: compatible a través de complemento como canal de mensajes directos utilizando webhooks de Synology Chat. El complemento acepta mensajes entrantes de webhooks salientes de Synology Chat y envía respuestas a través de un webhook entrante de Synology Chat.

## Complemento requerido

Synology Chat se basa en complementos y no forma parte de la instalación predeterminada del canal principal. Instala desde una copia local:

```bash
openclaw plugins install ./extensions/synology-chat
```

Detalles: [Complementos](../tools/plugin.md)

## Configuración rápida

1.  Instala y habilita el complemento Synology Chat.
2.  En las integraciones de Synology Chat:
    -   Crea un webhook entrante y copia su URL.
    -   Crea un webhook saliente con tu token secreto.
3.  Apunta la URL del webhook saliente a tu puerta de enlace OpenClaw:
    -   `https://gateway-host/webhook/synology` por defecto.
    -   O tu `channels.synology-chat.webhookPath` personalizado.
4.  Configura `channels.synology-chat` en OpenClaw.
5.  Reinicia la puerta de enlace y envía un MD al bot de Synology Chat.

Configuración mínima:

```json
{
  channels: {
    "synology-chat": {
      enabled: true,
      token: "synology-outgoing-token",
      incomingUrl: "https://nas.example.com/webapi/entry.cgi?api=SYNO.Chat.External&method=incoming&version=2&token=...",
      webhookPath: "/webhook/synology",
      dmPolicy: "allowlist",
      allowedUserIds: ["123456"],
      rateLimitPerMinute: 30,
      allowInsecureSsl: false,
    },
  },
}
```

## Variables de entorno

Para la cuenta predeterminada, puedes usar variables de entorno:

-   `SYNOLOGY_CHAT_TOKEN`
-   `SYNOLOGY_CHAT_INCOMING_URL`
-   `SYNOLOGY_NAS_HOST`
-   `SYNOLOGY_ALLOWED_USER_IDS` (separados por comas)
-   `SYNOLOGY_RATE_LIMIT`
-   `OPENCLAW_BOT_NAME`

Los valores de configuración anulan las variables de entorno.

## Política de MD y control de acceso

-   `dmPolicy: "allowlist"` es el valor predeterminado recomendado.
-   `allowedUserIds` acepta una lista (o cadena separada por comas) de ID de usuario de Synology.
-   En modo `allowlist`, una lista `allowedUserIds` vacía se trata como una mala configuración y la ruta del webhook no se iniciará (usa `dmPolicy: "open"` para permitir a todos).
-   `dmPolicy: "open"` permite cualquier remitente.
-   `dmPolicy: "disabled"` bloquea los MD.
-   Las aprobaciones de emparejamiento funcionan con:
    -   `openclaw pairing list synology-chat`
    -   `openclaw pairing approve synology-chat `

## Entrega saliente

Usa ID de usuario numéricos de Synology Chat como destinos. Ejemplos:

```bash
openclaw message send --channel synology-chat --target 123456 --text "Hello from OpenClaw"
openclaw message send --channel synology-chat --target synology-chat:123456 --text "Hello again"
```

Los envíos de medios son compatibles mediante la entrega de archivos basada en URL.

## Múltiples cuentas

Se admiten múltiples cuentas de Synology Chat bajo `channels.synology-chat.accounts`. Cada cuenta puede anular el token, la URL entrante, la ruta del webhook, la política de MD y los límites.

```json
{
  channels: {
    "synology-chat": {
      enabled: true,
      accounts: {
        default: {
          token: "token-a",
          incomingUrl: "https://nas-a.example.com/...token=...",
        },
        alerts: {
          token: "token-b",
          incomingUrl: "https://nas-b.example.com/...token=...",
          webhookPath: "/webhook/synology-alerts",
          dmPolicy: "allowlist",
          allowedUserIds: ["987654"],
        },
      },
    },
  },
}
```

## Notas de seguridad

-   Mantén el `token` en secreto y rótalo si se filtra.
-   Mantén `allowInsecureSsl: false` a menos que confíes explícitamente en un certificado local de NAS autofirmado.
-   Las solicitudes de webhook entrantes se verifican por token y tienen límite de tasa por remitente.
-   Prefiere `dmPolicy: "allowlist"` para producción.

[Signal](./signal.md)[Slack](./slack.md)

---