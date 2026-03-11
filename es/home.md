

  Inicio

  
# OpenClaw

  ![](../images/openclaw-logo-text-dark.png)
  ![](../images/openclaw-logo-text.png)

> *"¡EXFOLIAR! ¡EXFOLIAR!"* — Una langosta espacial, probablemente

**Puerta de enlace para cualquier SO para agentes de IA en WhatsApp, Telegram, Discord, iMessage y más.** Envía un mensaje, obtén una respuesta de un agente desde tu bolsillo. Los complementos añaden Mattermost y más.

  
  
  

## ¿Qué es OpenClaw?

OpenClaw es una **puerta de enlace autoalojada** que conecta tus aplicaciones de chat favoritas — WhatsApp, Telegram, Discord, iMessage y más — con agentes de IA de programación como Pi. Ejecutas un único proceso de Puerta de enlace en tu propia máquina (o en un servidor), y se convierte en el puente entre tus aplicaciones de mensajería y un asistente de IA siempre disponible.

**¿Para quién es?** Desarrolladores y usuarios avanzados que quieren un asistente de IA personal al que puedan enviar mensajes desde cualquier lugar — sin renunciar al control de sus datos ni depender de un servicio alojado.

**¿Qué lo hace diferente?**

- **Autoalojado**: se ejecuta en tu hardware, bajo tus reglas
- **Multicanal**: una Puerta de enlace sirve a WhatsApp, Telegram, Discord y más simultáneamente
- **Nativo para agentes**: construido para agentes de programación con uso de herramientas, sesiones, memoria y enrutamiento multiagente
- **Código abierto**: licencia MIT, impulsado por la comunidad

**¿Qué necesitas?** Node 22+, una clave API de tu proveedor elegido y 5 minutos. Para la mejor calidad y seguridad, usa el modelo más potente de última generación disponible.

## Cómo funciona

La Puerta de enlace es la única fuente de verdad para sesiones, enrutamiento y conexiones de canales.

## Capacidades clave

  - **Puerta de enlace multicanal** — WhatsApp, Telegram, Discord e iMessage con un único proceso de Puerta de enlace.

  - **Canales con complementos** — Añade Mattermost y más con paquetes de extensión.

  - **Enrutamiento multiagente** — Sesiones aisladas por agente, espacio de trabajo o remitente.

  - **Soporte multimedia** — Envía y recibe imágenes, audio y documentos.

  - **IU Web de Control** — Panel de control en el navegador para chat, configuración, sesiones y nodos.

  

## Inicio rápido

  ### Instalar OpenClaw

```bash
npm install -g openclaw@latest
```

  

  ### Incorporar e instalar el servicio

```bash
openclaw onboard --install-daemon
```

  

  ### Emparejar WhatsApp e iniciar la Puerta de enlace

```bash
openclaw channels login
openclaw gateway --port 18789
```

  

¿Necesitas la instalación completa y la configuración de desarrollo? Consulta [Inicio rápido](./start/quickstart.md).

## Panel de control

Abre la IU de Control en el navegador después de que se inicie la Puerta de enlace.

- Por defecto local: [http://127.0.0.1:18789/](http://127.0.0.1:18789/)
- Acceso remoto: [Superficies web](./web.md) y [Tailscale](./gateway/tailscale.md)

  ![](../images/whatsapp-openclaw.jpg)

## Configuración (opcional)

La configuración reside en `~/.openclaw/openclaw.json`.

- Si **no haces nada**, OpenClaw usa el binario Pi incluido en modo RPC con sesiones por remitente.
- Si quieres asegurarlo, comienza con `channels.whatsapp.allowFrom` y (para grupos) reglas de mención.

Ejemplo:

```json
{
  "channels": {
    "whatsapp": {
      "allowFrom": ["+15555550123"],
      "groups": { "*": { "requireMention": true } }
    }
  },
  "messages": { "groupChat": { "mentionPatterns": ["@openclaw"] } }
}
```

## Comienza aquí

  
  
  
  
  
  

## Aprende más

  
  
  
  
  

---