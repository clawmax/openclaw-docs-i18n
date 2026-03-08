

  Plataformas de mensajería

  
# iMessage

> **⚠️** Para nuevos despliegues de iMessage, usa [BlueBubbles](./bluebubbles.md). La integración `imsg` es heredada y podría eliminarse en una versión futura.

 Estado: integración CLI externa heredada. La puerta de enlace ejecuta `imsg rpc` y se comunica mediante JSON-RPC en stdio (sin demonio/puerto separado). 

## Configuración rápida

## Requisitos y permisos (macOS)

-   Messages debe estar iniciado sesión en el Mac que ejecuta `imsg`.
-   Se requiere Acceso Total al Disco para el contexto del proceso que ejecuta OpenClaw/`imsg` (acceso a la base de datos de Messages).
-   Se requiere permiso de Automatización para enviar mensajes a través de Messages.app.

> **💡** Los permisos se otorgan por contexto de proceso. Si la puerta de enlace se ejecuta sin interfaz gráfica (LaunchAgent/SSH), ejecuta un comando interactivo una sola vez en ese mismo contexto para activar las solicitudes:
> 
> Copiar
> 
> ```
> imsg chats --limit 1
> # o
> imsg send <handle> "test"
> ```

## Control de acceso y enrutamiento

`channels.imessage.dmPolicy` controla los mensajes directos:

-   `pairing` (por defecto)
-   `allowlist`
-   `open` (requiere que `allowFrom` incluya `"*"`)
-   `disabled`

Campo de lista de permitidos: `channels.imessage.allowFrom`.Las entradas de la lista de permitidos pueden ser identificadores o destinos de chat (`chat_id:*`, `chat_guid:*`, `chat_identifier:*`).

`channels.imessage.groupPolicy` controla el manejo de grupos:

-   `allowlist` (por defecto cuando está configurado)
-   `open`
-   `disabled`

Lista de permitidos de remitentes de grupo: `channels.imessage.groupAllowFrom`.Respaldo en tiempo de ejecución: si `groupAllowFrom` no está definido, las comprobaciones de remitente de grupo de iMessage recurren a `allowFrom` cuando está disponible. Nota de tiempo de ejecución: si `channels.imessage` falta completamente, el tiempo de ejecución recurre a `groupPolicy="allowlist"` y registra una advertencia (incluso si `channels.defaults.groupPolicy` está configurado).Control de menciones para grupos:

-   iMessage no tiene metadatos nativos de mención
-   la detección de menciones usa patrones de expresiones regulares (`agents.list[].groupChat.mentionPatterns`, respaldo `messages.groupChat.mentionPatterns`)
-   sin patrones configurados, no se puede aplicar el control de menciones

Los comandos de control de remitentes autorizados pueden omitir el control de menciones en grupos.

-   Los MD usan enrutamiento directo; los grupos usan enrutamiento de grupo.
-   Con el valor por defecto `session.dmScope=main`, los MD de iMessage se agrupan en la sesión principal del agente.
-   Las sesiones de grupo están aisladas (`agent::imessage:group:<chat_id>`).
-   Las respuestas se enrutan de vuelta a iMessage usando los metadatos del canal/destino de origen.

Comportamiento de hilos tipo grupo:Algunos hilos de iMessage con múltiples participantes pueden llegar con `is_group=false`. Si ese `chat_id` está configurado explícitamente bajo `channels.imessage.groups`, OpenClaw lo trata como tráfico de grupo (control de grupo + aislamiento de sesión de grupo).

## Patrones de despliegue

Usa un ID de Apple y un usuario de macOS dedicados para que el tráfico del bot esté aislado de tu perfil personal de Messages.Flujo típico:

1.  Crear/iniciar sesión con un usuario de macOS dedicado.
2.  Iniciar sesión en Messages con el ID de Apple del bot en ese usuario.
3.  Instalar `imsg` en ese usuario.
4.  Crear un envoltorio SSH para que OpenClaw pueda ejecutar `imsg` en el contexto de ese usuario.
5.  Apuntar `channels.imessage.accounts..cliPath` y `.dbPath` al perfil de ese usuario.

La primera ejecución puede requerir aprobaciones GUI (Automatización + Acceso Total al Disco) en la sesión de ese usuario bot.

Topología común:

-   la puerta de enlace se ejecuta en Linux/VM
-   iMessage + `imsg` se ejecuta en un Mac en tu tailnet
-   el envoltorio `cliPath` usa SSH para ejecutar `imsg`
-   `remoteHost` habilita la obtención de archivos adjuntos por SCP

Ejemplo:

```json
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db",
    },
  },
}
```

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

Usa claves SSH para que tanto SSH como SCP sean no interactivos. Asegúrate de que la clave del host sea confiable primero (por ejemplo `ssh bot@mac-mini.tailnet-1234.ts.net`) para que `known_hosts` esté poblado.

iMessage admite configuración por cuenta bajo `channels.imessage.accounts`.Cada cuenta puede sobrescribir campos como `cliPath`, `dbPath`, `allowFrom`, `groupPolicy`, `mediaMaxMb`, configuraciones de historial y listas de permitidos de raíz de archivos adjuntos.

## Medios, fragmentación y destinos de entrega

-   la ingesta de archivos adjuntos entrantes es opcional: `channels.imessage.includeAttachments`
-   las rutas de archivos adjuntos remotos se pueden obtener mediante SCP cuando `remoteHost` está configurado
-   las rutas de archivos adjuntos deben coincidir con raíces permitidas:
    -   `channels.imessage.attachmentRoots` (local)
    -   `channels.imessage.remoteAttachmentRoots` (modo SCP remoto)
    -   patrón de raíz por defecto: `/Users/*/Library/Messages/Attachments`
-   SCP usa comprobación estricta de clave de host (`StrictHostKeyChecking=yes`)
-   el tamaño de medios salientes usa `channels.imessage.mediaMaxMb` (por defecto 16 MB)

-   límite de fragmento de texto: `channels.imessage.textChunkLimit` (por defecto 4000)
-   modo de fragmentación: `channels.imessage.chunkMode`
    -   `length` (por defecto)
    -   `newline` (división priorizando párrafos)

Destinos explícitos preferidos:

-   `chat_id:123` (recomendado para enrutamiento estable)
-   `chat_guid:...`
-   `chat_identifier:...`

También se admiten destinos por identificador:

-   `imessage:+1555...`
-   `sms:+1555...`
-   `user@example.com`

```bash
imsg chats --limit 20
```

## Escrituras de configuración

iMessage permite escrituras de configuración iniciadas por el canal por defecto (para `/config set|unset` cuando `commands.config: true`). Deshabilitar:

```json
{
  channels: {
    imessage: {
      configWrites: false,
    },
  },
}
```

## Solución de problemas

Valida el binario y el soporte RPC:

```bash
imsg rpc --help
openclaw channels status --probe
```

Si la sonda reporta RPC no compatible, actualiza `imsg`.

Verifica:

-   `channels.imessage.dmPolicy`
-   `channels.imessage.allowFrom`
-   aprobaciones de emparejamiento (`openclaw pairing list imessage`)

Verifica:

-   `channels.imessage.groupPolicy`
-   `channels.imessage.groupAllowFrom`
-   comportamiento de la lista de permitidos `channels.imessage.groups`
-   configuración de patrones de mención (`agents.list[].groupChat.mentionPatterns`)

Verifica:

-   `channels.imessage.remoteHost`
-   `channels.imessage.remoteAttachmentRoots`
-   autenticación por clave SSH/SCP desde el host de la puerta de enlace
-   la clave del host existe en `~/.ssh/known_hosts` en el host de la puerta de enlace
-   legibilidad de la ruta remota en el Mac que ejecuta Messages

Vuelve a ejecutar en una terminal GUI interactiva en el mismo contexto de usuario/sesión y aprueba las solicitudes:

```bash
imsg chats --limit 1
imsg send <handle> "test"
```

Confirma que Acceso Total al Disco + Automatización están otorgados para el contexto del proceso que ejecuta OpenClaw/`imsg`.

## Puntos de referencia de configuración

-   [Referencia de configuración - iMessage](../gateway/configuration-reference.md#imessage)
-   [Configuración de la puerta de enlace](../gateway/configuration.md)
-   [Emparejamiento](./pairing.md)
-   [BlueBubbles](./bluebubbles.md)

[Google Chat](./googlechat.md)[IRC](./irc.md)