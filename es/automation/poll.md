

  Automatización

  
# Encuestas

## Canales compatibles

-   Telegram
-   WhatsApp (canal web)
-   Discord
-   MS Teams (Adaptive Cards)

## CLI

```bash
# Telegram
openclaw message poll --channel telegram --target 123456789 \
  --poll-question "¿Lo enviamos?" --poll-option "Sí" --poll-option "No"
openclaw message poll --channel telegram --target -1001234567890:topic:42 \
  --poll-question "Elige una hora" --poll-option "10am" --poll-option "2pm" \
  --poll-duration-seconds 300

# WhatsApp
openclaw message poll --target +15555550123 \
  --poll-question "¿Almuerzo hoy?" --poll-option "Sí" --poll-option "No" --poll-option "Tal vez"
openclaw message poll --target 123456789@g.us \
  --poll-question "¿Hora de la reunión?" --poll-option "10am" --poll-option "2pm" --poll-option "4pm" --poll-multi

# Discord
openclaw message poll --channel discord --target channel:123456789 \
  --poll-question "¿Snack?" --poll-option "Pizza" --poll-option "Sushi"
openclaw message poll --channel discord --target channel:123456789 \
  --poll-question "¿Plan?" --poll-option "A" --poll-option "B" --poll-duration-hours 48

# MS Teams
openclaw message poll --channel msteams --target conversation:19:abc@thread.tacv2 \
  --poll-question "¿Almuerzo?" --poll-option "Pizza" --poll-option "Sushi"
```

Opciones:

-   `--channel`: `whatsapp` (predeterminado), `telegram`, `discord`, o `msteams`
-   `--poll-multi`: permitir seleccionar múltiples opciones
-   `--poll-duration-hours`: Solo Discord (por defecto 24 si se omite)
-   `--poll-duration-seconds`: Solo Telegram (5-600 segundos)
-   `--poll-anonymous` / `--poll-public`: Visibilidad de la encuesta, solo Telegram

## Gateway RPC

Método: `poll` Parámetros:

-   `to` (string, requerido)
-   `question` (string, requerido)
-   `options` (string\[\], requerido)
-   `maxSelections` (number, opcional)
-   `durationHours` (number, opcional)
-   `durationSeconds` (number, opcional, solo Telegram)
-   `isAnonymous` (boolean, opcional, solo Telegram)
-   `channel` (string, opcional, predeterminado: `whatsapp`)
-   `idempotencyKey` (string, requerido)

## Diferencias entre canales

-   Telegram: 2-10 opciones. Soporta temas de foro mediante `threadId` o destinos `:topic:`. Usa `durationSeconds` en lugar de `durationHours`, limitado a 5-600 segundos. Soporta encuestas anónimas y públicas.
-   WhatsApp: 2-12 opciones, `maxSelections` debe estar dentro del conteo de opciones, ignora `durationHours`.
-   Discord: 2-10 opciones, `durationHours` limitado a 1-768 horas (predeterminado 24). `maxSelections > 1` habilita selección múltiple; Discord no soporta un conteo de selección estricto.
-   MS Teams: Encuestas con Adaptive Card (gestionadas por OpenClaw). No tiene API nativa de encuestas; `durationHours` es ignorado.

## Herramienta de Agente (Message)

Usa la herramienta `message` con la acción `poll` (`to`, `pollQuestion`, `pollOption`, opcionalmente `pollMulti`, `pollDurationHours`, `channel`). Para Telegram, la herramienta también acepta `pollDurationSeconds`, `pollAnonymous`, y `pollPublic`. Usa `action: "poll"` para la creación de encuestas. Los campos de encuesta pasados con `action: "send"` son rechazados. Nota: Discord no tiene modo "elegir exactamente N"; `pollMulti` se mapea a selección múltiple. Las encuestas de Teams se renderizan como Adaptive Cards y requieren que el gateway permanezca en línea para registrar votos en `~/.openclaw/msteams-polls.json`.

[Gmail PubSub](./gmail-pubsub.md)[Auth Monitoring](./auth-monitoring.md)

---