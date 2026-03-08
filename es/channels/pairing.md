

  Configuración

  
# Emparejamiento

El "Emparejamiento" es el paso de **aprobación explícita del propietario** en OpenClaw. Se utiliza en dos situaciones:

1.  **Emparejamiento de DM** (quién tiene permiso para hablar con el bot)
2.  **Emparejamiento de Nodo** (qué dispositivos/nodos tienen permiso para unirse a la red del gateway)

Contexto de seguridad: [Seguridad](../gateway/security.md)

## 1) Emparejamiento de DM (acceso a chat entrante)

Cuando un canal se configura con la política de DM `pairing`, los remitentes desconocidos reciben un código corto y su mensaje **no se procesa** hasta que tú lo apruebes. Las políticas de DM por defecto están documentadas en: [Seguridad](../gateway/security.md) Códigos de emparejamiento:

-   8 caracteres, mayúsculas, sin caracteres ambiguos (`0O1I`).
-   **Caducan después de 1 hora**. El bot solo envía el mensaje de emparejamiento cuando se crea una nueva solicitud (aproximadamente una vez por hora por remitente).
-   Las solicitudes de emparejamiento de DM pendientes están limitadas a **3 por canal** por defecto; las solicitudes adicionales se ignoran hasta que una caduque o sea aprobada.

### Aprobar un remitente

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

Canales soportados: `telegram`, `whatsapp`, `signal`, `imessage`, `discord`, `slack`, `feishu`.

### Dónde se almacena el estado

Almacenado en `~/.openclaw/credentials/`:

-   Solicitudes pendientes: `-pairing.json`
-   Almacén de lista de permitidos aprobados:
    -   Cuenta por defecto: `-allowFrom.json`
    -   Cuenta no por defecto: `--allowFrom.json`

Comportamiento de alcance de cuenta:

-   Las cuentas no por defecto leen/escriben solo su archivo de lista de permitidos con alcance.
-   La cuenta por defecto usa el archivo de lista de permitidos sin alcance específico del canal.

Trata estos archivos como sensibles (controlan el acceso a tu asistente).

## 2) Emparejamiento de dispositivos nodo (iOS/Android/macOS/nodos headless)

Los nodos se conectan al Gateway como **dispositivos** con `role: node`. El Gateway crea una solicitud de emparejamiento de dispositivo que debe ser aprobada.

### Emparejar vía Telegram (recomendado para iOS)

Si usas el plugin `device-pair`, puedes hacer el emparejamiento inicial del dispositivo completamente desde Telegram:

1.  En Telegram, envía un mensaje a tu bot: `/pair`
2.  El bot responde con dos mensajes: un mensaje de instrucciones y un mensaje separado con el **código de configuración** (fácil de copiar/pegar en Telegram).
3.  En tu teléfono, abre la app de OpenClaw para iOS → Configuración → Gateway.
4.  Pega el código de configuración y conéctate.
5.  De vuelta en Telegram: `/pair approve`

El código de configuración es una carga útil JSON codificada en base64 que contiene:

-   `url`: la URL WebSocket del Gateway (`ws://...` o `wss://...`)
-   `token`: un token de emparejamiento de corta duración

Trata el código de configuración como una contraseña mientras sea válido.

### Aprobar un dispositivo nodo

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
```

### Almacenamiento del estado de emparejamiento de nodo

Almacenado en `~/.openclaw/devices/`:

-   `pending.json` (de corta duración; las solicitudes pendientes caducan)
-   `paired.json` (dispositivos emparejados + tokens)

### Notas

-   La API heredada `node.pair.*` (CLI: `openclaw nodes pending/approve`) es un almacén de emparejamiento separado propiedad del gateway. Los nodos WS aún requieren emparejamiento de dispositivo.

## Documentación relacionada

-   Modelo de seguridad + inyección de prompt: [Seguridad](../gateway/security.md)
-   Actualizar de forma segura (ejecutar doctor): [Actualizando](../install/updating.md)
-   Configuraciones de canal:
    -   Telegram: [Telegram](./telegram.md)
    -   WhatsApp: [WhatsApp](./whatsapp.md)
    -   Signal: [Signal](./signal.md)
    -   BlueBubbles (iMessage): [BlueBubbles](./bluebubbles.md)
    -   iMessage (heredado): [iMessage](./imessage.md)
    -   Discord: [Discord](./discord.md)
    -   Slack: [Slack](./slack.md)

[Zalo Personal](./zalouser.md)[Mensajes de Grupo](./group-messages.md)