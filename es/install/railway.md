

  Alojamiento y despliegue

  
# Desplegar en Railway

Despliega OpenClaw en Railway con una plantilla de un clic y termina la configuración en tu navegador. Esta es la ruta más fácil "sin terminal en el servidor": Railway ejecuta la Puerta de Enlace por ti y tú configuras todo a través del asistente web `/setup`.

## Lista rápida (usuarios nuevos)

1.  Haz clic en **Desplegar en Railway** (abajo).
2.  Añade un **Volumen** montado en `/data`.
3.  Establece las **Variables** requeridas (al menos `SETUP_PASSWORD`).
4.  Habilita el **Proxy HTTP** en el puerto `8080`.
5.  Abre `https://<tu-dominio-railway>/setup` y termina el asistente.

## Despliegue de un clic

[Desplegar en Railway](https://railway.com/deploy/clawdbot-railway-template) Después del despliegue, encuentra tu URL pública en **Railway → tu servicio → Configuración → Dominios**. Railway te dará:

-   un dominio generado (a menudo `https://.up.railway.app`), o
-   usará tu dominio personalizado si has adjuntado uno.

Luego abre:

-   `https://<tu-dominio-railway>/setup` — asistente de configuración (protegido por contraseña)
-   `https://<tu-dominio-railway>/openclaw` — Interfaz de Control

## Lo que obtienes

-   Puerta de Enlace OpenClaw alojada + Interfaz de Control
-   Asistente de configuración web en `/setup` (sin comandos de terminal)
-   Almacenamiento persistente a través de Volumen Railway (`/data`) para que la configuración/credenciales/espacio de trabajo sobrevivan a los redespiegues
-   Exportación de respaldo en `/setup/export` para migrar fuera de Railway más tarde

## Configuración requerida en Railway

### Redes Públicas

Habilita el **Proxy HTTP** para el servicio.

-   Puerto: `8080`

### Volumen (requerido)

Adjunta un volumen montado en:

-   `/data`

### Variables

Establece estas variables en el servicio:

-   `SETUP_PASSWORD` (requerida)
-   `PORT=8080` (requerida — debe coincidir con el puerto en Redes Públicas)
-   `OPENCLAW_STATE_DIR=/data/.openclaw` (recomendado)
-   `OPENCLAW_WORKSPACE_DIR=/data/workspace` (recomendado)
-   `OPENCLAW_GATEWAY_TOKEN` (recomendado; trátalo como un secreto de administrador)

## Flujo de configuración

1.  Visita `https://<tu-dominio-railway>/setup` e introduce tu `SETUP_PASSWORD`.
2.  Elige un modelo/proveedor de autenticación y pega tu clave.
3.  (Opcional) Añade tokens de Telegram/Discord/Slack.
4.  Haz clic en **Ejecutar configuración**.

Si los MDs de Telegram están configurados para emparejamiento, el asistente de configuración puede aprobar el código de emparejamiento.

## Obteniendo tokens de chat

### Token de bot de Telegram

1.  Envía un mensaje a `@BotFather` en Telegram
2.  Ejecuta `/newbot`
3.  Copia el token (parece `123456789:AA...`)
4.  Pégalo en `/setup`

### Token de bot de Discord

1.  Ve a [https://discord.com/developers/applications](https://discord.com/developers/applications)
2.  **Nueva Aplicación** → elige un nombre
3.  **Bot** → **Añadir Bot**
4.  **Habilita MESSAGE CONTENT INTENT** en Bot → Privileged Gateway Intents (requerido o el bot fallará al iniciar)
5.  Copia el **Token del Bot** y pégalo en `/setup`
6.  Invita el bot a tu servidor (Generador de URL OAuth2; alcances: `bot`, `applications.commands`)

## Copias de seguridad y migración

Descarga una copia de seguridad en:

-   `https://<tu-dominio-railway>/setup/export`

Esto exporta tu estado de OpenClaw + espacio de trabajo para que puedas migrar a otro host sin perder configuración o memoria.

[exe.dev](./exe-dev.md)[Desplegar en Render](./render.md)

---