

  Configuración y operaciones

  
# Comprobaciones de Salud

Guía breve para verificar la conectividad de canales sin adivinar.

## Comprobaciones rápidas

-   `openclaw status` — resumen local: alcance/modo del gateway, sugerencia de actualización, antigüedad de la autenticación del canal vinculado, sesiones + actividad reciente.
-   `openclaw status --all` — diagnóstico local completo (solo lectura, color, seguro para pegar para depuración).
-   `openclaw status --deep` — también sondea el Gateway en ejecución (sondas por canal cuando son compatibles).
-   `openclaw health --json` — solicita al Gateway en ejecución una instantánea completa de salud (solo WS; sin socket directo de Baileys).
-   Envía `/status` como un mensaje independiente en WhatsApp/WebChat para obtener una respuesta de estado sin invocar al agente.
-   Registros: sigue la cola de `/tmp/openclaw/openclaw-*.log` y filtra por `web-heartbeat`, `web-reconnect`, `web-auto-reply`, `web-inbound`.

## Diagnósticos profundos

-   Credenciales en disco: `ls -l ~/.openclaw/credentials/whatsapp//creds.json` (mtime debe ser reciente).
-   Almacén de sesiones: `ls -l ~/.openclaw/agents//sessions/sessions.json` (la ruta se puede anular en la configuración). El recuento y los destinatarios recientes se muestran a través de `status`.
-   Flujo de reenlace: `openclaw channels logout && openclaw channels login --verbose` cuando aparecen códigos de estado 409–515 o `loggedOut` en los registros. (Nota: el flujo de inicio de sesión QR se reinicia automáticamente una vez para el estado 515 después del emparejamiento.)

## Cuando algo falla

-   `logged out` o estado 409–515 → reenlazar con `openclaw channels logout` y luego `openclaw channels login`.
-   Gateway inalcanzable → inícialo: `openclaw gateway --port 18789` (usa `--force` si el puerto está ocupado).
-   Sin mensajes entrantes → confirma que el teléfono vinculado esté en línea y que el remitente esté permitido (`channels.whatsapp.allowFrom`); para chats grupales, asegúrate de que coincidan la lista de permitidos + las reglas de mención (`channels.whatsapp.groups`, `agents.list[].groupChat.mentionPatterns`).

## Comando dedicado "health"

`openclaw health --json` solicita al Gateway en ejecución su instantánea de salud (sin sockets de canal directos desde la CLI). Informa la antigüedad de las credenciales/autenticación vinculadas cuando están disponibles, resúmenes de sonda por canal, resumen del almacén de sesiones y una duración de la sonda. Sale con un código distinto de cero si el Gateway es inalcanzable o si la sonda falla/se agota el tiempo. Usa `--timeout ` para anular el valor predeterminado de 10s.

[Autenticación de proxy confiable](./trusted-proxy-auth.md)[Latido del corazón](./heartbeat.md)

---