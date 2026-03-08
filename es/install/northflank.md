

  Alojamiento y despliegue

  
# Desplegar en Northflank

Despliega OpenClaw en Northflank con una plantilla de un clic y termina la configuración en tu navegador. Esta es la ruta más fácil "sin terminal en el servidor": Northflank ejecuta la Puerta de Enlace por ti y tú configuras todo mediante el asistente web `/setup`.

## Cómo empezar

1.  Haz clic en [Desplegar OpenClaw](https://northflank.com/stacks/deploy-openclaw) para abrir la plantilla.
2.  Crea una [cuenta en Northflank](https://app.northflank.com/signup) si aún no tienes una.
3.  Haz clic en **Desplegar OpenClaw ahora**.
4.  Establece la variable de entorno requerida: `SETUP_PASSWORD`.
5.  Haz clic en **Desplegar stack** para construir y ejecutar la plantilla de OpenClaw.
6.  Espera a que se complete el despliegue, luego haz clic en **Ver recursos**.
7.  Abre el servicio OpenClaw.
8.  Abre la URL pública de OpenClaw y completa la configuración en `/setup`.
9.  Abre la Interfaz de Control en `/openclaw`.

## Lo que obtienes

-   Puerta de Enlace OpenClaw + Interfaz de Control alojadas
-   Asistente de configuración web en `/setup` (sin comandos de terminal)
-   Almacenamiento persistente mediante Volumen de Northflank (`/data`) para que la configuración/credenciales/espacio de trabajo sobrevivan a redespliegues

## Flujo de configuración

1.  Visita `https://<tu-dominio-northflank>/setup` e introduce tu `SETUP_PASSWORD`.
2.  Elige un modelo/proveedor de autenticación y pega tu clave.
3.  (Opcional) Añade tokens de Telegram/Discord/Slack.
4.  Haz clic en **Ejecutar configuración**.
5.  Abre la Interfaz de Control en `https://<tu-dominio-northflank>/openclaw`

Si los mensajes directos de Telegram están configurados para emparejamiento, el asistente de configuración puede aprobar el código de emparejamiento.

## Obteniendo tokens de chat

### Token de bot de Telegram

1.  Envía un mensaje a `@BotFather` en Telegram
2.  Ejecuta `/newbot`
3.  Copia el token (parecido a `123456789:AA...`)
4.  Pégalo en `/setup`

### Token de bot de Discord

1.  Ve a [https://discord.com/developers/applications](https://discord.com/developers/applications)
2.  **Nueva Aplicación** → elige un nombre
3.  **Bot** → **Añadir Bot**
4.  **Habilitar MESSAGE CONTENT INTENT** en Bot → Privileged Gateway Intents (requerido o el bot fallará al iniciar)
5.  Copia el **Token del Bot** y pégala en `/setup`
6.  Invita el bot a tu servidor (Generador de URL OAuth2; alcances: `bot`, `applications.commands`)

[Desplegar en Render](./render.md)[Canales de Desarrollo](./development-channels.md)

---