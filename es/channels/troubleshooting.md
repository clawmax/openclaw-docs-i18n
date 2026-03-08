

  Configuración

  
# Solución de Problemas de Canales

Utilice esta página cuando un canal se conecta pero su comportamiento es incorrecto.

## Escalera de comandos

Ejecute estos en orden primero:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Línea base saludable:

-   `Runtime: running`
-   `RPC probe: ok`
-   El sondeo del canal muestra conectado/listo

## WhatsApp

### Firmas de fallo de WhatsApp

| Síntoma | Comprobación más rápida | Solución |
| --- | --- | --- |
| Conectado pero sin respuestas en DM | `openclaw pairing list whatsapp` | Apruebe el remitente o cambie la política de DM/lista de permitidos. |
| Mensajes de grupo ignorados | Verifique `requireMention` + patrones de mención en la configuración | Mencione al bot o relaje la política de mención para ese grupo. |
| Bucles de desconexión/reinicio de sesión aleatorios | `openclaw channels status --probe` + registros | Vuelva a iniciar sesión y verifique que el directorio de credenciales esté saludable. |

Solución de problemas completa: [/channels/whatsapp#troubleshooting-quick](./whatsapp.md#troubleshooting-quick)

## Telegram

### Firmas de fallo de Telegram

| Síntoma | Comprobación más rápida | Solución |
| --- | --- | --- |
| `/start` pero sin flujo de respuesta utilizable | `openclaw pairing list telegram` | Apruebe el emparejamiento o cambie la política de DM. |
| Bot en línea pero el grupo permanece en silencio | Verifique el requisito de mención y el modo de privacidad del bot | Desactive el modo de privacidad para la visibilidad del grupo o mencione al bot. |
| Fallos de envío con errores de red | Inspeccione los registros en busca de fallos en llamadas a la API de Telegram | Solucione el enrutamiento DNS/IPv6/proxy a `api.telegram.org`. |
| Actualizado y la lista de permitidos le bloquea | `openclaw security audit` y listas de permitidos de configuración | Ejecute `openclaw doctor --fix` o reemplace `@username` con IDs de remitente numéricos. |

Solución de problemas completa: [/channels/telegram#troubleshooting](./telegram.md#troubleshooting)

## Discord

### Firmas de fallo de Discord

| Síntoma | Comprobación más rápida | Solución |
| --- | --- | --- |
| Bot en línea pero sin respuestas en el gremio | `openclaw channels status --probe` | Permita el gremio/canal y verifique el intento de contenido de mensaje. |
| Mensajes de grupo ignorados | Verifique los registros para ver descartes por mención | Mencione al bot o establezca `requireMention: false` para el gremio/canal. |
| Faltan respuestas en DM | `openclaw pairing list discord` | Apruebe el emparejamiento de DM o ajuste la política de DM. |

Solución de problemas completa: [/channels/discord#troubleshooting](./discord.md#troubleshooting)

## Slack

### Firmas de fallo de Slack

| Síntoma | Comprobación más rápida | Solución |
| --- | --- | --- |
| Modo socket conectado pero sin respuestas | `openclaw channels status --probe` | Verifique el token de aplicación + token de bot y los alcances requeridos. |
| DMs bloqueados | `openclaw pairing list slack` | Apruebe el emparejamiento o relaje la política de DM. |
| Mensaje de canal ignorado | Verifique `groupPolicy` y la lista de permitidos del canal | Permita el canal o cambie la política a `open`. |

Solución de problemas completa: [/channels/slack#troubleshooting](./slack.md#troubleshooting)

## iMessage y BlueBubbles

### Firmas de fallo de iMessage y BlueBubbles

| Síntoma | Comprobación más rápida | Solución |
| --- | --- | --- |
| Sin eventos entrantes | Verifique la accesibilidad del webhook/servidor y los permisos de la aplicación | Solucione la URL del webhook o el estado del servidor de BlueBubbles. |
| Puede enviar pero no recibir en macOS | Verifique los permisos de privacidad de macOS para la automatización de Mensajes | Vuelva a otorgar permisos TCC y reinicie el proceso del canal. |
| Remitente de DM bloqueado | `openclaw pairing list imessage` o `openclaw pairing list bluebubbles` | Apruebe el emparejamiento o actualice la lista de permitidos. |

Solución de problemas completa:

-   [/channels/imessage#troubleshooting-macos-privacy-and-security-tcc](./imessage.md#troubleshooting-macos-privacy-and-security-tcc)
-   [/channels/bluebubbles#troubleshooting](./bluebubbles.md#troubleshooting)

## Signal

### Firmas de fallo de Signal

| Síntoma | Comprobación más rápida | Solución |
| --- | --- | --- |
| Demonio accesible pero el bot en silencio | `openclaw channels status --probe` | Verifique la URL/cuenta del demonio `signal-cli` y el modo de recepción. |
| DM bloqueado | `openclaw pairing list signal` | Apruebe el remitente o ajuste la política de DM. |
| Las respuestas de grupo no se activan | Verifique la lista de permitidos del grupo y los patrones de mención | Agregue remitente/grupo o relaje la restricción. |

Solución de problemas completa: [/channels/signal#troubleshooting](./signal.md#troubleshooting)

## Matrix

### Firmas de fallo de Matrix

| Síntoma | Comprobación más rápida | Solución |
| --- | --- | --- |
| Inició sesión pero ignora mensajes de sala | `openclaw channels status --probe` | Verifique `groupPolicy` y la lista de permitidos de la sala. |
| Los DMs no se procesan | `openclaw pairing list matrix` | Apruebe el remitente o ajuste la política de DM. |
| Fallan las salas encriptadas | Verifique el módulo de cifrado y la configuración de encriptación | Habilite el soporte de encriptación y vuelva a unirse/sincronizar la sala. |

Solución de problemas completa: [/channels/matrix#troubleshooting](./matrix.md#troubleshooting)

[Análisis de Ubicación de Canales](./location.md)

---