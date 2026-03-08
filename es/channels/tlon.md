

  Plataformas de mensajería

  
# Tlon

Tlon es un mensajero descentralizado construido sobre Urbit. OpenClaw se conecta a tu nave Urbit y puede responder a mensajes directos y mensajes de chat grupal. Las respuestas grupales requieren una mención @ por defecto y pueden restringirse aún más mediante listas de permitidos. Estado: compatible mediante complemento. Se admiten mensajes directos, menciones grupales, respuestas en hilos, formato de texto enriquecido y carga de imágenes. Las reacciones y encuestas aún no son compatibles.

## Complemento requerido

Tlon se distribuye como un complemento y no está incluido en la instalación principal. Instálalo mediante CLI (registro npm):

```bash
openclaw plugins install @openclaw/tlon
```

Instalación local (cuando se ejecuta desde un repositorio git):

```bash
openclaw plugins install ./extensions/tlon
```

Detalles: [Complementos](../tools/plugin.md)

## Configuración

1.  Instala el complemento Tlon.
2.  Obtén la URL de tu nave y el código de inicio de sesión.
3.  Configura `channels.tlon`.
4.  Reinicia la pasarela.
5.  Envía un mensaje directo al bot o menciónalo en un canal grupal.

Configuración mínima (cuenta única):

```json
{
  channels: {
    tlon: {
      enabled: true,
      ship: "~sampel-palnet",
      url: "https://your-ship-host",
      code: "lidlut-tabwed-pillex-ridrup",
      ownerShip: "~your-main-ship", // recomendado: tu nave, siempre permitida
    },
  },
}
```

## Naves privadas/en LAN

Por defecto, OpenClaw bloquea nombres de host privados/internos y rangos de IP para protección SSRF. Si tu nave se ejecuta en una red privada (localhost, IP LAN o nombre de host interno), debes habilitarlo explícitamente:

```json
{
  channels: {
    tlon: {
      url: "http://localhost:8080",
      allowPrivateNetwork: true,
    },
  },
}
```

Esto se aplica a URLs como:

-   `http://localhost:8080`
-   `http://192.168.x.x:8080`
-   `http://my-ship.local:8080`

⚠️ Habilita esto solo si confías en tu red local. Esta configuración desactiva las protecciones SSRF para las solicitudes a la URL de tu nave.

## Canales grupales

La auto-detección está habilitada por defecto. También puedes fijar canales manualmente:

```json
{
  channels: {
    tlon: {
      groupChannels: ["chat/~host-ship/general", "chat/~host-ship/support"],
    },
  },
}
```

Deshabilitar la auto-detección:

```json
{
  channels: {
    tlon: {
      autoDiscoverChannels: false,
    },
  },
}
```

## Control de acceso

Lista de permitidos para mensajes directos (vacía = no se permiten mensajes directos, usa `ownerShip` para flujo de aprobación):

```json
{
  channels: {
    tlon: {
      dmAllowlist: ["~zod", "~nec"],
    },
  },
}
```

Autorización grupal (restringida por defecto):

```json
{
  channels: {
    tlon: {
      defaultAuthorizedShips: ["~zod"],
      authorization: {
        channelRules: {
          "chat/~host-ship/general": {
            mode: "restricted",
            allowedShips: ["~zod", "~nec"],
          },
          "chat/~host-ship/announcements": {
            mode: "open",
          },
        },
      },
    },
  },
}
```

## Propietario y sistema de aprobación

Establece una nave propietaria para recibir solicitudes de aprobación cuando usuarios no autorizados intenten interactuar:

```json
{
  channels: {
    tlon: {
      ownerShip: "~your-main-ship",
    },
  },
}
```

La nave propietaria está **autorizada automáticamente en todas partes**: las invitaciones a mensajes directos se aceptan automáticamente y los mensajes en canales siempre están permitidos. No necesitas agregar al propietario a `dmAllowlist` o `defaultAuthorizedShips`. Cuando está configurado, el propietario recibe notificaciones por mensaje directo para:

-   Solicitudes de mensaje directo de naves no en la lista de permitidos
-   Menciones en canales sin autorización
-   Solicitudes de invitación a grupos

## Configuración de auto-aceptación

Auto-aceptar invitaciones a mensajes directos (para naves en dmAllowlist):

```json
{
  channels: {
    tlon: {
      autoAcceptDmInvites: true,
    },
  },
}
```

Auto-aceptar invitaciones a grupos:

```json
{
  channels: {
    tlon: {
      autoAcceptGroupInvites: true,
    },
  },
}
```

## Destinos de entrega (CLI/cron)

Úsalos con `openclaw message send` o entrega por cron:

-   Mensaje directo: `~sampel-palnet` o `dm/~sampel-palnet`
-   Grupo: `chat/~host-ship/channel` o `group:~host-ship/channel`

## Habilidad incluida

El complemento Tlon incluye una habilidad integrada ([`@tloncorp/tlon-skill`](https://github.com/tloncorp/tlon-skill)) que proporciona acceso por CLI a operaciones de Tlon:

-   **Contactos**: obtener/actualizar perfiles, listar contactos
-   **Canales**: listar, crear, publicar mensajes, obtener historial
-   **Grupos**: listar, crear, gestionar miembros
-   **Mensajes directos**: enviar mensajes, reaccionar a mensajes
-   **Reacciones**: agregar/eliminar reacciones de emoji a publicaciones y mensajes directos
-   **Configuración**: gestionar permisos del complemento mediante comandos de barra diagonal

La habilidad está disponible automáticamente cuando se instala el complemento.

## Capacidades

| Característica | Estado |
| --- | --- |
| Mensajes directos | ✅ Compatible |
| Grupos/canales | ✅ Compatible (restringido por mención por defecto) |
| Hilos | ✅ Compatible (respuestas automáticas en el hilo) |
| Texto enriquecido | ✅ Markdown convertido al formato Tlon |
| Imágenes | ✅ Cargadas al almacenamiento de Tlon |
| Reacciones | ✅ Mediante [habilidad incluida](#habilidad-incluida) |
| Encuestas | ❌ Aún no compatible |
| Comandos nativos | ✅ Compatible (solo propietario por defecto) |

## Solución de problemas

Ejecuta esta escalera primero:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
```

Fallos comunes:

-   **Mensajes directos ignorados**: el remitente no está en `dmAllowlist` y no hay `ownerShip` configurado para el flujo de aprobación.
-   **Mensajes grupales ignorados**: canal no detectado o remitente no autorizado.
-   **Errores de conexión**: verifica que la URL de la nave sea accesible; habilita `allowPrivateNetwork` para naves locales.
-   **Errores de autenticación**: verifica que el código de inicio de sesión sea actual (los códigos rotan).

## Referencia de configuración

Configuración completa: [Configuración](../gateway/configuration.md) Opciones del proveedor:

-   `channels.tlon.enabled`: habilitar/deshabilitar el inicio del canal.
-   `channels.tlon.ship`: nombre de la nave Urbit del bot (ej. `~sampel-palnet`).
-   `channels.tlon.url`: URL de la nave (ej. `https://sampel-palnet.tlon.network`).
-   `channels.tlon.code`: código de inicio de sesión de la nave.
-   `channels.tlon.allowPrivateNetwork`: permitir URLs de localhost/LAN (omisión SSRF).
-   `channels.tlon.ownerShip`: nave propietaria para el sistema de aprobación (siempre autorizada).
-   `channels.tlon.dmAllowlist`: naves permitidas para mensajes directos (vacía = ninguna).
-   `channels.tlon.autoAcceptDmInvites`: auto-aceptar mensajes directos de naves en la lista de permitidos.
-   `channels.tlon.autoAcceptGroupInvites`: auto-aceptar todas las invitaciones a grupos.
-   `channels.tlon.autoDiscoverChannels`: auto-detectar canales grupales (por defecto: true).
-   `channels.tlon.groupChannels`: nidos de canales fijados manualmente.
-   `channels.tlon.defaultAuthorizedShips`: naves autorizadas para todos los canales.
-   `channels.tlon.authorization.channelRules`: reglas de autorización por canal.
-   `channels.tlon.showModelSignature`: agregar el nombre del modelo a los mensajes.

## Notas

-   Las respuestas grupales requieren una mención (ej. `~your-bot-ship`) para responder.
-   Respuestas en hilos: si el mensaje entrante está en un hilo, OpenClaw responde dentro del hilo.
-   Texto enriquecido: el formato Markdown (negrita, cursiva, código, encabezados, listas) se convierte al formato nativo de Tlon.
-   Imágenes: las URLs se cargan al almacenamiento de Tlon y se incrustan como bloques de imagen.

[Telegram](./telegram.md)[Twitch](./twitch.md)