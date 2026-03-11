

  Plataformas de mensajería

  
# Discord

Estado: listo para mensajes directos y canales de gremio a través de la puerta de enlace oficial de Discord.

## Configuración rápida

Necesitarás crear una nueva aplicación con un bot, añadir el bot a tu servidor y vincularlo a OpenClaw. Recomendamos añadir tu bot a tu propio servidor privado. Si aún no tienes uno, [crea uno primero](https://support.discord.com/hc/en-us/articles/204849977-How-do-I-create-a-server) (elige **Crear el mío propio > Para mí y mis amigos**).

### Paso 1: Crear una aplicación y un bot de Discord

Ve al [Portal de Desarrolladores de Discord](https://discord.com/developers/applications) y haz clic en **Nueva Aplicación**. Nómbrala algo como "OpenClaw". Haz clic en **Bot** en la barra lateral. Establece el **Nombre de usuario** como quieras llamar a tu agente OpenClaw.

### Paso 2: Habilitar intenciones privilegiadas

Todavía en la página **Bot**, desplázate hacia abajo hasta **Intenciones Privilegiadas de la Puerta de Enlace** y habilita:

-   **Intención de Contenido de Mensajes** (requerida)
-   **Intención de Miembros del Servidor** (recomendada; requerida para listas de permisos de roles y coincidencia de nombre a ID)
-   **Intención de Presencia** (opcional; solo necesaria para actualizaciones de presencia)

### Paso 3: Copiar tu token del bot

Desplázate hacia arriba en la página **Bot** y haz clic en **Restablecer Token**.

> **ℹ️** A pesar del nombre, esto genera tu primer token — no se está "restableciendo" nada.

Copia el token y guárdalo en algún lugar. Este es tu **Token del Bot** y lo necesitarás en breve.

### Paso 4: Generar una URL de invitación y añadir el bot a tu servidor

Haz clic en **OAuth2** en la barra lateral. Generarás una URL de invitación con los permisos correctos para añadir el bot a tu servidor. Desplázate hacia abajo hasta **Generador de URL OAuth2** y habilita:

-   `bot`
-   `applications.commands`

Aparecerá una sección **Permisos del Bot** debajo. Habilita:

-   Ver Canales
-   Enviar Mensajes
-   Leer Historial de Mensajes
-   Insertar Enlaces
-   Adjuntar Archivos
-   Añadir Reacciones (opcional)

Copia la URL generada en la parte inferior, pégala en tu navegador, selecciona tu servidor y haz clic en **Continuar** para conectar. Ahora deberías ver tu bot en el servidor de Discord.

### Paso 5: Habilitar Modo Desarrollador y recopilar tus IDs

De vuelta en la aplicación de Discord, necesitas habilitar el Modo Desarrollador para poder copiar IDs internos.

1.  Haz clic en **Configuración de Usuario** (icono de engranaje junto a tu avatar) → **Avanzado** → activa **Modo Desarrollador**
2.  Haz clic derecho en el **icono de tu servidor** en la barra lateral → **Copiar ID del Servidor**
3.  Haz clic derecho en **tu propio avatar** → **Copiar ID de Usuario**

Guarda tu **ID del Servidor** y **ID de Usuario** junto con tu Token del Bot — enviarás los tres a OpenClaw en el siguiente paso.

### Paso 6: Permitir mensajes directos de miembros del servidor

Para que la vinculación funcione, Discord necesita permitir que tu bot te envíe mensajes directos. Haz clic derecho en el **icono de tu servidor** → **Configuración de Privacidad** → activa **Mensajes Directos**. Esto permite que los miembros del servidor (incluidos los bots) te envíen mensajes directos. Mantén esto habilitado si quieres usar mensajes directos de Discord con OpenClaw. Si solo planeas usar canales de gremio, puedes deshabilitar los mensajes directos después de la vinculación.

### Paso 7: Paso 0: Establecer tu token del bot de forma segura (no lo envíes en el chat)

Tu token del bot de Discord es un secreto (como una contraseña). Establécelo en la máquina que ejecuta OpenClaw antes de enviar mensajes a tu agente.

```bash
openclaw config set channels.discord.token '"YOUR_BOT_TOKEN"' --json
openclaw config set channels.discord.enabled true --json
openclaw gateway
```

Si OpenClaw ya se está ejecutando como un servicio en segundo plano, usa `openclaw gateway restart` en su lugar.

### Paso 8: Configurar OpenClaw y vincular

Chatea con tu agente OpenClaw en cualquier canal existente (por ejemplo, Telegram) y dile. Si Discord es tu primer canal, usa la CLI / pestaña de configuración en su lugar.

> “Ya configuré mi token del bot de Discord en la configuración. Por favor, termina la configuración de Discord con el ID de Usuario `<user_id>` y el ID del Servidor `<server_id>`.”

```json
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN",
    },
  },
}
```

### Paso 9: Aprobar la primera vinculación por mensaje directo

Espera hasta que la puerta de enlace esté en ejecución, luego envía un mensaje directo a tu bot en Discord. Responderá con un código de vinculación.

Envía el código de vinculación a tu agente en tu canal existente:

> “Aprueba este código de vinculación de Discord: ``”

```bash
openclaw pairing list discord
openclaw pairing approve discord <CODE>
```

Los códigos de vinculación expiran después de 1 hora. Ahora deberías poder chatear con tu agente en Discord a través de mensajes directos.

 

> **ℹ️** La resolución de tokens es consciente de la cuenta. Los valores de token en la configuración tienen prioridad sobre el valor de respaldo de la variable de entorno. `DISCORD_BOT_TOKEN` solo se usa para la cuenta predeterminada.

## Recomendado: Configurar un espacio de trabajo de gremio

Una vez que los mensajes directos funcionen, puedes configurar tu servidor de Discord como un espacio de trabajo completo donde cada canal obtiene su propia sesión de agente con su propio contexto. Esto es recomendado para servidores privados donde solo estás tú y tu bot.

### Paso 1: Añadir tu servidor a la lista de permisos del gremio

Esto permite que tu agente responda en cualquier canal de tu servidor, no solo en mensajes directos.

> “Añade mi ID del Servidor de Discord `<server_id>` a la lista de permisos del gremio”

```json
{
  channels: {
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        YOUR_SERVER_ID: {
          requireMention: true,
          users: ["YOUR_USER_ID"],
        },
      },
    },
  },
}
```

### Paso 2: Permitir respuestas sin @mención

Por defecto, tu agente solo responde en canales de gremio cuando es @mencionado. Para un servidor privado, probablemente quieras que responda a cada mensaje.

> “Permite que mi agente responda en este servidor sin tener que ser @mencionado”

```json
{
  channels: {
    discord: {
      guilds: {
        YOUR_SERVER_ID: {
          requireMention: false,
        },
      },
    },
  },
}
```

### Paso 3: Planificar la memoria en canales de gremio

Por defecto, la memoria a largo plazo (MEMORY.md) solo se carga en sesiones de mensajes directos. Los canales de gremio no cargan automáticamente MEMORY.md.

> “Cuando haga preguntas en canales de Discord, usa memory\_search o memory\_get si necesitas contexto a largo plazo de MEMORY.md.”

Si necesitas contexto compartido en cada canal, coloca las instrucciones estables en `AGENTS.md` o `USER.md` (se inyectan para cada sesión). Mantén notas a largo plazo en `MEMORY.md` y accede a ellas bajo demanda con las herramientas de memoria.

 Ahora crea algunos canales en tu servidor de Discord y comienza a chatear. Tu agente puede ver el nombre del canal, y cada canal obtiene su propia sesión aislada — para que puedas configurar `#coding`, `#home`, `#research`, o lo que se ajuste a tu flujo de trabajo.

## Modelo de tiempo de ejecución

-   La puerta de enlace posee la conexión de Discord.
-   El enrutamiento de respuestas es determinista: las respuestas entrantes de Discord vuelven a Discord.
-   Por defecto (`session.dmScope=main`), los chats directos comparten la sesión principal del agente (`agent:main:main`).
-   Los canales de gremio son claves de sesión aisladas (`agent::discord:channel:`).
-   Los mensajes directos grupales se ignoran por defecto (`channels.discord.dm.groupEnabled=false`).
-   Los comandos nativos de barra se ejecutan en sesiones de comando aisladas (`agent::discord:slash:`), mientras aún llevan `CommandTargetSessionKey` a la sesión de conversación enrutada.

## Canales de foro

Los canales de foro y medios de Discord solo aceptan publicaciones en hilos. OpenClaw admite dos formas de crearlos:

-   Envía un mensaje al padre del foro (`channel:`) para crear automáticamente un hilo. El título del hilo usa la primera línea no vacía de tu mensaje.
-   Usa `openclaw message thread create` para crear un hilo directamente. No pases `--message-id` para canales de foro.

Ejemplo: enviar al padre del foro para crear un hilo

```bash
openclaw message send --channel discord --target channel:<forumId> \
  --message "Título del tema\nCuerpo de la publicación"
```

Ejemplo: crear un hilo de foro explícitamente

```bash
openclaw message thread create --channel discord --target channel:<forumId> \
  --thread-name "Título del tema" --message "Cuerpo de la publicación"
```

Los padres de foro no aceptan componentes de Discord. Si necesitas componentes, envía al hilo mismo (`channel:`).

## Componentes interactivos

OpenClaw admite contenedores de componentes v2 de Discord para mensajes del agente. Usa la herramienta de mensaje con un payload `components`. Los resultados de la interacción se enrutan de vuelta al agente como mensajes entrantes normales y siguen la configuración existente de Discord `replyToMode`. Bloques admitidos:

-   `text`, `section`, `separator`, `actions`, `media-gallery`, `file`
-   Las filas de acciones permiten hasta 5 botones o un solo menú desplegable
-   Tipos de selección: `string`, `user`, `role`, `mentionable`, `channel`

Por defecto, los componentes son de un solo uso. Establece `components.reusable=true` para permitir que botones, selecciones y formularios se usen múltiples veces hasta que expiren. Para restringir quién puede hacer clic en un botón, establece `allowedUsers` en ese botón (IDs de usuario de Discord, etiquetas o `*`). Cuando está configurado, los usuarios no coincidentes reciben una denegación efímera. Los comandos de barra `/model` y `/models` abren un selector de modelo interactivo con menús desplegables de proveedor y modelo más un paso de Enviar. La respuesta del selector es efímera y solo el usuario que lo invoca puede usarlo. Archivos adjuntos:

-   Los bloques `file` deben apuntar a una referencia de adjunto (`attachment://`)
-   Proporciona el adjunto a través de `media`/`path`/`filePath` (archivo único); usa `media-gallery` para múltiples archivos
-   Usa `filename` para anular el nombre de carga cuando debe coincidir con la referencia del adjunto

Formularios modales:

-   Añade `components.modal` con hasta 5 campos
-   Tipos de campo: `text`, `checkbox`, `radio`, `select`, `role-select`, `user-select`
-   OpenClaw añade automáticamente un botón de activación

Ejemplo:

```json
{
  channel: "discord",
  action: "send",
  to: "channel:123456789012345678",
  message: "Texto de respaldo opcional",
  components: {
    reusable: true,
    text: "Elige un camino",
    blocks: [
      {
        type: "actions",
        buttons: [
          {
            label: "Aprobar",
            style: "success",
            allowedUsers: ["123456789012345678"],
          },
          { label: "Rechazar", style: "danger" },
        ],
      },
      {
        type: "actions",
        select: {
          type: "string",
          placeholder: "Elige una opción",
          options: [
            { label: "Opción A", value: "a" },
            { label: "Opción B", value: "b" },
          ],
        },
      },
    ],
    modal: {
      title: "Detalles",
      triggerLabel: "Abrir formulario",
      fields: [
        { type: "text", label: "Solicitante" },
        {
          type: "select",
          label: "Prioridad",
          options: [
            { label: "Baja", value: "low" },
            { label: "Alta", value: "high" },
          ],
        },
      ],
    },
  },
}
```

## Control de acceso y enrutamiento

`channels.discord.dmPolicy` controla el acceso a mensajes directos (heredado: `channels.discord.dm.policy`):

-   `pairing` (predeterminado)
-   `allowlist`
-   `open` (requiere que `channels.discord.allowFrom` incluya `"*"`; heredado: `channels.discord.dm.allowFrom`)
-   `disabled`

Si la política de MD no está abierta, los usuarios desconocidos son bloqueados (o se les solicita vinculación en modo `pairing`). Precedencia multi-cuenta:

-   `channels.discord.accounts.default.allowFrom` se aplica solo a la cuenta `default`.
-   Las cuentas nombradas heredan `channels.discord.allowFrom` cuando su propio `allowFrom` no está establecido.
-   Las cuentas nombradas no heredan `channels.discord.accounts.default.allowFrom`.

Formato de destino de MD para entrega:

-   `user:`
-   mención `<@id>`

Los IDs numéricos simples son ambiguos y se rechazan a menos que se proporcione un tipo de destino de usuario/canal explícito.

```json
{
  channels: {
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        "123456789012345678": {
          requireMention: true,
          ignoreOtherMentions: true,
          users: ["987654321098765432"],
          roles: ["123456789012345678"],
          channels: {
            general: { allow: true },
            help: { allow: true, requireMention: true },
          },
        },
      },
    },
  },
}
```

Los mensajes de gremio están protegidos por mención por defecto. La detección de menciones incluye:

-   mención explícita al bot
-   patrones de mención configurados (`agents.list[].groupChat.mentionPatterns`, respaldo `messages.groupChat.mentionPatterns`)
-   comportamiento implícito de respuesta-al-bot en casos admitidos

`requireMention` se configura por gremio/canal (`channels.discord.guilds...`). `ignoreOtherMentions` opcionalmente descarta mensajes que mencionan a otro usuario/rol pero no al bot (excluyendo @everyone/@here). MD grupales:

-   predeterminado: ignorados (`dm.groupEnabled=false`)
-   lista de permisos opcional a través de `dm.groupChannels` (IDs de canal o slugs)

### Enrutamiento de agente basado en roles

Usa `bindings[].match.roles` para enrutar miembros del gremio de Discord a diferentes agentes por ID de rol. Los enlaces basados en roles aceptan solo IDs de rol y se evalúan después de los enlaces de par o par-padre y antes de los enlaces solo de gremio. Si un enlace también establece otros campos de coincidencia (por ejemplo `peer` + `guildId` + `roles`), todos los campos configurados deben coincidir.

```json
{
  bindings: [
    {
      agentId: "opus",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
        roles: ["111111111111111111"],
      },
    },
    {
      agentId: "sonnet",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
      },
    },
  ],
}
```

## Configuración del Portal de Desarrollador

1.  Portal de Desarrolladores de Discord -> **Aplicaciones** -> **Nueva Aplicación**
2.  **Bot** -> **Añadir Bot**
3.  Copiar token del bot

En **Bot -> Intenciones Privilegiadas de la Puerta de Enlace**, habilita:

-   Intención de Contenido de Mensajes
-   Intención de Miembros del Servidor (recomendada)

La intención de presencia es opcional y solo se requiere si quieres recibir actualizaciones de presencia. Establecer la presencia del bot (`setPresence`) no requiere habilitar actualizaciones de presencia para miembros.

Generador de URL OAuth:

-   ámbitos: `bot`, `applications.commands`

Permisos base típicos:

-   Ver Canales
-   Enviar Mensajes
-   Leer Historial de Mensajes
-   Insertar Enlaces
-   Adjuntar Archivos
-   Añadir Reacciones (opcional)

Evita `Administrator` a menos que sea explícitamente necesario.

Habilita el Modo Desarrollador de Discord, luego copia:

-   ID del servidor
-   ID del canal
-   ID de usuario

Prefiere IDs numéricos en la configuración de OpenClaw para auditorías y sondeos confiables.

## Comandos nativos y autenticación de comandos

-   `commands.native` por defecto es `"auto"` y está habilitado para Discord.
-   Anulación por canal: `channels.discord.commands.native`.
-   `commands.native=false` borra explícitamente los comandos nativos de Discord registrados previamente.
-   La autenticación de comandos nativos usa las mismas listas de permisos/políticas de Discord que el manejo normal de mensajes.
-   Los comandos aún pueden ser visibles en la interfaz de usuario de Discord para usuarios no autorizados; la ejecución aún aplica la autenticación de OpenClaw y devuelve "no autorizado".

Consulta [Comandos de barra](../tools/slash-commands.md) para el catálogo y comportamiento de comandos. Configuración predeterminada de comandos de barra:

-   `ephemeral: true`

## Detalles de características

Discord admite etiquetas de respuesta en la salida del agente:

-   `[[reply_to_current]]`
-   `[[reply_to:]]`

Controlado por `channels.discord.replyToMode`:

-   `off` (predeterminado)
-   `first`
-   `all`

Nota: `off` deshabilita el hilo de respuesta implícito. Las etiquetas explícitas `[[reply_to_*]]` aún se respetan. Los IDs de mensaje se muestran en el contexto/historial para que los agentes puedan apuntar a mensajes específicos.

OpenClaw puede transmitir borradores de respuestas enviando un mensaje temporal y editándolo a medida que llega el texto.

-   `channels.discord.streaming` controla la transmisión de vista previa (`off` | `partial` | `block` | `progress`, predeterminado: `off`).
-   `progress` se acepta para consistencia entre canales y se asigna a `partial` en Discord.
-   `channels.discord.streamMode` es un alias heredado y se migra automáticamente.
-   `partial` edita un solo mensaje de vista previa a medida que llegan los tokens.
-   `block` emite fragmentos del tamaño de un borrador (usa `draftChunk` para ajustar el tamaño y los puntos de ruptura).

Ejemplo:

```json
{
  channels: {
    discord: {
      streaming: "partial",
    },
  },
}
```

Fragmentación predeterminada del modo `block` (limitada a `channels.discord.textChunkLimit`):

```json
{
  channels: {
    discord: {
      streaming: "block",
      draftChunk: {
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph",
      },
    },
  },
}
```

La transmisión de vista previa es solo de texto; las respuestas con medios vuelven a la entrega normal. Nota: la transmisión de vista previa es independiente de la transmisión por bloques. Cuando la transmisión por bloques está explícitamente habilitada para Discord, OpenClaw omite la transmisión de vista previa para evitar una doble transmisión.

Contexto de historial del gremio:

-   `channels.discord.historyLimit` predeterminado `20`
-   respaldo: `messages.groupChat.historyLimit`
-   `0` deshabilita

Controles de historial de MD:

-   `channels.discord.dmHistoryLimit`
-   `channels.discord.dms["<user_id>"].historyLimit`

Comportamiento de hilos:

-   Los hilos de Discord se enrutan como sesiones de canal
-   los metadatos del hilo padre pueden usarse para el enlace de sesión-padre
-   la configuración del hilo hereda la configuración del canal padre a menos que exista una entrada específica del hilo

Los temas del canal se inyectan como contexto **no confiable** (no como prompt del sistema).

Discord puede vincular un hilo a un objetivo de sesión para que los mensajes de seguimiento en ese hilo mantengan el enrutamiento a la misma sesión (incluidas sesiones de subagentes). Comandos:

-   `/focus ` vincular hilo actual/nuevo a un objetivo de subagente/sesión
-   `/unfocus` eliminar el vínculo del hilo actual
-   `/agents` mostrar ejecuciones activas y estado de vinculación
-   `/session idle <duration|off>` inspeccionar/actualizar inactividad auto-desvincular para vínculos enfocados
-   `/session max-age <duration|off>` inspeccionar/actualizar edad máxima dura para vínculos enfocados

Config:

```json
{
  session: {
    threadBindings: {
      enabled: true,
      idleHours: 24,
      maxAgeHours: 0,
    },
  },
  channels: {
    discord: {
      threadBindings: {
        enabled: true,
        idleHours: 24,
        maxAgeHours: 0,
        spawnSubagentSessions: false, // opt-in
      },
    },
  },
}
```

Notas:

-   `session.threadBindings.*` establece los valores predeterminados globales.
-   `channels.discord.threadBindings.*` anula el comportamiento de Discord.
-   `spawnSubagentSessions` debe ser true para crear/vincular automáticamente hilos para `sessions_spawn({ thread: true })`.
-   `spawnAcpSessions` debe ser true para crear/vincular automáticamente hilos para ACP (`/acp spawn ... --thread ...` o `sessions_spawn({ runtime: "acp", thread: true })`).
-   Si los vínculos de hilo están deshabilitados para una cuenta, `/focus` y las operaciones de vinculación de hilos relacionadas no están disponibles.

Consulta [Sub-agentes](../tools/subagents.md), [Agentes ACP](../tools/acp-agents.md) y [Referencia de Configuración](../gateway/configuration-reference.md).

Para espacios de trabajo ACP "siempre activos" estables, configura enlaces ACP de alto nivel tipados que apunten a conversaciones de Discord. Ruta de configuración:

-   `bindings[]` con `type: "acp"` y `match.channel: "discord"`

Ejemplo:

```json
{
  agents: {
    list: [
      {
        id: "codex",
        runtime: {
          type: "acp",
          acp: {
            agent: "codex",
            backend: "acpx",
            mode: "persistent",
            cwd: "/workspace/openclaw",
          },
        },
      },
    ],
  },
  bindings: [
    {
      type: "acp",
      agentId: "codex",
      match: {
        channel: "discord",
        accountId: "default",
        peer: { kind: "channel", id: "222222222222222222" },
      },
      acp: { label: "codex-main" },
    },
  ],
  channels: {
    discord: {
      guilds: {
        "111111111111111111": {
          channels: {
            "222222222222222222": {
              requireMention: false,
            },
          },
        },
      },
    },
  },
}
```

Notas:

-   Los mensajes del hilo pueden heredar el vínculo ACP del canal padre.
-   En un canal o hilo vinculado, `/new` y `/reset` reinician la misma sesión ACP en su lugar.
-   Los vínculos de hilo temporales aún funcionan y pueden anular la resolución de objetivos mientras están activos.

Consulta [Agentes ACP](../tools/acp-agents.md) para detalles del comportamiento de vinculación.

Modo de notificación de reacción por gremio:

-   `off`
-   `own` (predeterminado)
-   `all`
-   `allowlist` (usa `guilds..users`)

Los eventos de reacción se convierten en eventos del sistema y se adjuntan a la sesión de Discord enrutada.

`ackReaction` envía un emoji de confirmación mientras OpenClaw está procesando un mensaje entrante. Orden de resolución:

-   `channels.discord.accounts..ackReaction`
-   `channels.discord.ackReaction`
-   `messages.ackReaction`
-   respaldo de emoji de identidad del agente (`agents.list[].identity.emoji`, si no ”👀”)

Notas:

-   Discord acepta emoji unicode o nombres de emoji personalizados.
-   Usa `""` para deshabilitar la reacción para un canal o cuenta.

Las escrituras de configuración iniciadas por el canal están habilitadas por defecto. Esto afecta a los flujos `/config set|unset` (cuando las características de comando están habilitadas). Deshabilitar:

```json
{
  channels: {
    discord: {
      configWrites: false,
    },
  },
}
```

Enruta el tráfico WebSocket de la puerta de enlace de Discord y las búsquedas REST de inicio (ID de aplicación + resolución de lista de permisos) a través de un proxy HTTP(S) con `channels.discord.proxy`.

```json
{
  channels: {
    discord: {
      proxy: "http://proxy.example:8080",
    },
  },
}
```

Anulación por cuenta:

```json
{
  channels: {
    discord: {
      accounts: {
        primary: {
          proxy: "http://proxy.example:8080",
        },
      },
    },
  },
}
```

Habilita la resolución de PluralKit para mapear mensajes proxy a la identidad del miembro del sistema:

```json
{
  channels: {
    discord: {
      pluralkit: {
        enabled: true,
        token: "pk_live_...", // opcional; necesario para sistemas privados
      },
    },
  },
}
```

Notas:

-   las listas de permisos pueden usar `pk:`
-   los nombres para mostrar de miembros coinciden solo por nombre/slug cuando `channels.discord.dangerouslyAllowNameMatching: true`
-   las búsquedas usan el ID de mensaje original y están restringidas por ventana de tiempo
-   si la búsqueda falla, los mensajes proxy se tratan como mensajes de bot y se descartan a menos que `allowBots=true`

Las actualizaciones de presencia se aplican cuando estableces un campo de estado o actividad, o cuando habilitas la presencia automática. Ejemplo solo de estado:

```json
{
  channels: {
    discord: {
      status: "idle",
    },
  },
}
```

Ejemplo de actividad (el estado personalizado es el tipo de actividad predeterminado):

```json
{
  channels: {
    discord: {
      activity: "Tiempo de enfoque",
      activityType: 4,
    },
  },
}
```

Ejemplo de transmisión:

```json
{
  channels: {
    discord: {
      activity: "Codificación en vivo",
      activityType: 1,
      activityUrl: "https://twitch.tv/openclaw",
    },
  },
}
```

Mapa de tipo de actividad:

-   0: Jugando
-   1: Transmitiendo (requiere `activityUrl`)
-   2: Escuchando
-   3: Viendo
-   4: Personalizado (usa el texto de actividad como estado; el emoji es opcional)
-   5: Compitiendo

Ejemplo de presencia automática (señal de salud del tiempo de ejecución):

```json
{
  channels: {
    discord: {
      autoPresence: {
        enabled: true,
        intervalMs: 30000,
        minUpdateIntervalMs: 15000,
        exhaustedText: "tokens agotados",
      },
    },
  },
}
```

La presencia automática asigna la disponibilidad del tiempo de ejecución al estado de Discord: salud => en línea, degradado o desconocido => inactivo, agotado o no disponible => no molestar. Anulaciones de texto opcionales:

-   `autoPresence.healthyText`
-   `autoPresence.degradedText`
-   `autoPresence.exhaustedText` (soporta el marcador `{reason}`)

Discord admite aprobaciones de ejecución basadas en botones en mensajes directos y puede publicar opcionalmente solicitudes de aprobación en el canal de origen. Ruta de configuración:

-   `channels.discord.execApprovals.enabled`
-   `channels.discord.execApprovals.approvers`
-   `channels.discord.execApprovals.target` (`dm` | `channel` | `both`, predeterminado: `dm`)
-   `agentFilter`, `sessionFilter`, `cleanupAfterResolve`

Cuando `target` es `channel` o `both`, la solicitud de aprobación es visible en el canal. Solo los aprobadores configurados pueden usar los botones; otros usuarios reciben una denegación efímera. Las solicitudes de aprobación incluyen el texto del comando, así que habilita la entrega en el canal solo en canales de confianza. Si el ID del canal no puede derivarse de la clave de sesión, OpenClaw vuelve a la entrega por mensaje directo.

Si las aprobaciones fallan con IDs de aprobación desconocidos, verifica la lista de aprobadores y la habilitación de la función.

Documentos relacionados: [Aprobaciones de ejecución](../tools/exec-approvals.md)

## Herramientas y puertas de acción