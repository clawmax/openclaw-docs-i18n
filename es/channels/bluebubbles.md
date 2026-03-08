

  Plataformas de mensajería

  
# BlueBubbles

Estado: plugin incluido que se comunica con el servidor BlueBubbles de macOS a través de HTTP. **Recomendado para integración con iMessage** debido a su API más rica y configuración más fácil en comparación con el canal heredado imsg.

## Descripción general

-   Se ejecuta en macOS a través de la aplicación auxiliar BlueBubbles ([bluebubbles.app](https://bluebubbles.app)).
-   Recomendado/probado: macOS Sequoia (15). macOS Tahoe (26) funciona; la edición actualmente está rota en Tahoe, y las actualizaciones de iconos de grupo pueden reportar éxito pero no sincronizarse.
-   OpenClaw se comunica con él a través de su API REST (`GET /api/v1/ping`, `POST /message/text`, `POST /chat/:id/*`).
-   Los mensajes entrantes llegan a través de webhooks; las respuestas salientes, indicadores de escritura, confirmaciones de lectura y tapbacks son llamadas REST.
-   Los archivos adjuntos y pegatinas se ingieren como medios entrantes (y se muestran al agente cuando es posible).
-   El emparejamiento/lista de permitidos funciona de la misma manera que otros canales (`/channels/pairing` etc) con `channels.bluebubbles.allowFrom` + códigos de emparejamiento.
-   Las reacciones se muestran como eventos del sistema al igual que en Slack/Telegram para que los agentes puedan "mencionarlas" antes de responder.
-   Funciones avanzadas: editar, eliminar, respuesta en hilo, efectos de mensaje, gestión de grupos.

## Inicio rápido

1.  Instala el servidor BlueBubbles en tu Mac (sigue las instrucciones en [bluebubbles.app/install](https://bluebubbles.app/install)).
2.  En la configuración de BlueBubbles, habilita la API web y establece una contraseña.
3.  Ejecuta `openclaw onboard` y selecciona BlueBubbles, o configura manualmente:
    
    Copiar
    
    ```json
    {
      channels: {
        bluebubbles: {
          enabled: true,
          serverUrl: "http://192.168.1.100:1234",
          password: "example-password",
          webhookPath: "/bluebubbles-webhook",
        },
      },
    }
    ```
    
4.  Dirige los webhooks de BlueBubbles a tu puerta de enlace (ejemplo: `https://your-gateway-host:3000/bluebubbles-webhook?password=`).
5.  Inicia la puerta de enlace; registrará el manejador del webhook y comenzará el emparejamiento.

Nota de seguridad:

-   Siempre establece una contraseña para el webhook.
-   La autenticación del webhook siempre es requerida. OpenClaw rechaza las solicitudes de webhook de BlueBubbles a menos que incluyan una contraseña/guid que coincida con `channels.bluebubbles.password` (por ejemplo `?password=` o `x-password`), independientemente de la topología de bucle local/proxy.
-   La autenticación por contraseña se verifica antes de leer/analizar los cuerpos completos de los webhooks.

## Mantener Messages.app activo (configuraciones VM / sin cabeza)

Algunas configuraciones de VM de macOS / siempre activas pueden terminar con Messages.app en estado "inactivo" (los eventos entrantes se detienen hasta que la aplicación se abre/se pone en primer plano). Una solución simple es **activar Messages cada 5 minutos** usando un AppleScript + LaunchAgent.

### 1) Guarda el AppleScript

Guarda esto como:

-   `~/Scripts/poke-messages.scpt`

Script de ejemplo (no interactivo; no roba el foco):

```
try
  tell application "Messages"
    if not running then
      launch
    end if

    -- Toca la interfaz de scripting para mantener el proceso receptivo.
    set _chatCount to (count of chats)
  end tell
on error
  -- Ignora fallos transitorios (avisos de primera ejecución, sesión bloqueada, etc.).
end try
```

### 2) Instala un LaunchAgent

Guarda esto como:

-   `~/Library/LaunchAgents/com.user.poke-messages.plist`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>com.user.poke-messages</string>

    <key>ProgramArguments</key>
    <array>
      <string>/bin/bash</string>
      <string>-lc</string>
      <string>/usr/bin/osascript &quot;$HOME/Scripts/poke-messages.scpt&quot;</string>
    </array>

    <key>RunAtLoad</key>
    <true/>

    <key>StartInterval</key>
    <integer>300</integer>

    <key>StandardOutPath</key>
    <string>/tmp/poke-messages.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/poke-messages.err</string>
  </dict>
</plist>
```

Notas:

-   Esto se ejecuta **cada 300 segundos** y **al iniciar sesión**.
-   La primera ejecución puede activar avisos de **Automatización** de macOS (`osascript` → Messages). Aprueba en la misma sesión de usuario que ejecuta el LaunchAgent.

Cárgalo:

```bash
launchctl unload ~/Library/LaunchAgents/com.user.poke-messages.plist 2>/dev/null || true
launchctl load ~/Library/LaunchAgents/com.user.poke-messages.plist
```

## Incorporación

BlueBubbles está disponible en el asistente de configuración interactivo:

```bash
openclaw onboard
```

El asistente solicita:

-   **URL del servidor** (requerido): Dirección del servidor BlueBubbles (ej., `http://192.168.1.100:1234`)
-   **Contraseña** (requerido): Contraseña de la API desde la configuración del Servidor BlueBubbles
-   **Ruta del webhook** (opcional): Por defecto `/bluebubbles-webhook`
-   **Política de MD**: emparejamiento, lista de permitidos, abierto o deshabilitado
-   **Lista de permitidos**: Números de teléfono, correos electrónicos o destinos de chat

También puedes añadir BlueBubbles vía CLI:

```bash
openclaw channels add bluebubbles --http-url http://192.168.1.100:1234 --password <password>
```

## Control de acceso (MD + grupos)

MDs:

-   Por defecto: `channels.bluebubbles.dmPolicy = "pairing"`.
-   Los remitentes desconocidos reciben un código de emparejamiento; los mensajes se ignoran hasta ser aprobados (los códigos expiran después de 1 hora).
-   Aprueba mediante:
    -   `openclaw pairing list bluebubbles`
    -   `openclaw pairing approve bluebubbles `
-   El emparejamiento es el intercambio de tokens por defecto. Detalles: [Emparejamiento](./pairing.md)

Grupos:

-   `channels.bluebubbles.groupPolicy = open | allowlist | disabled` (por defecto: `allowlist`).
-   `channels.bluebubbles.groupAllowFrom` controla quién puede activar en grupos cuando `allowlist` está establecido.

### Control por mención (grupos)

BlueBubbles soporta control por mención para chats grupales, coincidiendo con el comportamiento de iMessage/WhatsApp:

-   Usa `agents.list[].groupChat.mentionPatterns` (o `messages.groupChat.mentionPatterns`) para detectar menciones.
-   Cuando `requireMention` está habilitado para un grupo, el agente solo responde cuando es mencionado.
-   Los comandos de control de remitentes autorizados omiten el control por mención.

Configuración por grupo:

```json
{
  channels: {
    bluebubbles: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true }, // valor por defecto para todos los grupos
        "iMessage;-;chat123": { requireMention: false }, // anulación para un grupo específico
      },
    },
  },
}
```

### Control de comandos

-   Los comandos de control (ej., `/config`, `/model`) requieren autorización.
-   Usa `allowFrom` y `groupAllowFrom` para determinar la autorización de comandos.
-   Los remitentes autorizados pueden ejecutar comandos de control incluso sin mencionar en grupos.

## Indicadores de escritura + confirmaciones de lectura

-   **Indicadores de escritura**: Enviados automáticamente antes y durante la generación de la respuesta.
-   **Confirmaciones de lectura**: Controladas por `channels.bluebubbles.sendReadReceipts` (por defecto: `true`).
-   **Indicadores de escritura**: OpenClaw envía eventos de inicio de escritura; BlueBubbles limpia la escritura automáticamente al enviar o por tiempo de espera (la parada manual vía DELETE no es confiable).

```json
{
  channels: {
    bluebubbles: {
      sendReadReceipts: false, // deshabilitar confirmaciones de lectura
    },
  },
}
```

## Acciones avanzadas

BlueBubbles soporta acciones de mensaje avanzadas cuando están habilitadas en la configuración:

```json
{
  channels: {
    bluebubbles: {
      actions: {
        reactions: true, // tapbacks (por defecto: true)
        edit: true, // editar mensajes enviados (macOS 13+, roto en macOS 26 Tahoe)
        unsend: true, // eliminar mensajes (macOS 13+)
        reply: true, // respuesta en hilo por GUID de mensaje
        sendWithEffect: true, // efectos de mensaje (slam, loud, etc.)
        renameGroup: true, // renombrar chats grupales
        setGroupIcon: true, // establecer icono/foto del chat grupal (inestable en macOS 26 Tahoe)
        addParticipant: true, // añadir participantes a grupos
        removeParticipant: true, // eliminar participantes de grupos
        leaveGroup: true, // abandonar chats grupales
        sendAttachment: true, // enviar archivos adjuntos/media
      },
    },
  },
}
```

Acciones disponibles:

-   **react**: Añadir/eliminar reacciones tapback (`messageId`, `emoji`, `remove`)
-   **edit**: Editar un mensaje enviado (`messageId`, `text`)
-   **unsend**: Eliminar un mensaje (`messageId`)
-   **reply**: Responder a un mensaje específico (`messageId`, `text`, `to`)
-   **sendWithEffect**: Enviar con efecto de iMessage (`text`, `to`, `effectId`)
-   **renameGroup**: Renombrar un chat grupal (`chatGuid`, `displayName`)
-   **setGroupIcon**: Establecer el icono/foto de un chat grupal (`chatGuid`, `media`) — inestable en macOS 26 (Tahoe) (la API puede devolver éxito pero el icono no se sincroniza).
-   **addParticipant**: Añadir alguien a un grupo (`chatGuid`, `address`)
-   **removeParticipant**: Eliminar a alguien de un grupo (`chatGuid`, `address`)
-   **leaveGroup**: Abandonar un chat grupal (`chatGuid`)
-   **sendAttachment**: Enviar medios/archivos (`to`, `buffer`, `filename`, `asVoice`)
    -   Notas de voz: establece `asVoice: true` con audio **MP3** o **CAF** para enviar como mensaje de voz de iMessage. BlueBubbles convierte MP3 → CAF al enviar notas de voz.

### IDs de mensaje (cortos vs completos)

OpenClaw puede mostrar IDs de mensaje *cortos* (ej., `1`, `2`) para ahorrar tokens.

-   `MessageSid` / `ReplyToId` pueden ser IDs cortos.
-   `MessageSidFull` / `ReplyToIdFull` contienen los IDs completos del proveedor.
-   Los IDs cortos están en memoria; pueden expirar al reiniciar o por expulsión de la caché.
-   Las acciones aceptan `messageId` corto o completo, pero los IDs cortos darán error si ya no están disponibles.

Usa IDs completos para automatizaciones y almacenamiento duraderos:

-   Plantillas: `{{MessageSidFull}}`, `{{ReplyToIdFull}}`
-   Contexto: `MessageSidFull` / `ReplyToIdFull` en las cargas útiles entrantes

Ver [Configuración](../gateway/configuration.md) para variables de plantilla.

## Transmisión por bloques

Controla si las respuestas se envían como un solo mensaje o se transmiten en bloques:

```json
{
  channels: {
    bluebubbles: {
      blockStreaming: true, // habilitar transmisión por bloques (deshabilitado por defecto)
    },
  },
}
```

## Medios + límites

-   Los archivos adjuntos entrantes se descargan y almacenan en la caché de medios.
-   Límite de medios vía `channels.bluebubbles.mediaMaxMb` para medios entrantes y salientes (por defecto: 8 MB).
-   El texto saliente se divide en fragmentos de `channels.bluebubbles.textChunkLimit` (por defecto: 4000 caracteres).

## Referencia de configuración

Configuración completa: [Configuración](../gateway/configuration.md) Opciones del proveedor:

-   `channels.bluebubbles.enabled`: Habilitar/deshabilitar el canal.
-   `channels.bluebubbles.serverUrl`: URL base de la API REST de BlueBubbles.
-   `channels.bluebubbles.password`: Contraseña de la API.
-   `channels.bluebubbles.webhookPath`: Ruta del endpoint del webhook (por defecto: `/bluebubbles-webhook`).
-   `channels.bluebubbles.dmPolicy`: `pairing | allowlist | open | disabled` (por defecto: `pairing`).
-   `channels.bluebubbles.allowFrom`: Lista de permitidos para MD (identificadores, correos, números E.164, `chat_id:*`, `chat_guid:*`).
-   `channels.bluebubbles.groupPolicy`: `open | allowlist | disabled` (por defecto: `allowlist`).
-   `channels.bluebubbles.groupAllowFrom`: Lista de permitidos de remitentes de grupo.
-   `channels.bluebubbles.groups`: Configuración por grupo (`requireMention`, etc.).
-   `channels.bluebubbles.sendReadReceipts`: Enviar confirmaciones de lectura (por defecto: `true`).
-   `channels.bluebubbles.blockStreaming`: Habilitar transmisión por bloques (por defecto: `false`; requerido para respuestas en transmisión).
-   `channels.bluebubbles.textChunkLimit`: Tamaño del fragmento saliente en caracteres (por defecto: 4000).
-   `channels.bluebubbles.chunkMode`: `length` (por defecto) divide solo cuando excede `textChunkLimit`; `newline` divide en líneas en blanco (límites de párrafo) antes de la división por longitud.
-   `channels.bluebubbles.mediaMaxMb`: Límite de medios entrantes/salientes en MB (por defecto: 8).
-   `channels.bluebubbles.mediaLocalRoots`: Lista de permitidos explícita de directorios locales absolutos permitidos para rutas de medios locales salientes. Los envíos de rutas locales se deniegan por defecto a menos que esto esté configurado. Anulación por cuenta: `channels.bluebubbles.accounts..mediaLocalRoots`.
-   `channels.bluebubbles.historyLimit`: Máximo de mensajes de grupo para contexto (0 deshabilita).
-   `channels.bluebubbles.dmHistoryLimit`: Límite de historial de MD.
-   `channels.bluebubbles.actions`: Habilitar/deshabilitar acciones específicas.
-   `channels.bluebubbles.accounts`: Configuración multi-cuenta.

Opciones globales relacionadas:

-   `agents.list[].groupChat.mentionPatterns` (o `messages.groupChat.mentionPatterns`).
-   `messages.responsePrefix`.

## Direccionamiento / destinos de entrega

Prefiere `chat_guid` para enrutamiento estable:

-   `chat_guid:iMessage;-;+15555550123` (preferido para grupos)
-   `chat_id:123`
-   `chat_identifier:...`
-   Identificadores directos: `+15555550123`, `user@example.com`
    -   Si un identificador directo no tiene un chat de MD existente, OpenClaw creará uno vía `POST /api/v1/chat/new`. Esto requiere que la API Privada de BlueBubbles esté habilitada.

## Seguridad

-   Las solicitudes de webhook se autentican comparando los parámetros de consulta o encabezados `guid`/`password` con `channels.bluebubbles.password`. Las solicitudes desde `localhost` también se aceptan.
-   Mantén la contraseña de la API y el endpoint del webhook en secreto (trátalos como credenciales).
-   La confianza en localhost significa que un proxy inverso en el mismo host puede omitir la contraseña involuntariamente. Si usas proxy para la puerta de enlace, requiere autenticación en el proxy y configura `gateway.trustedProxies`. Ver [Seguridad de la puerta de enlace](../gateway/security.md#reverse-proxy-configuration).
-   Habilita HTTPS + reglas de firewall en el servidor BlueBubbles si lo expones fuera de tu LAN.

## Resolución de problemas

-   Si los eventos de escritura/lectura dejan de funcionar, revisa los registros de webhook de BlueBubbles y verifica que la ruta de la puerta de enlace coincida con `channels.bluebubbles.webhookPath`.
-   Los códigos de emparejamiento expiran después de una hora; usa `openclaw pairing list bluebubbles` y `openclaw pairing approve bluebubbles `.
-   Las reacciones requieren la API privada de BlueBubbles (`POST /api/v1/message/react`); asegúrate de que la versión del servidor la exponga.
-   Editar/eliminar requieren macOS 13+ y una versión compatible del servidor BlueBubbles. En macOS 26 (Tahoe), editar está actualmente roto debido a cambios en la API privada.
-   Las actualizaciones de iconos de grupo pueden ser inestables en macOS 26 (Tahoe): la API puede devolver éxito pero el nuevo icono no se sincroniza.
-   OpenClaw oculta automáticamente las acciones conocidas como rotas según la versión de macOS del servidor BlueBubbles. Si editar aún aparece en macOS 26 (Tahoe), deshabilítalo manualmente con `channels.bluebubbles.actions.edit=false`.
-   Para información de estado/salud: `openclaw status --all` o `openclaw status --deep`.

Para referencia general del flujo de trabajo de canales, ver [Canales](../channels.md) y la guía de [Plugins](../tools/plugin.md).

[Canales de chat](../channels.md)[Discord](./discord.md)