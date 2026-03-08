

  Visión General

  
# Canales de Chat

OpenClaw puede hablar contigo en cualquier aplicación de chat que ya uses. Cada canal se conecta a través del Gateway. El texto es compatible en todas partes; los medios y las reacciones varían según el canal.

## Canales soportados

-   [BlueBubbles](./channels/bluebubbles.md) — **Recomendado para iMessage**; utiliza la API REST del servidor BlueBubbles para macOS con soporte completo de funciones (editar, eliminar envío, efectos, reacciones, gestión de grupos — la edición actualmente está rota en macOS 26 Tahoe).
-   [Discord](./channels/discord.md) — API de Bot de Discord + Gateway; soporta servidores, canales y MDs.
-   [Feishu](./channels/feishu.md) — Bot de Feishu/Lark vía WebSocket (plugin, instalado por separado).
-   [Google Chat](./channels/googlechat.md) — Aplicación de la API de Google Chat vía webhook HTTP.
-   [iMessage (heredado)](./channels/imessage.md) — Integración heredada de macOS vía CLI imsg (obsoleto, usa BlueBubbles para nuevas configuraciones).
-   [IRC](./channels/irc.md) — Servidores IRC clásicos; canales + MDs con controles de emparejamiento/lista de permitidos.
-   [LINE](./channels/line.md) — Bot de la API de Mensajería de LINE (plugin, instalado por separado).
-   [Matrix](./channels/matrix.md) — Protocolo Matrix (plugin, instalado por separado).
-   [Mattermost](./channels/mattermost.md) — API de Bot + WebSocket; canales, grupos, MDs (plugin, instalado por separado).
-   [Microsoft Teams](./channels/msteams.md) — Bot Framework; soporte empresarial (plugin, instalado por separado).
-   [Nextcloud Talk](./channels/nextcloud-talk.md) — Chat autoalojado vía Nextcloud Talk (plugin, instalado por separado).
-   [Nostr](./channels/nostr.md) — MDs descentralizados vía NIP-04 (plugin, instalado por separado).
-   [Signal](./channels/signal.md) — signal-cli; centrado en la privacidad.
-   [Synology Chat](./channels/synology-chat.md) — Chat de Synology NAS vía webhooks salientes+entrantes (plugin, instalado por separado).
-   [Slack](./channels/slack.md) — SDK Bolt; aplicaciones de espacio de trabajo.
-   [Telegram](./channels/telegram.md) — API de Bot vía grammY; soporta grupos.
-   [Tlon](./channels/tlon.md) — Mensajero basado en Urbit (plugin, instalado por separado).
-   [Twitch](./channels/twitch.md) — Chat de Twitch vía conexión IRC (plugin, instalado por separado).
-   [WebChat](./web/webchat.md) — Interfaz de usuario WebChat del Gateway sobre WebSocket.
-   [WhatsApp](./channels/whatsapp.md) — El más popular; utiliza Baileys y requiere emparejamiento por QR.
-   [Zalo](./channels/zalo.md) — API de Bot de Zalo; mensajero popular de Vietnam (plugin, instalado por separado).
-   [Zalo Personal](./channels/zalouser.md) — Cuenta personal de Zalo vía inicio de sesión por QR (plugin, instalado por separado).

## Notas

-   Los canales pueden ejecutarse simultáneamente; configura varios y OpenClaw enrutará por chat.
-   La configuración más rápida suele ser **Telegram** (token de bot simple). WhatsApp requiere emparejamiento por QR y almacena más estado en el disco.
-   El comportamiento en grupos varía según el canal; consulta [Grupos](./channels/groups.md).
-   El emparejamiento en MDs y las listas de permitidos se aplican por seguridad; consulta [Seguridad](./gateway/security.md).
-   Solución de problemas: [Solución de problemas de canales](./channels/troubleshooting.md).
-   Los proveedores de modelos están documentados por separado; consulta [Proveedores de Modelos](./providers/models.md).

[BlueBubbles](./channels/bluebubbles.md)

---