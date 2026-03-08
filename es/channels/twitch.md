

  Plataformas de mensajería

  
# Twitch

Soporte de chat de Twitch mediante conexión IRC. OpenClaw se conecta como un usuario de Twitch (cuenta de bot) para recibir y enviar mensajes en canales.

## Plugin requerido

Twitch se distribuye como un plugin y no está incluido en la instalación principal. Instálalo mediante CLI (registro npm):

```bash
openclaw plugins install @openclaw/twitch
```

Checkout local (cuando se ejecuta desde un repositorio git):

```bash
openclaw plugins install ./extensions/twitch
```

Detalles: [Plugins](../tools/plugin.md)

## Configuración rápida (principiante)

1.  Crea una cuenta de Twitch dedicada para el bot (o usa una cuenta existente).
2.  Genera credenciales: [Generador de tokens de Twitch](https://twitchtokengenerator.com/)
    -   Selecciona **Bot Token**
    -   Verifica que los alcances `chat:read` y `chat:write` estén seleccionados
    -   Copia el **Client ID** y el **Access Token**
3.  Encuentra tu ID de usuario de Twitch: [https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/](https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/)
4.  Configura el token:
    -   Variable de entorno: `OPENCLAW_TWITCH_ACCESS_TOKEN=...` (solo cuenta predeterminada)
    -   O en configuración: `channels.twitch.accessToken`
    -   Si ambos están configurados, la configuración tiene prioridad (la variable de entorno es solo para la cuenta predeterminada).
5.  Inicia la pasarela.

**⚠️ Importante:** Añade control de acceso (`allowFrom` o `allowedRoles`) para evitar que usuarios no autorizados activen el bot. `requireMention` es `true` por defecto. Configuración mínima:

```json
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw", // Cuenta de Twitch del bot
      accessToken: "oauth:abc123...", // Token de acceso OAuth (o usa la variable de entorno OPENCLAW_TWITCH_ACCESS_TOKEN)
      clientId: "xyz789...", // Client ID del Generador de Tokens
      channel: "vevisk", // Canal de Twitch al que unirse (requerido)
      allowFrom: ["123456789"], // (recomendado) Solo tu ID de usuario de Twitch - consíguelo en https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/
    },
  },
}
```

## Qué es

-   Un canal de Twitch propiedad de la Pasarela.
-   Enrutamiento determinista: las respuestas siempre regresan a Twitch.
-   Cada cuenta se asigna a una clave de sesión aislada `agent::twitch:`.
-   `username` es la cuenta del bot (quien autentica), `channel` es a qué sala de chat unirse.

## Configuración (detallada)

### Generar credenciales

Usa [Generador de tokens de Twitch](https://twitchtokengenerator.com/):

-   Selecciona **Bot Token**
-   Verifica que los alcances `chat:read` y `chat:write` estén seleccionados
-   Copia el **Client ID** y el **Access Token**

No se necesita registro manual de aplicación. Los tokens expiran después de varias horas.

### Configurar el bot

**Variable de entorno (solo cuenta predeterminada):**

```
OPENCLAW_TWITCH_ACCESS_TOKEN=oauth:abc123...
```

**O configuración:**

```json
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",
      accessToken: "oauth:abc123...",
      clientId: "xyz789...",
      channel: "vevisk",
    },
  },
}
```

Si se configuran tanto la variable de entorno como la configuración, esta última tiene prioridad.

### Control de acceso (recomendado)

```json
{
  channels: {
    twitch: {
      allowFrom: ["123456789"], // (recomendado) Solo tu ID de usuario de Twitch
    },
  },
}
```

Prefiere `allowFrom` para una lista blanca estricta. Usa `allowedRoles` en su lugar si deseas acceso basado en roles. **Roles disponibles:** `"moderator"`, `"owner"`, `"vip"`, `"subscriber"`, `"all"`. **¿Por qué IDs de usuario?** Los nombres de usuario pueden cambiar, permitiendo suplantación. Los IDs de usuario son permanentes. Encuentra tu ID de usuario de Twitch: [https://www.streamweasels.com/tools/convert-twitch-username-%20to-user-id/](https://www.streamweasels.com/tools/convert-twitch-username-%20to-user-id/) (Convierte tu nombre de usuario de Twitch a ID)

## Refresco de token (opcional)

Los tokens de [Generador de tokens de Twitch](https://twitchtokengenerator.com/) no se pueden refrescar automáticamente - regenéralos cuando expiren. Para refresco automático de tokens, crea tu propia aplicación de Twitch en [Consola de desarrolladores de Twitch](https://dev.twitch.tv/console) y añade a la configuración:

```json
{
  channels: {
    twitch: {
      clientSecret: "your_client_secret",
      refreshToken: "your_refresh_token",
    },
  },
}
```

El bot refresca automáticamente los tokens antes de su expiración y registra los eventos de refresco.

## Soporte multi-cuenta

Usa `channels.twitch.accounts` con tokens por cuenta. Consulta [`gateway/configuration`](../gateway/configuration.md) para el patrón compartido. Ejemplo (una cuenta de bot en dos canales):

```json
{
  channels: {
    twitch: {
      accounts: {
        channel1: {
          username: "openclaw",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "vevisk",
        },
        channel2: {
          username: "openclaw",
          accessToken: "oauth:def456...",
          clientId: "uvw012...",
          channel: "secondchannel",
        },
      },
    },
  },
}
```

**Nota:** Cada cuenta necesita su propio token (un token por canal).

## Control de acceso

### Restricciones basadas en roles

```json
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowedRoles: ["moderator", "vip"],
        },
      },
    },
  },
}
```

### Lista blanca por ID de usuario (más seguro)

```json
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowFrom: ["123456789", "987654321"],
        },
      },
    },
  },
}
```

### Acceso basado en roles (alternativa)

`allowFrom` es una lista blanca estricta. Cuando está configurada, solo esos IDs de usuario están permitidos. Si deseas acceso basado en roles, deja `allowFrom` sin configurar y configura `allowedRoles` en su lugar:

```json
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowedRoles: ["moderator"],
        },
      },
    },
  },
}
```

### Desactivar el requisito de mención @

Por defecto, `requireMention` es `true`. Para desactivarlo y responder a todos los mensajes:

```json
{
  channels: {
    twitch: {
      accounts: {
        default: {
          requireMention: false,
        },
      },
    },
  },
}
```

## Solución de problemas

Primero, ejecuta comandos de diagnóstico:

```bash
openclaw doctor
openclaw channels status --probe
```

### El bot no responde a los mensajes

**Verifica el control de acceso:** Asegúrate de que tu ID de usuario esté en `allowFrom`, o elimina temporalmente `allowFrom` y configura `allowedRoles: ["all"]` para probar. **Verifica que el bot esté en el canal:** El bot debe unirse al canal especificado en `channel`.

### Problemas con el token

**“Error de conexión” o errores de autenticación:**

-   Verifica que `accessToken` sea el valor del token de acceso OAuth (normalmente comienza con el prefijo `oauth:`)
-   Comprueba que el token tenga los alcances `chat:read` y `chat:write`
-   Si usas refresco de token, verifica que `clientSecret` y `refreshToken` estén configurados

### El refresco de token no funciona

**Revisa los registros en busca de eventos de refresco:**

```bash
Using env token source for mybot
Access token refreshed for user 123456 (expires in 14400s)
```

Si ves “token refresh disabled (no refresh token)”:

-   Asegúrate de que se proporcione `clientSecret`
-   Asegúrate de que se proporcione `refreshToken`

## Configuración

**Configuración de cuenta:**

-   `username` - Nombre de usuario del bot
-   `accessToken` - Token de acceso OAuth con `chat:read` y `chat:write`
-   `clientId` - Client ID de Twitch (del Generador de Tokens o tu aplicación)
-   `channel` - Canal al que unirse (requerido)
-   `enabled` - Habilitar esta cuenta (predeterminado: `true`)
-   `clientSecret` - Opcional: Para refresco automático de token
-   `refreshToken` - Opcional: Para refresco automático de token
-   `expiresIn` - Expiración del token en segundos
-   `obtainmentTimestamp` - Marca de tiempo de obtención del token
-   `allowFrom` - Lista blanca de IDs de usuario
-   `allowedRoles` - Control de acceso basado en roles (`"moderator" | "owner" | "vip" | "subscriber" | "all"`)
-   `requireMention` - Requerir mención @ (predeterminado: `true`)

**Opciones del proveedor:**

-   `channels.twitch.enabled` - Habilitar/deshabilitar inicio del canal
-   `channels.twitch.username` - Nombre de usuario del bot (configuración simplificada de una sola cuenta)
-   `channels.twitch.accessToken` - Token de acceso OAuth (configuración simplificada de una sola cuenta)
-   `channels.twitch.clientId` - Client ID de Twitch (configuración simplificada de una sola cuenta)
-   `channels.twitch.channel` - Canal al que unirse (configuración simplificada de una sola cuenta)
-   `channels.twitch.accounts.` - Configuración multi-cuenta (todos los campos de cuenta anteriores)

Ejemplo completo:

```json
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",
      accessToken: "oauth:abc123...",
      clientId: "xyz789...",
      channel: "vevisk",
      clientSecret: "secret123...",
      refreshToken: "refresh456...",
      allowFrom: ["123456789"],
      allowedRoles: ["moderator", "vip"],
      accounts: {
        default: {
          username: "mybot",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "your_channel",
          enabled: true,
          clientSecret: "secret123...",
          refreshToken: "refresh456...",
          expiresIn: 14400,
          obtainmentTimestamp: 1706092800000,
          allowFrom: ["123456789", "987654321"],
          allowedRoles: ["moderator"],
        },
      },
    },
  },
}
```

## Acciones de herramienta

El agente puede llamar a `twitch` con la acción:

-   `send` - Enviar un mensaje a un canal

Ejemplo:

```json
{
  action: "twitch",
  params: {
    message: "Hello Twitch!",
    to: "#mychannel",
  },
}
```

## Seguridad y operaciones

-   **Trata los tokens como contraseñas** - Nunca los guardes en git
-   **Usa refresco automático de token** para bots de larga duración
-   **Usa listas blancas de ID de usuario** en lugar de nombres de usuario para control de acceso
-   **Monitorea los registros** en busca de eventos de refresco de token y estado de conexión
-   **Limita los alcances de los tokens** - Solicita solo `chat:read` y `chat:write`
-   **Si se bloquea**: Reinicia la pasarela después de confirmar que ningún otro proceso posee la sesión

## Límites

-   **500 caracteres** por mensaje (dividido automáticamente en límites de palabras)
-   El Markdown se elimina antes de la división
-   Sin limitación de tasa (usa los límites de tasa incorporados de Twitch)

[Tlon](./tlon.md)[WhatsApp](./whatsapp.md)