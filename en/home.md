

  Home

  
# OpenClaw

  ![](../images/openclaw-logo-text-dark.png)
  ![](../images/openclaw-logo-text.png)

> *"EXFOLIATE! EXFOLIATE!"* — A space lobster, probably

  **Any OS gateway for AI agents across WhatsApp, Telegram, Discord, iMessage, and more.** Send a message, get an agent response from your pocket. Plugins add Mattermost and more.

  
  
  

## What is OpenClaw?

OpenClaw is a **self-hosted gateway** that connects your favorite chat apps — WhatsApp, Telegram, Discord, iMessage, and more — to AI coding agents like Pi. You run a single Gateway process on your own machine (or a server), and it becomes the bridge between your messaging apps and an always-available AI assistant.

**Who is it for?** Developers and power users who want a personal AI assistant they can message from anywhere — without giving up control of their data or relying on a hosted service.

**What makes it different?**

- **Self-hosted**: runs on your hardware, your rules
- **Multi-channel**: one Gateway serves WhatsApp, Telegram, Discord, and more simultaneously
- **Agent-native**: built for coding agents with tool use, sessions, memory, and multi-agent routing
- **Open source**: MIT licensed, community-driven

**What do you need?** Node 22+, an API key from your chosen provider, and 5 minutes. For best quality and security, use the strongest latest-generation model available.

## How it works

The Gateway is the single source of truth for sessions, routing, and channel connections.

## Key capabilities

  - **Multi-channel gateway** — WhatsApp, Telegram, Discord, and iMessage with a single Gateway process.

  - **Plugin channels** — Add Mattermost and more with extension packages.

  - **Multi-agent routing** — Isolated sessions per agent, workspace, or sender.

  - **Media support** — Send and receive images, audio, and documents.

  - **Web Control UI** — Browser dashboard for chat, config, sessions, and nodes.

  

## Quick start

  ### Install OpenClaw

```bash
npm install -g openclaw@latest
```

  

  ### Onboard and install the service

```bash
openclaw onboard --install-daemon
```

  

  ### Pair WhatsApp and start the Gateway

```bash
openclaw channels login
openclaw gateway --port 18789
```

  

Need the full install and dev setup? See [Quick start](./start/quickstart.md).

## Dashboard

Open the browser Control UI after the Gateway starts.

- Local default: [http://127.0.0.1:18789/](http://127.0.0.1:18789/)
- Remote access: [Web surfaces](./web.md) and [Tailscale](./gateway/tailscale.md)

  ![](../images/whatsapp-openclaw.jpg)

## Configuration (optional)

Config lives at `~/.openclaw/openclaw.json`.

- If you **do nothing**, OpenClaw uses the bundled Pi binary in RPC mode with per-sender sessions.
- If you want to lock it down, start with `channels.whatsapp.allowFrom` and (for groups) mention rules.

Example:

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

## Start here

  
  
  
  
  
  

## Learn more

  
  
  
  
  

