

  Plataformas de mensajería

  
# Nextcloud Talk

Estado: compatible mediante plugin (bot de webhook). Se admiten mensajes directos, salas, reacciones y mensajes en markdown.

## Plugin requerido

Nextcloud Talk se distribuye como un plugin y no está incluido en la instalación principal. Instálalo mediante CLI (registro de npm):

```bash
openclaw plugins install @openclaw/nextcloud-talk
```

Desde un repositorio local (cuando se ejecuta desde un repositorio git):

```bash
openclaw plugins install ./extensions/nextcloud-talk
```

Si eliges Nextcloud Talk durante la configuración/onboarding y se detecta un repositorio git, OpenClaw ofrecerá la ruta de instalación local automáticamente. Detalles: [Plugins](../tools/plugin.md)

## Configuración rápida (principiante)

1.  Instala el plugin de Nextcloud Talk.
2.  En tu servidor Nextcloud, crea un bot:
    
    Copiar
    
    ```bash
    ./occ talk:bot:install "OpenClaw" "<shared-secret>" "<webhook-url>" --feature reaction
    ```
    
3.  Habilita el bot en la configuración de la sala objetivo.
4.  Configura OpenClaw:
    -   Config: `channels.nextcloud-talk.baseUrl` + `channels.nextcloud-talk.botSecret`
    -   O env: `NEXTCLOUD_TALK_BOT_SECRET` (solo cuenta por defecto)
5.  Reinicia el gateway (o finaliza el onboarding).

Configuración mínima:

```json
{
  channels: {
    "nextcloud-talk": {
      enabled: true,
      baseUrl: "https://cloud.example.com",
      botSecret: "shared-secret",
      dmPolicy: "pairing",
    },
  },
}
```

## Notas

-   Los bots no pueden iniciar MDs. El usuario debe enviar un mensaje al bot primero.
-   La URL del webhook debe ser accesible por el Gateway; configura `webhookPublicUrl` si está detrás de un proxy.
-   Las subidas de medios no son compatibles con la API del bot; los medios se envían como URLs.
-   La carga útil del webhook no distingue entre MDs y salas; configura `apiUser` + `apiPassword` para habilitar búsquedas de tipo de sala (de lo contrario, los MDs se tratan como salas).

## Control de acceso (MDs)

-   Por defecto: `channels.nextcloud-talk.dmPolicy = "pairing"`. Los remitentes desconocidos reciben un código de emparejamiento.
-   Aprobar mediante:
    -   `openclaw pairing list nextcloud-talk`
    -   `openclaw pairing approve nextcloud-talk `
-   MDs públicos: `channels.nextcloud-talk.dmPolicy="open"` más `channels.nextcloud-talk.allowFrom=["*"]`.
-   `allowFrom` coincide solo con IDs de usuario de Nextcloud; se ignoran los nombres para mostrar.

## Salas (grupos)

-   Por defecto: `channels.nextcloud-talk.groupPolicy = "allowlist"` (acceso por mención).
-   Permite salas con `channels.nextcloud-talk.rooms`:

```json
{
  channels: {
    "nextcloud-talk": {
      rooms: {
        "room-token": { requireMention: true },
      },
    },
  },
}
```

-   Para no permitir ninguna sala, mantén la lista de permitidos vacía o configura `channels.nextcloud-talk.groupPolicy="disabled"`.

## Capacidades

| Característica | Estado |
| --- | --- |
| Mensajes directos | Compatible |
| Salas | Compatible |
| Hilos | No compatible |
| Medios | Solo URL |
| Reacciones | Compatible |
| Comandos nativos | No compatible |

## Referencia de configuración (Nextcloud Talk)

Configuración completa: [Configuración](../gateway/configuration.md) Opciones del proveedor:

-   `channels.nextcloud-talk.enabled`: habilitar/deshabilitar el inicio del canal.
-   `channels.nextcloud-talk.baseUrl`: URL de la instancia de Nextcloud.
-   `channels.nextcloud-talk.botSecret`: secreto compartido del bot.
-   `channels.nextcloud-talk.botSecretFile`: ruta del archivo de secreto.
-   `channels.nextcloud-talk.apiUser`: usuario de la API para búsquedas de salas (detección de MDs).
-   `channels.nextcloud-talk.apiPassword`: contraseña de API/aplicación para búsquedas de salas.
-   `channels.nextcloud-talk.apiPasswordFile`: ruta del archivo de contraseña de API.
-   `channels.nextcloud-talk.webhookPort`: puerto del listener del webhook (por defecto: 8788).
-   `channels.nextcloud-talk.webhookHost`: host del webhook (por defecto: 0.0.0.0).
-   `channels.nextcloud-talk.webhookPath`: ruta del webhook (por defecto: /nextcloud-talk-webhook).
-   `channels.nextcloud-talk.webhookPublicUrl`: URL del webhook accesible externamente.
-   `channels.nextcloud-talk.dmPolicy`: `pairing | allowlist | open | disabled`.
-   `channels.nextcloud-talk.allowFrom`: lista de permitidos para MDs (IDs de usuario). `open` requiere `"*"`.
-   `channels.nextcloud-talk.groupPolicy`: `allowlist | open | disabled`.
-   `channels.nextcloud-talk.groupAllowFrom`: lista de permitidos para grupos (IDs de usuario).
-   `channels.nextcloud-talk.rooms`: configuración y lista de permitidos por sala.
-   `channels.nextcloud-talk.historyLimit`: límite de historial de grupo (0 lo deshabilita).
-   `channels.nextcloud-talk.dmHistoryLimit`: límite de historial de MDs (0 lo deshabilita).
-   `channels.nextcloud-talk.dms`: anulaciones por MD (historyLimit).
-   `channels.nextcloud-talk.textChunkLimit`: tamaño de fragmento de texto saliente (caracteres).
-   `channels.nextcloud-talk.chunkMode`: `length` (por defecto) o `newline` para dividir en líneas en blanco (límites de párrafo) antes de la fragmentación por longitud.
-   `channels.nextcloud-talk.blockStreaming`: deshabilita el streaming de bloques para este canal.
-   `channels.nextcloud-talk.blockStreamingCoalesce`: ajuste de coalescencia del streaming de bloques.
-   `channels.nextcloud-talk.mediaMaxMb`: límite de medios entrantes (MB).

[Microsoft Teams](./msteams.md)[Nostr](./nostr.md)