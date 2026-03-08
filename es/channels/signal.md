

  Plataformas de mensajería

  
# Signal

Estado: integración CLI externa. La puerta de enlace se comunica con `signal-cli` a través de HTTP JSON-RPC + SSE.

## Requisitos previos

-   OpenClaw instalado en tu servidor (el flujo de Linux a continuación probado en Ubuntu 24).
-   `signal-cli` disponible en el host donde se ejecuta la puerta de enlace.
-   Un número de teléfono que pueda recibir un SMS de verificación (para la ruta de registro por SMS).
-   Acceso a un navegador para el captcha de Signal (`signalcaptchas.org`) durante el registro.

## Configuración rápida (principiante)

1.  Usa un **número de Signal separado** para el bot (recomendado).
2.  Instala `signal-cli` (se requiere Java si usas la compilación JVM).
3.  Elige una ruta de configuración:
    -   **Ruta A (enlace QR):** `signal-cli link -n "OpenClaw"` y escanéalo con Signal.
    -   **Ruta B (registro por SMS):** registra un número dedicado con captcha + verificación por SMS.
4.  Configura OpenClaw y reinicia la puerta de enlace.
5.  Envía un primer MD y aprueba el emparejamiento (`openclaw pairing approve signal `).

Configuración mínima:

```json
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

Referencia de campos:

| Campo | Descripción |
| --- | --- |
| `account` | Número de teléfono del bot en formato E.164 (`+15551234567`) |
| `cliPath` | Ruta a `signal-cli` (`signal-cli` si está en `PATH`) |
| `dmPolicy` | Política de acceso a MD (`pairing` recomendado) |
| `allowFrom` | Números de teléfono o valores `uuid:` permitidos para enviar MD |

## Qué es

-   Canal Signal a través de `signal-cli` (no libsignal integrado).
-   Enrutamiento determinista: las respuestas siempre regresan a Signal.
-   Los MD comparten la sesión principal del agente; los grupos están aislados (`agent::signal:group:`).

## Escrituras de configuración

Por defecto, Signal tiene permitido escribir actualizaciones de configuración activadas por `/config set|unset` (requiere `commands.config: true`). Deshabilita con:

```json
{
  channels: { signal: { configWrites: false } },
}
```

## El modelo del número (importante)

-   La puerta de enlace se conecta a un **dispositivo Signal** (la cuenta `signal-cli`).
-   Si ejecutas el bot en **tu cuenta personal de Signal**, ignorará tus propios mensajes (protección contra bucles).
-   Para "Yo le envío un mensaje al bot y él responde", usa un **número de bot separado**.

## Ruta de configuración A: vincular cuenta de Signal existente (QR)

1.  Instala `signal-cli` (compilación JVM o nativa).
2.  Vincula una cuenta de bot:
    -   `signal-cli link -n "OpenClaw"` luego escanea el QR en Signal.
3.  Configura Signal e inicia la puerta de enlace.

Ejemplo:

```json
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

Soporte multi-cuenta: usa `channels.signal.accounts` con configuración por cuenta y `name` opcional. Consulta [`gateway/configuration`](../gateway/configuration.md#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) para el patrón compartido.

## Ruta de configuración B: registrar número de bot dedicado (SMS, Linux)

Usa esto cuando quieras un número de bot dedicado en lugar de vincular una cuenta existente de la aplicación Signal.

1.  Obtén un número que pueda recibir SMS (o verificación por voz para líneas fijas).
    -   Usa un número de bot dedicado para evitar conflictos de cuenta/sesión.
2.  Instala `signal-cli` en el host de la puerta de enlace:

```
VERSION=$(curl -Ls -o /dev/null -w %{url_effective} https://github.com/AsamK/signal-cli/releases/latest | sed -e 's/^.*\/v//')
curl -L -O "https://github.com/AsamK/signal-cli/releases/download/v${VERSION}/signal-cli-${VERSION}-Linux-native.tar.gz"
sudo tar xf "signal-cli-${VERSION}-Linux-native.tar.gz" -C /opt
sudo ln -sf /opt/signal-cli /usr/local/bin/
signal-cli --version
```

Si usas la compilación JVM (`signal-cli-${VERSION}.tar.gz`), instala JRE 25+ primero. Mantén `signal-cli` actualizado; las notas del proyecto indican que versiones antiguas pueden fallar a medida que cambian las APIs del servidor de Signal.

3.  Registra y verifica el número:

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register
```

Si se requiere captcha:

1.  Abre `https://signalcaptchas.org/registration/generate.html`.
2.  Completa el captcha, copia el destino del enlace `signalcaptcha://...` desde "Open Signal".
3.  Ejecuta desde la misma IP externa que la sesión del navegador cuando sea posible.
4.  Ejecuta el registro nuevamente inmediatamente (los tokens de captcha expiran rápidamente):

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register --captcha '<SIGNALCAPTCHA_URL>'
signal-cli -a +<BOT_PHONE_NUMBER> verify <VERIFICATION_CODE>
```

4.  Configura OpenClaw, reinicia la puerta de enlace, verifica el canal:

```bash
# Si ejecutas la puerta de enlace como un servicio systemd de usuario:
systemctl --user restart openclaw-gateway

# Luego verifica:
openclaw doctor
openclaw channels status --probe
```

5.  Empareja tu remitente de MD:
    -   Envía cualquier mensaje al número del bot.
    -   Aprueba el código en el servidor: `openclaw pairing approve signal <PAIRING_CODE>`.
    -   Guarda el número del bot como contacto en tu teléfono para evitar "Contacto desconocido".

Importante: registrar una cuenta de número de teléfono con `signal-cli` puede desautenticar la sesión principal de la aplicación Signal para ese número. Prefiere un número de bot dedicado, o usa el modo de enlace QR si necesitas mantener tu configuración existente de la aplicación en el teléfono. Referencias del proyecto:

-   README de `signal-cli`: `https://github.com/AsamK/signal-cli`
-   Flujo de captcha: `https://github.com/AsamK/signal-cli/wiki/Registration-with-captcha`
-   Flujo de vinculación: `https://github.com/AsamK/signal-cli/wiki/Linking-other-devices-(Provisioning)`

## Modo demonio externo (httpUrl)

Si quieres gestionar `signal-cli` tú mismo (inicios lentos en frío de JVM, inicialización de contenedores, o CPUs compartidas), ejecuta el demonio por separado y apunta OpenClaw a él:

```json
{
  channels: {
    signal: {
      httpUrl: "http://127.0.0.1:8080",
      autoStart: false,
    },
  },
}
```

Esto omite el auto-inicio y la espera de inicio dentro de OpenClaw. Para inicios lentos cuando se auto-inicia, configura `channels.signal.startupTimeoutMs`.

## Control de acceso (MD + grupos)

MD:

-   Por defecto: `channels.signal.dmPolicy = "pairing"`.
-   Los remitentes desconocidos reciben un código de emparejamiento; los mensajes se ignoran hasta que se aprueban (los códigos expiran después de 1 hora).
-   Aprueba mediante:
    -   `openclaw pairing list signal`
    -   `openclaw pairing approve signal `
-   El emparejamiento es el intercambio de tokens predeterminado para los MD de Signal. Detalles: [Emparejamiento](./pairing.md)
-   Los remitentes solo por UUID (de `sourceUuid`) se almacenan como `uuid:` en `channels.signal.allowFrom`.

Grupos:

-   `channels.signal.groupPolicy = open | allowlist | disabled`.
-   `channels.signal.groupAllowFrom` controla quién puede activar en grupos cuando `allowlist` está configurado.
-   Nota de tiempo de ejecución: si `channels.signal` falta completamente, el tiempo de ejecución vuelve a `groupPolicy="allowlist"` para las comprobaciones de grupo (incluso si `channels.defaults.groupPolicy` está configurado).

## Cómo funciona (comportamiento)

-   `signal-cli` se ejecuta como un demonio; la puerta de enlace lee eventos a través de SSE.
-   Los mensajes entrantes se normalizan en el sobre de canal compartido.
-   Las respuestas siempre se enrutan de vuelta al mismo número o grupo.

## Medios + límites

-   El texto saliente se divide en fragmentos de `channels.signal.textChunkLimit` (por defecto 4000).
-   División opcional por saltos de línea: configura `channels.signal.chunkMode="newline"` para dividir en líneas en blanco (límites de párrafo) antes de la división por longitud.
-   Archivos adjuntos admitidos (base64 obtenido de `signal-cli`).
-   Límite de medios por defecto: `channels.signal.mediaMaxMb` (por defecto 8).
-   Usa `channels.signal.ignoreAttachments` para omitir la descarga de medios.
-   El contexto del historial de grupos usa `channels.signal.historyLimit` (o `channels.signal.accounts.*.historyLimit`), volviendo a `messages.groupChat.historyLimit`. Configura `0` para deshabilitar (por defecto 50).

## Escribiendo + recibos de lectura

-   **Indicadores de escritura**: OpenClaw envía señales de escritura a través de `signal-cli sendTyping` y las actualiza mientras se ejecuta una respuesta.
-   **Recibos de lectura**: cuando `channels.signal.sendReadReceipts` es verdadero, OpenClaw reenvía recibos de lectura para MD permitidos.
-   Signal-cli no expone recibos de lectura para grupos.

## Reacciones (herramienta de mensaje)

-   Usa `message action=react` con `channel=signal`.
-   Destinos: E.164 del remitente o UUID (usa `uuid:` de la salida de emparejamiento; el UUID solo también funciona).
-   `messageId` es la marca de tiempo de Signal para el mensaje al que estás reaccionando.
-   Las reacciones en grupo requieren `targetAuthor` o `targetAuthorUuid`.

Ejemplos:

```bash
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅
```

Configuración:

-   `channels.signal.actions.reactions`: habilita/deshabilita acciones de reacción (por defecto verdadero).
-   `channels.signal.reactionLevel`: `off | ack | minimal | extensive`.
    -   `off`/`ack` deshabilita las reacciones del agente (la herramienta de mensaje `react` dará error).
    -   `minimal`/`extensive` habilita las reacciones del agente y establece el nivel de guía.
-   Anulaciones por cuenta: `channels.signal.accounts..actions.reactions`, `channels.signal.accounts..reactionLevel`.

## Destinos de entrega (CLI/cron)

-   MD: `signal:+15551234567` (o E.164 simple).
-   MD por UUID: `uuid:` (o UUID solo).
-   Grupos: `signal:group:`.
-   Nombres de usuario: `username:` (si es compatible con tu cuenta de Signal).

## Solución de problemas

Ejecuta esta escalera primero:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Luego confirma el estado de emparejamiento de MD si es necesario:

```bash
openclaw pairing list signal
```

Fallos comunes:

-   Demonio accesible pero sin respuestas: verifica la configuración de cuenta/demonio (`httpUrl`, `account`) y el modo de recepción.
-   MD ignorados: el remitente está pendiente de aprobación de emparejamiento.
-   Mensajes de grupo ignorados: el control de acceso de remitente/mención en grupo bloquea la entrega.
-   Errores de validación de configuración después de ediciones: ejecuta `openclaw doctor --fix`.
-   Signal falta en los diagnósticos: confirma `channels.signal.enabled: true`.

Comprobaciones adicionales:

```bash
openclaw pairing list signal
pgrep -af signal-cli
grep -i "signal" "/tmp/openclaw/openclaw-$(date +%Y-%m-%d).log" | tail -20
```

Para flujo de triaje: [/channels/troubleshooting](./troubleshooting.md).

## Notas de seguridad

-   `signal-cli` almacena las claves de la cuenta localmente (típicamente `~/.local/share/signal-cli/data/`).
-   Haz una copia de seguridad del estado de la cuenta de Signal antes de migrar o reconstruir el servidor.
-   Mantén `channels.signal.dmPolicy: "pairing"` a menos que quieras explícitamente un acceso más amplio a MD.
-   La verificación por SMS solo es necesaria para flujos de registro o recuperación, pero perder el control del número/cuenta puede complicar el re-registro.

## Referencia de configuración (Signal)

Configuración completa: [Configuración](../gateway/configuration.md) Opciones del proveedor:

-   `channels.signal.enabled`: habilita/deshabilita el inicio del canal.
-   `channels.signal.account`: E.164 para la cuenta del bot.
-   `channels.signal.cliPath`: ruta a `signal-cli`.
-   `channels.signal.httpUrl`: URL completa del demonio (anula host/puerto).
-   `channels.signal.httpHost`, `channels.signal.httpPort`: enlace del demonio (por defecto 127.0.0.1:8080).
-   `channels.signal.autoStart`: auto-iniciar demonio (por defecto verdadero si `httpUrl` no está configurado).
-   `channels.signal.startupTimeoutMs`: tiempo de espera de inicio en ms (límite 120000).
-   `channels.signal.receiveMode`: `on-start | manual`.
-   `channels.signal.ignoreAttachments`: omitir descargas de archivos adjuntos.
-   `channels.signal.ignoreStories`: ignorar historias del demonio.
-   `channels.signal.sendReadReceipts`: reenviar recibos de lectura.
-   `channels.signal.dmPolicy`: `pairing | allowlist | open | disabled` (por defecto: pairing).
-   `channels.signal.allowFrom`: lista de permitidos para MD (E.164 o `uuid:`). `open` requiere `"*"`. Signal no tiene nombres de usuario; usa identificadores de teléfono/UUID.
-   `channels.signal.groupPolicy`: `open | allowlist | disabled` (por defecto: allowlist).
-   `channels.signal.groupAllowFrom`: lista de permitidos de remitentes de grupo.
-   `channels.signal.historyLimit`: máximo de mensajes de grupo para incluir como contexto (0 deshabilita).
-   `channels.signal.dmHistoryLimit`: límite de historial de MD en turnos de usuario. Anulaciones por usuario: `channels.signal.dms["<phone_or_uuid>"].historyLimit`.
-   `channels.signal.textChunkLimit`: tamaño de fragmento saliente (caracteres).
-   `channels.signal.chunkMode`: `length` (por defecto) o `newline` para dividir en líneas en blanco (límites de párrafo) antes de la división por longitud.
-   `channels.signal.mediaMaxMb`: límite de medios entrantes/salientes (MB).

Opciones globales relacionadas:

-   `agents.list[].groupChat.mentionPatterns` (Signal no admite menciones nativas).
-   `messages.groupChat.mentionPatterns` (respaldo global).
-   `messages.responsePrefix`.

[Nostr](./nostr.md)[Synology Chat](./synology-chat.md)