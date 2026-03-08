

  Plataformas de mensajería

  
# LINE

LINE se conecta a OpenClaw a través de la API de Mensajería de LINE. El plugin funciona como un receptor de webhook en la puerta de enlace y utiliza tu token de acceso del canal + secreto del canal para la autenticación. Estado: compatible mediante plugin. Se admiten mensajes directos, chats grupales, medios, ubicaciones, mensajes Flex, mensajes de plantilla y respuestas rápidas. No se admiten reacciones ni hilos.

## Plugin requerido

Instala el plugin de LINE:

```bash
openclaw plugins install @openclaw/line
```

Instalación local (cuando se ejecuta desde un repositorio git):

```bash
openclaw plugins install ./extensions/line
```

## Configuración

1.  Crea una cuenta de LINE Developers y abre la Consola: [https://developers.line.biz/console/](https://developers.line.biz/console/)
2.  Crea (o selecciona) un Proveedor y añade un canal de **Messaging API**.
3.  Copia el **Channel access token** y el **Channel secret** desde la configuración del canal.
4.  Habilita **Use webhook** en la configuración de la API de Mensajería.
5.  Establece la URL del webhook a tu endpoint de la puerta de enlace (se requiere HTTPS):

```
https://gateway-host/line/webhook
```

La puerta de enlace responde a la verificación del webhook de LINE (GET) y a los eventos entrantes (POST). Si necesitas una ruta personalizada, configura `channels.line.webhookPath` o `channels.line.accounts..webhookPath` y actualiza la URL en consecuencia. Nota de seguridad:

-   La verificación de firma de LINE depende del cuerpo (HMAC sobre el cuerpo crudo), por lo que OpenClaw aplica límites estrictos de tamaño de cuerpo y tiempo de espera antes de la verificación.

## Configurar

Configuración mínima:

```json
{
  channels: {
    line: {
      enabled: true,
      channelAccessToken: "LINE_CHANNEL_ACCESS_TOKEN",
      channelSecret: "LINE_CHANNEL_SECRET",
      dmPolicy: "pairing",
    },
  },
}
```

Variables de entorno (solo cuenta por defecto):

-   `LINE_CHANNEL_ACCESS_TOKEN`
-   `LINE_CHANNEL_SECRET`

Archivos de token/secreto:

```json
{
  channels: {
    line: {
      tokenFile: "/path/to/line-token.txt",
      secretFile: "/path/to/line-secret.txt",
    },
  },
}
```

Múltiples cuentas:

```json
{
  channels: {
    line: {
      accounts: {
        marketing: {
          channelAccessToken: "...",
          channelSecret: "...",
          webhookPath: "/line/marketing",
        },
      },
    },
  },
}
```

## Control de acceso

Los mensajes directos por defecto requieren emparejamiento. Los remitentes desconocidos reciben un código de emparejamiento y sus mensajes se ignoran hasta que sean aprobados.

```bash
openclaw pairing list line
openclaw pairing approve line <CODE>
```

Listas de permitidos y políticas:

-   `channels.line.dmPolicy`: `pairing | allowlist | open | disabled`
-   `channels.line.allowFrom`: IDs de usuario de LINE permitidos para mensajes directos
-   `channels.line.groupPolicy`: `allowlist | open | disabled`
-   `channels.line.groupAllowFrom`: IDs de usuario de LINE permitidos para grupos
-   Anulaciones por grupo: `channels.line.groups..allowFrom`
-   Nota de tiempo de ejecución: si `channels.line` falta completamente, el tiempo de ejecución recurre a `groupPolicy="allowlist"` para las comprobaciones de grupo (incluso si `channels.defaults.groupPolicy` está configurado).

Los IDs de LINE distinguen entre mayúsculas y minúsculas. Los IDs válidos tienen este aspecto:

-   Usuario: `U` + 32 caracteres hexadecimales
-   Grupo: `C` + 32 caracteres hexadecimales
-   Sala: `R` + 32 caracteres hexadecimales

## Comportamiento de los mensajes

-   El texto se divide en fragmentos de 5000 caracteres.
-   El formato Markdown se elimina; los bloques de código y las tablas se convierten en tarjetas Flex cuando es posible.
-   Las respuestas en flujo se almacenan en búfer; LINE recibe fragmentos completos con una animación de carga mientras el agente trabaja.
-   Las descargas de medios están limitadas por `channels.line.mediaMaxMb` (por defecto 10).

## Datos del canal (mensajes enriquecidos)

Usa `channelData.line` para enviar respuestas rápidas, ubicaciones, tarjetas Flex o mensajes de plantilla.

```json
{
  text: "Aquí tienes",
  channelData: {
    line: {
      quickReplies: ["Estado", "Ayuda"],
      location: {
        title: "Oficina",
        address: "123 Calle Principal",
        latitude: 35.681236,
        longitude: 139.767125,
      },
      flexMessage: {
        altText: "Tarjeta de estado",
        contents: {
          /* Payload Flex */
        },
      },
      templateMessage: {
        type: "confirm",
        text: "¿Continuar?",
        confirmLabel: "Sí",
        confirmData: "yes",
        cancelLabel: "No",
        cancelData: "no",
      },
    },
  },
}
```

El plugin de LINE también incluye un comando `/card` para plantillas de mensajes Flex:

```bash
/card info "Bienvenido" "¡Gracias por unirte!"
```

## Solución de problemas

-   **Falla la verificación del webhook:** asegúrate de que la URL del webhook sea HTTPS y de que `channelSecret` coincida con la consola de LINE.
-   **No hay eventos entrantes:** confirma que la ruta del webhook coincida con `channels.line.webhookPath` y que la puerta de enlace sea accesible desde LINE.
-   **Errores al descargar medios:** aumenta `channels.line.mediaMaxMb` si los medios superan el límite por defecto.

[IRC](./irc.md)[Matrix](./matrix.md)

---