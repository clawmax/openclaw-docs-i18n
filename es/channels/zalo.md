

  Plataformas de mensajería

  
# Zalo

Estado: experimental. Los DMs están soportados; el manejo de grupos está disponible con controles de política de grupo explícitos.

## Plugin requerido

Zalo se distribuye como un plugin y no está incluido en la instalación principal.

-   Instalar vía CLI: `openclaw plugins install @openclaw/zalo`
-   O selecciona **Zalo** durante la incorporación y confirma el mensaje de instalación
-   Detalles: [Plugins](../tools/plugin.md)

## Configuración rápida (principiante)

1.  Instala el plugin de Zalo:
    -   Desde un checkout de fuente: `openclaw plugins install ./extensions/zalo`
    -   Desde npm (si está publicado): `openclaw plugins install @openclaw/zalo`
    -   O elige **Zalo** en la incorporación y confirma el mensaje de instalación
2.  Establece el token:
    -   Variable de entorno: `ZALO_BOT_TOKEN=...`
    -   O en configuración: `channels.zalo.botToken: "..."`.
3.  Reinicia el gateway (o finaliza la incorporación).
4.  El acceso por DM es por emparejamiento por defecto; aprueba el código de emparejamiento en el primer contacto.

Configuración mínima:

```json
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "12345689:abc-xyz",
      dmPolicy: "pairing",
    },
  },
}
```

## Qué es

Zalo es una aplicación de mensajería centrada en Vietnam; su Bot API permite que el Gateway ejecute un bot para conversaciones 1:1. Es una buena opción para soporte o notificaciones donde se desee un enrutamiento determinístico de vuelta a Zalo.

-   Un canal de Zalo Bot API propiedad del Gateway.
-   Enrutamiento determinístico: las respuestas vuelven a Zalo; el modelo nunca elige canales.
-   Los DMs comparten la sesión principal del agente.
-   Los grupos están soportados con controles de política (`groupPolicy` + `groupAllowFrom`) y por defecto tienen un comportamiento de lista de permitidos cerrado ante fallos.

## Configuración (ruta rápida)

### 1) Crear un token de bot (Zalo Bot Platform)

1.  Ve a [https://bot.zaloplatforms.com](https://bot.zaloplatforms.com) e inicia sesión.
2.  Crea un nuevo bot y configura sus ajustes.
3.  Copia el token del bot (formato: `12345689:abc-xyz`).

### 2) Configurar el token (variable de entorno o configuración)

Ejemplo:

```json
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "12345689:abc-xyz",
      dmPolicy: "pairing",
    },
  },
}
```

Opción de variable de entorno: `ZALO_BOT_TOKEN=...` (funciona solo para la cuenta por defecto). Soporte multi-cuenta: usa `channels.zalo.accounts` con tokens por cuenta y opcionalmente `name`.

3.  Reinicia el gateway. Zalo se inicia cuando se resuelve un token (variable de entorno o configuración).
4.  El acceso por DM por defecto es por emparejamiento. Aprueba el código cuando el bot sea contactado por primera vez.

## Cómo funciona (comportamiento)

-   Los mensajes entrantes se normalizan en el sobre de canal compartido con marcadores de posición para medios.
-   Las respuestas siempre se enrutan de vuelta al mismo chat de Zalo.
-   Long-polling por defecto; modo webhook disponible con `channels.zalo.webhookUrl`.

## Límites

-   El texto saliente se divide en fragmentos de 2000 caracteres (límite de la API de Zalo).
-   Las descargas/subidas de medios están limitadas por `channels.zalo.mediaMaxMb` (por defecto 5).
-   El streaming está bloqueado por defecto debido a que el límite de 2000 caracteres hace que el streaming sea menos útil.

## Control de acceso (DMs)

### Acceso por DM

-   Por defecto: `channels.zalo.dmPolicy = "pairing"`. Los remitentes desconocidos reciben un código de emparejamiento; los mensajes se ignoran hasta que sean aprobados (los códigos expiran después de 1 hora).
-   Aprueba mediante:
    -   `openclaw pairing list zalo`
    -   `openclaw pairing approve zalo <CÓDIGO>`
-   El emparejamiento es el intercambio de tokens por defecto. Detalles: [Emparejamiento](./pairing.md)
-   `channels.zalo.allowFrom` acepta IDs de usuario numéricos (no hay búsqueda de nombre de usuario disponible).

## Control de acceso (Grupos)

-   `channels.zalo.groupPolicy` controla el manejo de mensajes entrantes en grupos: `open | allowlist | disabled`.
-   El comportamiento por defecto es cerrado ante fallos: `allowlist`.
-   `channels.zalo.groupAllowFrom` restringe qué IDs de remitente pueden activar el bot en grupos.
-   Si `groupAllowFrom` no está establecido, Zalo recurre a `allowFrom` para las comprobaciones de remitente.
-   `groupPolicy: "disabled"` bloquea todos los mensajes de grupo.
-   `groupPolicy: "open"` permite a cualquier miembro del grupo (activado por mención).
-   Nota de tiempo de ejecución: si `channels.zalo` falta por completo, el tiempo de ejecución aún recurre a `groupPolicy="allowlist"` por seguridad.

## Long-polling vs webhook

-   Por defecto: long-polling (no se requiere URL pública).
-   Modo webhook: establece `channels.zalo.webhookUrl` y `channels.zalo.webhookSecret`.
    -   El secreto del webhook debe tener entre 8 y 256 caracteres.
    -   La URL del webhook debe usar HTTPS.
    -   Zalo envía eventos con la cabecera `X-Bot-Api-Secret-Token` para verificación.
    -   El Gateway HTTP maneja las peticiones del webhook en `channels.zalo.webhookPath` (por defecto la ruta de la URL del webhook).
    -   Las peticiones deben usar `Content-Type: application/json` (o tipos de medios `+json`).
    -   Los eventos duplicados (`event_name + message_id`) se ignoran durante una breve ventana de repetición.
    -   El tráfico en ráfagas está limitado por tasa por ruta/fuente y puede devolver HTTP 429.

**Nota:** getUpdates (polling) y webhook son mutuamente excluyentes según la documentación de la API de Zalo.

## Tipos de mensajes soportados

-   **Mensajes de texto**: Soporte completo con división en fragmentos de 2000 caracteres.
-   **Mensajes de imagen**: Descarga y procesa imágenes entrantes; envía imágenes vía `sendPhoto`.
-   **Stickers**: Se registran pero no se procesan completamente (sin respuesta del agente).
-   **Tipos no soportados**: Se registran (ej., mensajes de usuarios protegidos).

## Capacidades

| Característica | Estado |
| --- | --- |
| Mensajes directos | ✅ Soportado |
| Grupos | ⚠️ Soportado con controles de política (lista de permitidos por defecto) |
| Medios (imágenes) | ✅ Soportado |
| Reacciones | ❌ No soportado |
| Hilos | ❌ No soportado |
| Encuestas | ❌ No soportado |
| Comandos nativos | ❌ No soportado |
| Streaming | ⚠️ Bloqueado (límite de 2000 caracteres) |

## Destinos de entrega (CLI/cron)

-   Usa un id de chat como destino.
-   Ejemplo: `openclaw message send --channel zalo --target 123456789 --message "hola"`.

## Solución de problemas

**El bot no responde:**

-   Comprueba que el token sea válido: `openclaw channels status --probe`
-   Verifica que el remitente esté aprobado (emparejamiento o allowFrom)
-   Revisa los registros del gateway: `openclaw logs --follow`

**El webhook no recibe eventos:**

-   Asegúrate de que la URL del webhook use HTTPS
-   Verifica que el token secreto tenga entre 8 y 256 caracteres
-   Confirma que el endpoint HTTP del gateway sea accesible en la ruta configurada
-   Comprueba que el polling getUpdates no se esté ejecutando (son mutuamente excluyentes)

## Referencia de configuración (Zalo)

Configuración completa: [Configuración](../gateway/configuration.md) Opciones del proveedor:

-   `channels.zalo.enabled`: activar/desactivar el inicio del canal.
-   `channels.zalo.botToken`: token del bot desde Zalo Bot Platform.
-   `channels.zalo.tokenFile`: leer token desde ruta de archivo.
-   `channels.zalo.dmPolicy`: `pairing | allowlist | open | disabled` (por defecto: pairing).
-   `channels.zalo.allowFrom`: lista de permitidos para DMs (IDs de usuario). `open` requiere `"*"`. El asistente pedirá IDs numéricos.
-   `channels.zalo.groupPolicy`: `open | allowlist | disabled` (por defecto: allowlist).
-   `channels.zalo.groupAllowFrom`: lista de permitidos de remitentes de grupo (IDs de usuario). Recurre a `allowFrom` cuando no esté establecido.
-   `channels.zalo.mediaMaxMb`: límite de medios entrantes/salientes (MB, por defecto 5).
-   `channels.zalo.webhookUrl`: activar modo webhook (se requiere HTTPS).
-   `channels.zalo.webhookSecret`: secreto del webhook (8-256 caracteres).
-   `channels.zalo.webhookPath`: ruta del webhook en el servidor HTTP del gateway.
-   `channels.zalo.proxy`: URL del proxy para peticiones de la API.

Opciones multi-cuenta:

-   `channels.zalo.accounts..botToken`: token por cuenta.
-   `channels.zalo.accounts..tokenFile`: archivo de token por cuenta.
-   `channels.zalo.accounts..name`: nombre para mostrar.
-   `channels.zalo.accounts..enabled`: activar/desactivar cuenta.
-   `channels.zalo.accounts..dmPolicy`: política de DM por cuenta.
-   `channels.zalo.accounts..allowFrom`: lista de permitidos por cuenta.
-   `channels.zalo.accounts..groupPolicy`: política de grupo por cuenta.
-   `channels.zalo.accounts..groupAllowFrom`: lista de permitidos de remitentes de grupo por cuenta.
-   `channels.zalo.accounts..webhookUrl`: URL de webhook por cuenta.
-   `channels.zalo.accounts..webhookSecret`: secreto de webhook por cuenta.
-   `channels.zalo.accounts..webhookPath`: ruta de webhook por cuenta.
-   `channels.zalo.accounts..proxy`: URL de proxy por cuenta.

[WhatsApp](./whatsapp.md)[Zalo Personal](./zalouser.md)

---