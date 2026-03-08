

  Plataformas de mensajería

  
# Feishu

Feishu (Lark) es una plataforma de chat para equipos utilizada por empresas para mensajería y colaboración. Este plugin conecta OpenClaw a un bot de Feishu/Lark usando la suscripción a eventos WebSocket de la plataforma, permitiendo recibir mensajes sin exponer una URL de webhook pública.

* * *

## Plugin requerido

Instala el plugin de Feishu:

```bash
openclaw plugins install @openclaw/feishu
```

Instalación local (cuando se ejecuta desde un repositorio git):

```bash
openclaw plugins install ./extensions/feishu
```

* * *

## Inicio rápido

Hay dos formas de agregar el canal de Feishu:

### Método 1: asistente de configuración (recomendado)

Si acabas de instalar OpenClaw, ejecuta el asistente:

```bash
openclaw onboard
```

El asistente te guiará a través de:

1.  Crear una aplicación de Feishu y recopilar credenciales
2.  Configurar las credenciales de la aplicación en OpenClaw
3.  Iniciar la puerta de enlace (gateway)

✅ **Después de la configuración**, verifica el estado de la puerta de enlace:

-   `openclaw gateway status`
-   `openclaw logs --follow`

### Método 2: configuración por CLI

Si ya completaste la instalación inicial, agrega el canal mediante la CLI:

```bash
openclaw channels add
```

Elige **Feishu**, luego ingresa el ID de la App y el Secreto de la App. ✅ **Después de la configuración**, gestiona la puerta de enlace:

-   `openclaw gateway status`
-   `openclaw gateway restart`
-   `openclaw logs --follow`

* * *

## Paso 1: Crear una aplicación de Feishu

### 1\. Abrir la Plataforma Abierta de Feishu

Visita la [Plataforma Abierta de Feishu](https://open.feishu.cn/app) e inicia sesión. Los inquilinos de Lark (global) deben usar [https://open.larksuite.com/app](https://open.larksuite.com/app) y configurar `domain: "lark"` en la configuración de Feishu.

### 2\. Crear una aplicación

1.  Haz clic en **Crear aplicación empresarial**
2.  Completa el nombre de la aplicación + descripción
3.  Elige un icono para la aplicación

![Crear aplicación empresarial](../images/channels-feishu-step2-create-app.png.md)

### 3\. Copiar credenciales

Desde **Credenciales e información básica**, copia:

-   **ID de la App** (formato: `cli_xxx`)
-   **Secreto de la App**

❗ **Importante:** mantén el Secreto de la App en privado. ![Obtener credenciales](../images/channels-feishu-step3-credentials.png.md)

### 4\. Configurar permisos

En **Permisos**, haz clic en **Importar por lotes** y pega:

```json
{
  "scopes": {
    "tenant": [
      "aily:file:read",
      "aily:file:write",
      "application:application.app_message_stats.overview:readonly",
      "application:application:self_manage",
      "application:bot.menu:write",
      "cardkit:card:read",
      "cardkit:card:write",
      "contact:user.employee_id:readonly",
      "corehr:file:download",
      "event:ip_list",
      "im:chat.access_event.bot_p2p_chat:read",
      "im:chat.members:bot_access",
      "im:message",
      "im:message.group_at_msg:readonly",
      "im:message.p2p_msg:readonly",
      "im:message:readonly",
      "im:message:send_as_bot",
      "im:resource"
    ],
    "user": ["aily:file:read", "aily:file:write", "im:chat.access_event.bot_p2p_chat:read"]
  }
}
```

![Configurar permisos](../images/channels-feishu-step4-permissions.png.md)

### 5\. Habilitar la capacidad de bot

En **Capacidades de la App** > **Bot**:

1.  Habilita la capacidad de bot
2.  Establece el nombre del bot

![Habilitar capacidad de bot](../images/channels-feishu-step5-bot-capability.png.md)

### 6\. Configurar suscripción a eventos

⚠️ **Importante:** antes de configurar la suscripción a eventos, asegúrate de:

1.  Ya haber ejecutado `openclaw channels add` para Feishu
2.  La puerta de enlace esté en ejecución (`openclaw gateway status`)

En **Suscripción a eventos**:

1.  Elige **Usar conexión larga para recibir eventos** (WebSocket)
2.  Agrega el evento: `im.message.receive_v1`

⚠️ Si la puerta de enlace no está en ejecución, la configuración de conexión larga podría fallar al guardarse. ![Configurar suscripción a eventos](../images/channels-feishu-step6-event-subscription.png.md)

### 7\. Publicar la aplicación

1.  Crea una versión en **Gestión de versiones y publicación**
2.  Envíala para revisión y publícala
3.  Espera la aprobación del administrador (las aplicaciones empresariales suelen auto-aprobarse)

* * *

## Paso 2: Configurar OpenClaw

### Configurar con el asistente (recomendado)

```bash
openclaw channels add
```

Elige **Feishu** y pega tu ID de App + Secreto de la App.

### Configurar mediante archivo de configuración

Edita `~/.openclaw/openclaw.json`:

```json
{
  channels: {
    feishu: {
      enabled: true,
      dmPolicy: "pairing",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
          botName: "Mi asistente de IA",
        },
      },
    },
  },
}
```

Si usas `connectionMode: "webhook"`, configura `verificationToken`. El servidor de webhook de Feishu se vincula a `127.0.0.1` por defecto; configura `webhookHost` solo si intencionalmente necesitas una dirección de enlace diferente.

#### Token de verificación (modo webhook)

Cuando uses el modo webhook, configura `channels.feishu.verificationToken` en tu configuración. Para obtener el valor:

1.  En la Plataforma Abierta de Feishu, abre tu aplicación
2.  Ve a **Desarrollo** → **Eventos y devoluciones de llamada** (开发配置 → 事件与回调)
3.  Abre la pestaña **Cifrado** (加密策略)
4.  Copia el **Token de verificación**

![Ubicación del Token de verificación](../images/channels-feishu-verification-token.png.md)

### Configurar mediante variables de entorno

```bash
export FEISHU_APP_ID="cli_xxx"
export FEISHU_APP_SECRET="xxx"
```

### Dominio de Lark (global)

Si tu inquilino está en Lark (internacional), configura el dominio a `lark` (o una cadena de dominio completa). Puedes configurarlo en `channels.feishu.domain` o por cuenta (`channels.feishu.accounts..domain`).

```json
{
  channels: {
    feishu: {
      domain: "lark",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
        },
      },
    },
  },
}
```

### Banderas de optimización de cuota

Puedes reducir el uso de la API de Feishu con dos banderas opcionales:

-   `typingIndicator` (por defecto `true`): cuando es `false`, omite las llamadas de reacción de escritura.
-   `resolveSenderNames` (por defecto `true`): cuando es `false`, omite las llamadas de búsqueda de perfil del remitente.

Configúralas a nivel superior o por cuenta:

```json
{
  channels: {
    feishu: {
      typingIndicator: false,
      resolveSenderNames: false,
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
          typingIndicator: true,
          resolveSenderNames: false,
        },
      },
    },
  },
}
```

* * *

## Paso 3: Iniciar + probar

### 1\. Iniciar la puerta de enlace

```bash
openclaw gateway
```

### 2\. Enviar un mensaje de prueba

En Feishu, encuentra tu bot y envía un mensaje.

### 3\. Aprobar el emparejamiento

Por defecto, el bot responde con un código de emparejamiento. Apruebalo:

```bash
openclaw pairing approve feishu <CODE>
```

Después de la aprobación, puedes chatear normalmente.

* * *

## Resumen

-   **Canal de bot de Feishu**: Bot de Feishu gestionado por la puerta de enlace
-   **Enrutamiento determinista**: las respuestas siempre regresan a Feishu
-   **Aislamiento de sesión**: los mensajes directos comparten una sesión principal; los grupos están aislados
-   **Conexión WebSocket**: conexión larga a través del SDK de Feishu, no se necesita una URL pública

* * *

## Control de acceso

### Mensajes directos

-   **Por defecto**: `dmPolicy: "pairing"` (los usuarios desconocidos reciben un código de emparejamiento)
-   **Aprobar emparejamiento**:
    
    Copiar
    
    ```bash
    openclaw pairing list feishu
    openclaw pairing approve feishu <CODE>
    ```
    
-   **Modo lista de permitidos**: configura `channels.feishu.allowFrom` con los Open IDs permitidos

### Chats de grupo

**1\. Política de grupo** (`channels.feishu.groupPolicy`):

-   `"open"` = permitir a todos en los grupos (por defecto)
-   `"allowlist"` = solo permitir `groupAllowFrom`
-   `"disabled"` = deshabilitar mensajes de grupo

**2\. Requerir mención** (`channels.feishu.groups.<chat_id>.requireMention`):

-   `true` = requiere mención @ (por defecto)
-   `false` = responde sin menciones

* * *

## Ejemplos de configuración de grupo

### Permitir todos los grupos, requerir mención @ (por defecto)

```json
{
  channels: {
    feishu: {
      groupPolicy: "open",
      // Por defecto requireMention: true
    },
  },
}
```

### Permitir todos los grupos, no se requiere mención @

```json
{
  channels: {
    feishu: {
      groups: {
        oc_xxx: { requireMention: false },
      },
    },
  },
}
```

### Permitir solo grupos específicos

```json
{
  channels: {
    feishu: {
      groupPolicy: "allowlist",
      // Los IDs de grupo de Feishu (chat_id) tienen este aspecto: oc_xxx
      groupAllowFrom: ["oc_xxx", "oc_yyy"],
    },
  },
}
```

### Restringir qué remitentes pueden enviar mensajes en un grupo (lista de permitidos de remitentes)

Además de permitir el grupo en sí, **todos los mensajes** en ese grupo están controlados por el open\_id del remitente: solo los usuarios listados en `groups.<chat_id>.allowFrom` tienen sus mensajes procesados; los mensajes de otros miembros se ignoran (esto es un control a nivel de remitente completo, no solo para comandos de control como /reset o /new).

```json
{
  channels: {
    feishu: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["oc_xxx"],
      groups: {
        oc_xxx: {
          // Los IDs de usuario de Feishu (open_id) tienen este aspecto: ou_xxx
          allowFrom: ["ou_user1", "ou_user2"],
        },
      },
    },
  },
}
```

* * *

## Obtener IDs de grupo/usuario

### IDs de grupo (chat\_id)

Los IDs de grupo tienen este aspecto `oc_xxx`. **Método 1 (recomendado)**

1.  Inicia la puerta de enlace y menciona al bot en el grupo con @
2.  Ejecuta `openclaw logs --follow` y busca `chat_id`

**Método 2** Usa el depurador de la API de Feishu para listar los chats de grupo.

### IDs de usuario (open\_id)

Los IDs de usuario tienen este aspecto `ou_xxx`. **Método 1 (recomendado)**

1.  Inicia la puerta de enlace y envía un mensaje directo al bot
2.  Ejecuta `openclaw logs --follow` y busca `open_id`

**Método 2** Revisa las solicitudes de emparejamiento para obtener los Open IDs de usuario:

```bash
openclaw pairing list feishu
```

* * *

## Comandos comunes

| Comando | Descripción |
| --- | --- |
| `/status` | Mostrar el estado del bot |
| `/reset` | Reiniciar la sesión |
| `/model` | Mostrar/cambiar modelo |

> Nota: Feishu aún no admite menús de comandos nativos, por lo que los comandos deben enviarse como texto.

## Comandos de gestión de la puerta de enlace

| Comando | Descripción |
| --- | --- |
| `openclaw gateway status` | Mostrar el estado de la puerta de enlace |
| `openclaw gateway install` | Instalar/iniciar el servicio de puerta de enlace |
| `openclaw gateway stop` | Detener el servicio de puerta de enlace |
| `openclaw gateway restart` | Reiniciar el servicio de puerta de enlace |
| `openclaw logs --follow` | Seguir los registros de la puerta de enlace |

* * *

## Solución de problemas

### El bot no responde en chats de grupo

1.  Asegúrate de que el bot esté agregado al grupo
2.  Asegúrate de mencionar al bot con @ (comportamiento por defecto)
3.  Verifica que `groupPolicy` no esté configurado como `"disabled"`
4.  Revisa los registros: `openclaw logs --follow`

### El bot no recibe mensajes

1.  Asegúrate de que la aplicación esté publicada y aprobada
2.  Asegúrate de que la suscripción a eventos incluya `im.message.receive_v1`
3.  Asegúrate de que la **conexión larga** esté habilitada
4.  Asegúrate de que los permisos de la aplicación estén completos
5.  Asegúrate de que la puerta de enlace esté en ejecución: `openclaw gateway status`
6.  Revisa los registros: `openclaw logs --follow`

### Filtración del Secreto de la App

1.  Restablece el Secreto de la App en la Plataforma Abierta de Feishu
2.  Actualiza el Secreto de la App en tu configuración
3.  Reinicia la puerta de enlace

### Fallos al enviar mensajes

1.  Asegúrate de que la aplicación tenga el permiso `im:message:send_as_bot`
2.  Asegúrate de que la aplicación esté publicada
3.  Revisa los registros para ver errores detallados

* * *

## Configuración avanzada

### Múltiples cuentas

```json
{
  channels: {
    feishu: {
      defaultAccount: "main",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
          botName: "Bot principal",
        },
        backup: {
          appId: "cli_yyy",
          appSecret: "yyy",
          botName: "Bot de respaldo",
          enabled: false,
        },
      },
    },
  },
}
```

`defaultAccount` controla qué cuenta de Feishu se usa cuando las APIs salientes no especifican explícitamente un `accountId`.

### Límites de mensajes

-   `textChunkLimit`: tamaño del fragmento de texto saliente (por defecto: 2000 caracteres)
-   `mediaMaxMb`: límite de carga/descarga de medios (por defecto: 30MB)

### Streaming

Feishu admite respuestas en streaming a través de tarjetas interactivas. Cuando está habilitado, el bot actualiza una tarjeta a medida que genera texto.

```json
{
  channels: {
    feishu: {
      streaming: true, // habilitar salida de tarjeta en streaming (por defecto true)
      blockStreaming: true, // habilitar streaming a nivel de bloque (por defecto true)
    },
  },
}
```

Configura `streaming: false` para esperar la respuesta completa antes de enviar.

### Enrutamiento multi-agente

Usa `bindings` para enrutar mensajes directos o grupos de Feishu a diferentes agentes.

```json
{
  agents: {
    list: [
      { id: "main" },
      {
        id: "clawd-fan",
        workspace: "/home/user/clawd-fan",
        agentDir: "/home/user/.openclaw/agents/clawd-fan/agent",
      },
      {
        id: "clawd-xi",
        workspace: "/home/user/clawd-xi",
        agentDir: "/home/user/.openclaw/agents/clawd-xi/agent",
      },
    ],
  },
  bindings: [
    {
      agentId: "main",
      match: {
        channel: "feishu",
        peer: { kind: "direct", id: "ou_xxx" },
      },
    },
    {
      agentId: "clawd-fan",
      match: {
        channel: "feishu",
        peer: { kind: "direct", id: "ou_yyy" },
      },
    },
    {
      agentId: "clawd-xi",
      match: {
        channel: "feishu",
        peer: { kind: "group", id: "oc_zzz" },
      },
    },
  ],
}
```

Campos de enrutamiento:

-   `match.channel`: `"feishu"`
-   `match.peer.kind`: `"direct"` o `"group"`
-   `match.peer.id`: Open ID de usuario (`ou_xxx`) o ID de grupo (`oc_xxx`)

Consulta [Obtener IDs de grupo/usuario](#get-groupuser-ids) para consejos de búsqueda.

* * *

## Referencia de configuración

Configuración completa: [Configuración de la puerta de enlace](../gateway/configuration.md) Opciones clave:

| Configuración | Descripción | Por defecto |
| --- | --- | --- |
| `channels.feishu.enabled` | Habilitar/deshabilitar canal | `true` |
| `channels.feishu.domain` | Dominio de la API (`feishu` o `lark`) | `feishu` |
| `channels.feishu.connectionMode` | Modo de transporte de eventos | `websocket` |
| `channels.feishu.defaultAccount` | ID de cuenta por defecto para enrutamiento saliente | `default` |
| `channels.feishu.verificationToken` | Requerido para el modo webhook | \- |
| `channels.feishu.webhookPath` | Ruta de la ruta del webhook | `/feishu/events` |
| `channels.feishu.webhookHost` | Host de enlace del webhook | `127.0.0.1` |
| `channels.feishu.webhookPort` | Puerto de enlace del webhook | `3000` |
| `channels.feishu.accounts..appId` | ID de la App | \- |
| `channels.feishu.accounts..appSecret` | Secreto de la App | \- |
| `channels.feishu.accounts..domain` | Anulación de dominio de API por cuenta | `feishu` |
| `channels.feishu.dmPolicy` | Política de mensajes directos | `pairing` |
| `channels.feishu.allowFrom` | Lista de permitidos para MD (lista de open\_id) | \- |
| `channels.feishu.groupPolicy` | Política de grupo | `open` |
| `channels.feishu.groupAllowFrom` | Lista de permitidos de grupo | \- |
| `channels.feishu.groups.<chat_id>.requireMention` | Requerir mención @ | `true` |
| `channels.feishu.groups.<chat_id>.enabled` | Habilitar grupo | `true` |
| `channels.feishu.textChunkLimit` | Tamaño del fragmento de mensaje | `2000` |
| `channels.feishu.mediaMaxMb` | Límite de tamaño de medios | `30` |
| `channels.feishu.streaming` | Habilitar salida de tarjeta en streaming | `true` |
| `channels.feishu.blockStreaming` | Habilitar streaming de bloque | `true` |

* * *

## Referencia de dmPolicy

| Valor | Comportamiento |
| --- | --- |
| `"pairing"` | **Por defecto.** Los usuarios desconocidos reciben un código de emparejamiento; debe ser aprobado |
| `"allowlist"` | Solo los usuarios en `allowFrom` pueden chatear |
| `"open"` | Permitir a todos los usuarios (requiere `"*"` en allowFrom) |
| `"disabled"` | Deshabilitar mensajes directos |

* * *

## Tipos de mensajes admitidos

### Recibir

-   ✅ Texto
-   ✅ Texto enriquecido (post)
-   ✅ Imágenes
-   ✅ Archivos
-   ✅ Audio
-   ✅ Video
-   ✅ Stickers

### Enviar

-   ✅ Texto
-   ✅ Imágenes
-   ✅ Archivos
-   ✅ Audio
-   ⚠️ Texto enriquecido (soporte parcial)

[Discord](./discord.md)[Google Chat](./googlechat.md)