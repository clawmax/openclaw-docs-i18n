

  Plataformas de mensajería

  
# Slack

Estado: listo para producción en MDs + canales mediante integraciones de aplicaciones de Slack. El modo predeterminado es Socket Mode; también se admite el modo HTTP Events API.

## Configuración rápida

## Modelo de tokens

-   `botToken` + `appToken` son necesarios para Socket Mode.
-   El modo HTTP requiere `botToken` + `signingSecret`.
-   Los tokens de configuración anulan el respaldo de variables de entorno.
-   El respaldo de las variables de entorno `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` solo se aplica a la cuenta predeterminada.
-   `userToken` (`xoxp-...`) es solo de configuración (sin respaldo de variable de entorno) y por defecto tiene comportamiento de solo lectura (`userTokenReadOnly: true`).
-   Opcional: añade `chat:write.customize` si quieres que los mensajes salientes usen la identidad del agente activo (`username` e icono personalizados). `icon_emoji` usa la sintaxis `:nombre_emoji:`.

> **💡** Para acciones/lecturas de directorio, el token de usuario puede ser preferible cuando está configurado. Para escrituras, el token de bot sigue siendo preferido; las escrituras con token de usuario solo se permiten cuando `userTokenReadOnly: false` y el token de bot no está disponible.

## Control de acceso y enrutamiento

`channels.slack.dmPolicy` controla el acceso a MDs (heredado: `channels.slack.dm.policy`):

-   `pairing` (predeterminado)
-   `allowlist`
-   `open` (requiere que `channels.slack.allowFrom` incluya `"*"`; heredado: `channels.slack.dm.allowFrom`)
-   `disabled`

Banderas de MDs:

-   `dm.enabled` (verdadero por defecto)
-   `channels.slack.allowFrom` (preferido)
-   `dm.allowFrom` (heredado)
-   `dm.groupEnabled` (MDs grupales falso por defecto)
-   `dm.groupChannels` (lista de permitidos MPIM opcional)

Precedencia de cuentas múltiples:

-   `channels.slack.accounts.default.allowFrom` se aplica solo a la cuenta `default`.
-   Las cuentas nombradas heredan `channels.slack.allowFrom` cuando su propio `allowFrom` no está definido.
-   Las cuentas nombradas no heredan `channels.slack.accounts.default.allowFrom`.

El emparejamiento en MDs usa `openclaw pairing approve slack <código>`.

`channels.slack.groupPolicy` controla el manejo de canales:

-   `open`
-   `allowlist`
-   `disabled`

La lista de permitidos de canales está bajo `channels.slack.channels`.
Nota de tiempo de ejecución: si `channels.slack` está completamente ausente (configuración solo con variables de entorno), el tiempo de ejecución recurre a `groupPolicy="allowlist"` y registra una advertencia (incluso si `channels.defaults.groupPolicy` está configurado).
Resolución de nombre/ID:

-   las entradas de la lista de permitidos de canales y las entradas de la lista de permitidos de MDs se resuelven al inicio cuando el acceso del token lo permite
-   las entradas no resueltas se mantienen como configuradas
-   la coincidencia de autorización entrante es por ID primero por defecto; la coincidencia directa por nombre de usuario/slug requiere `channels.slack.dangerouslyAllowNameMatching: true`

Los mensajes del canal están condicionados a mención por defecto.
Fuentes de mención:

-   mención explícita de la aplicación (`<@botId>`)
-   patrones de expresión regular de mención (`agents.list[].groupChat.mentionPatterns`, respaldo `messages.groupChat.mentionPatterns`)
-   comportamiento implícito de respuesta-al-bot en hilo

Controles por canal (`channels.slack.channels.<id|nombre>`):

-   `requireMention`
-   `users` (lista de permitidos)
-   `allowBots`
-   `skills`
-   `systemPrompt`
-   `tools`, `toolsBySender`
-   formato de clave `toolsBySender`: `id:`, `e164:`, `username:`, `name:`, o comodín `"*"` (las claves heredadas sin prefijo aún se asignan solo a `id:`)

## Comandos y comportamiento de barra

-   El modo automático de comandos nativos está **desactivado** para Slack (`commands.native: "auto"` no habilita comandos nativos de Slack).
-   Habilita los manejadores de comandos nativos de Slack con `channels.slack.commands.native: true` (o globalmente `commands.native: true`).
-   Cuando los comandos nativos están habilitados, registra comandos de barra coincidentes en Slack (nombres `/`), con una excepción:
    -   registra `/agentstatus` para el comando de estado (Slack reserva `/status`)
-   Si los comandos nativos no están habilitados, puedes ejecutar un único comando de barra configurado mediante `channels.slack.slashCommand`.
-   Los menús de argumentos nativos ahora adaptan su estrategia de renderizado:
    -   hasta 5 opciones: bloques de botones
    -   6-100 opciones: menú de selección estático
    -   más de 100 opciones: selección externa con filtrado asíncrono de opciones cuando hay manejadores de opciones de interactividad disponibles
    -   si los valores de opción codificados exceden los límites de Slack, el flujo recurre a botones
-   Para cargas útiles largas de opciones, los menús de argumentos de comandos de barra usan un diálogo de confirmación antes de enviar un valor seleccionado.

Configuración predeterminada de comandos de barra:

-   `enabled: false`
-   `name: "openclaw"`
-   `sessionPrefix: "slack:slash"`
-   `ephemeral: true`

Las sesiones de barra usan claves aisladas:

-   `agent::slack:slash:`

y aún enrutan la ejecución del comando contra la sesión de conversación objetivo (`CommandTargetSessionKey`).

## Hilos, sesiones y etiquetas de respuesta

-   Los MDs se enrutan como `direct`; los canales como `channel`; los MPIMs como `group`.
-   Con el valor predeterminado `session.dmScope=main`, los MDs de Slack se colapsan a la sesión principal del agente.
-   Sesiones de canal: `agent::slack:channel:`.
-   Las respuestas en hilo pueden crear sufijos de sesión de hilo (`:thread:`) cuando corresponda.
-   `channels.slack.thread.historyScope` por defecto es `thread`; `thread.inheritParent` por defecto es `false`.
-   `channels.slack.thread.initialHistoryLimit` controla cuántos mensajes existentes del hilo se obtienen cuando comienza una nueva sesión de hilo (predeterminado `20`; establece `0` para deshabilitar).

Controles de hilos de respuesta:

-   `channels.slack.replyToMode`: `off|first|all` (predeterminado `off`)
-   `channels.slack.replyToModeByChatType`: por `direct|group|channel`
-   respaldo heredado para chats directos: `channels.slack.dm.replyToMode`

Se admiten etiquetas de respuesta manual:

-   `[[reply_to_current]]`
-   `[[reply_to:]]`

Nota: `replyToMode="off"` deshabilita **todos** los hilos de respuesta en Slack, incluidas las etiquetas explícitas `[[reply_to_*]]`. Esto difiere de Telegram, donde las etiquetas explícitas aún se respetan en modo `"off"`. La diferencia refleja los modelos de hilos de las plataformas: los hilos de Slack ocultan mensajes del canal, mientras que las respuestas de Telegram permanecen visibles en el flujo principal del chat.

## Medios, fragmentación y entrega

Los archivos adjuntos de Slack se descargan desde URLs privadas alojadas por Slack (flujo de solicitud autenticada con token) y se escriben en el almacén de medios cuando la obtención tiene éxito y los límites de tamaño lo permiten.
El límite de tamaño entrante en tiempo de ejecución es por defecto `20MB` a menos que se anule con `channels.slack.mediaMaxMb`.

-   los fragmentos de texto usan `channels.slack.textChunkLimit` (predeterminado 4000)
-   `channels.slack.chunkMode="newline"` habilita la división por párrafo primero
-   los envíos de archivos usan las APIs de carga de Slack y pueden incluir respuestas en hilo (`thread_ts`)
-   el límite de medios salientes sigue `channels.slack.mediaMaxMb` cuando está configurado; de lo contrario, los envíos de canal usan los valores predeterminados por tipo MIME de la canalización de medios

Destinos explícitos preferidos:

-   `user:` para MDs
-   `channel:` para canales

Los MDs de Slack se abren mediante las APIs de conversación de Slack al enviar a destinos de usuario.

## Acciones y compuertas

Las acciones de Slack se controlan mediante `channels.slack.actions.*`. Grupos de acciones disponibles en las herramientas actuales de Slack:

| Grupo | Predeterminado |
| --- | --- |
| messages | habilitado |
| reactions | habilitado |
| pins | habilitado |
| memberInfo | habilitado |
| emojiList | habilitado |

## Eventos y comportamiento operativo

-   Las ediciones/eliminaciones de mensajes y transmisiones de hilos se mapean a eventos del sistema.
-   Los eventos de añadir/eliminar reacciones se mapean a eventos del sistema.
-   Los eventos de unirse/salir de miembros, crear/renombrar canales y añadir/eliminar fijaciones se mapean a eventos del sistema.
-   Las actualizaciones de estado de hilos del asistente (para indicadores de "está escribiendo..." en hilos) usan `assistant.threads.setStatus` y requieren el alcance de bot `assistant:write`.
-   `channel_id_changed` puede migrar claves de configuración de canal cuando `configWrites` está habilitado.
-   Los metadatos de tema/propósito del canal se tratan como contexto no confiable y pueden inyectarse en el contexto de enrutamiento.
-   Las acciones de bloque y las interacciones modales emiten eventos estructurados del sistema `Slack interaction: ...` con campos de carga útil ricos:
    -   acciones de bloque: valores seleccionados, etiquetas, valores del selector y metadatos `workflow_*`
    -   eventos modales `view_submission` y `view_closed` con metadatos de canal enrutados y entradas de formulario

## Reacciones de confirmación

`ackReaction` envía un emoji de confirmación mientras OpenClaw procesa un mensaje entrante. Orden de resolución:

-   `channels.slack.accounts..ackReaction`
-   `channels.slack.ackReaction`
-   `messages.ackReaction`
-   respaldo de emoji de identidad del agente (`agents.list[].identity.emoji`, si no ”👀”)

Notas:

-   Slack espera códigos cortos (por ejemplo `"eyes"`).
-   Usa `""` para deshabilitar la reacción para la cuenta de Slack o globalmente.

## Respaldo de reacción de escritura

`typingReaction` añade una reacción temporal al mensaje entrante de Slack mientras OpenClaw procesa una respuesta, luego la elimina cuando la ejecución termina. Este es un respaldo útil cuando la escritura nativa del asistente de Slack no está disponible, especialmente en MDs. Orden de resolución:

-   `channels.slack.accounts..typingReaction`
-   `channels.slack.typingReaction`

Notas:

-   Slack espera códigos cortos (por ejemplo `"hourglass_flowing_sand"`).
-   La reacción es de mejor esfuerzo y la limpieza se intenta automáticamente después de que la respuesta o la ruta de fallo se complete.

## Manifiesto y lista de verificación de alcances

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "Conector de Slack para OpenClaw"
  },
  "features": {
    "bot_user": {
      "display_name": "OpenClaw",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "Envía un mensaje a OpenClaw",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "im:history",
        "im:read",
        "im:write",
        "mpim:history",
        "mpim:read",
        "mpim:write",
        "users:read",
        "app_mentions:read",
        "assistant:write",
        "reactions:read",
        "reactions:write",
        "pins:read",
        "pins:write",
        "emoji:read",
        "commands",
        "files:read",
        "files:write"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "reaction_added",
        "reaction_removed",
        "member_joined_channel",
        "member_left_channel",
        "channel_rename",
        "pin_added",
        "pin_removed"
      ]
    }
  }
}
```

Si configuras `channels.slack.userToken`, los alcances de lectura típicos son:

-   `channels:history`, `groups:history`, `im:history`, `mpim:history`
-   `channels:read`, `groups:read`, `im:read`, `mpim:read`
-   `users:read`
-   `reactions:read`
-   `pins:read`
-   `emoji:read`
-   `search:read` (si dependes de lecturas de búsqueda de Slack)

## Solución de problemas

Verifica, en orden:

-   `groupPolicy`
-   lista de permitidos de canales (`channels.slack.channels`)
-   `requireMention`
-   lista de permitidos `users` por canal

Comandos útiles:

```bash
openclaw channels status --probe
openclaw logs --follow
openclaw doctor
```

Verifica:

-   `channels.slack.dm.enabled`
-   `channels.slack.dmPolicy` (o heredado `channels.slack.dm.policy`)
-   aprobaciones de emparejamiento / entradas de lista de permitidos

```bash
openclaw pairing list slack
```

Valida los tokens de bot + app y la habilitación de Socket Mode en la configuración de la aplicación de Slack.

Valida:

-   secreto de firma
-   ruta del webhook
-   URLs de solicitud de Slack (Events + Interactivity + Slash Commands)
-   `webhookPath` único por cuenta HTTP

Verifica si pretendías:

-   modo de comando nativo (`channels.slack.commands.native: true`) con comandos de barra coincidentes registrados en Slack
-   o modo de comando de barra único (`channels.slack.slashCommand.enabled: true`)

También verifica `commands.useAccessGroups` y las listas de permitidos de canal/usuario.

## Transmisión de texto

OpenClaw admite transmisión de texto nativa de Slack mediante la API de Agentes y Aplicaciones de IA. `channels.slack.streaming` controla el comportamiento de vista previa en vivo:

-   `off`: deshabilita la transmisión de vista previa en vivo.
-   `partial` (predeterminado): reemplaza el texto de vista previa con la salida parcial más reciente.
-   `block`: añade actualizaciones de vista previa fragmentadas.
-   `progress`: muestra texto de estado de progreso mientras se genera, luego envía el texto final.

`channels.slack.nativeStreaming` controla la API de transmisión nativa de Slack (`chat.startStream` / `chat.appendStream` / `chat.stopStream`) cuando `streaming` es `partial` (predeterminado: `true`). Deshabilita la transmisión nativa de Slack (mantén el comportamiento de vista previa de borrador):

```yaml
channels:
  slack:
    streaming: partial
    nativeStreaming: false
```

Claves heredadas:

-   `channels.slack.streamMode` (`replace | status_final | append`) se migra automáticamente a `channels.slack.streaming`.
-   el booleano `channels.slack.streaming` se migra automáticamente a `channels.slack.nativeStreaming`.

### Requisitos

1.  Habilita **Agentes y Aplicaciones de IA** en la configuración de tu aplicación de Slack.
2.  Asegúrate de que la aplicación tenga el alcance `assistant:write`.
3.  Debe haber un hilo de respuesta disponible para ese mensaje. La selección de hilo aún sigue `replyToMode`.

### Comportamiento

-   El primer fragmento de texto inicia una transmisión (`chat.startStream`).
-   Los fragmentos de texto posteriores se añaden a la misma transmisión (`chat.appendStream`).
-   El final de la respuesta finaliza la transmisión (`chat.stopStream`).
-   Los medios y cargas útiles que no son texto recurren a la entrega normal.
-   Si la transmisión falla a mitad de la respuesta, OpenClaw recurre a la entrega normal para las cargas útiles restantes.

## Puntos de referencia de configuración

Referencia principal:

-   [Referencia de configuración - Slack](../gateway/configuration-reference.md#slack) Campos de Slack de alta señal:
    -   modo/autenticación: `mode`, `botToken`, `appToken`, `signingSecret`, `webhookPath`, `accounts.*`
    -   acceso a MDs: `dm.enabled`, `dmPolicy`, `allowFrom` (heredado: `dm.policy`, `dm.allowFrom`), `dm.groupEnabled`, `dm.groupChannels`
    -   interruptor de compatibilidad: `dangerouslyAllowNameMatching` (rompe-cristales; mantén desactivado a menos que sea necesario)
    -   acceso a canales: `groupPolicy`, `channels.*`, `channels.*.users`, `channels.*.requireMention`
    -   hilos/historial: `replyToMode`, `replyToModeByChatType`, `thread.*`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
    -   entrega: `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `streaming`, `nativeStreaming`
    -   operaciones/características: `configWrites`, `commands.native`, `slashCommand.*`, `actions.*`, `userToken`, `userTokenReadOnly`

## Relacionado

-   [Emparejamiento](./pairing.md)
-   [Enrutamiento de canales](./channel-routing.md)
-   [Solución de problemas](./troubleshooting.md)
-   [Configuración](../gateway/configuration.md)
-   [Comandos de barra](../tools/slash-commands.md)

[Synology Chat](./synology-chat.md)[Telegram](./telegram.md)