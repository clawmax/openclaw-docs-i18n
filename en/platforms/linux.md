

  Platforms overview

  
# Linux App

The Gateway is fully supported on Linux. **Node is the recommended runtime**. Bun is not recommended for the Gateway (WhatsApp/Telegram bugs). Native Linux companion apps are planned. Contributions are welcome if you want to help build one.

## Beginner quick path (VPS)

1.  Install Node 22+
2.  `npm i -g openclaw@latest`
3.  `openclaw onboard --install-daemon`
4.  From your laptop: `ssh -N -L 18789:127.0.0.1:18789 @`
5.  Open `http://127.0.0.1:18789/` and paste your token

Step-by-step VPS guide: [exe.dev](../install/exe-dev.md)

## Install

-   [Getting Started](../start/getting-started.md)
-   [Install & updates](../install/updating.md)
-   Optional flows: [Bun (experimental)](../install/bun.md), [Nix](../install/nix.md), [Docker](../install/docker.md)

## Gateway

-   [Gateway runbook](../gateway.md)
-   [Configuration](../gateway/configuration.md)

## Gateway service install (CLI)

Use one of these:

```bash
openclaw onboard --install-daemon
```

Or:

```bash
openclaw gateway install
```

Or:

```bash
openclaw configure
```

Select **Gateway service** when prompted. Repair/migrate:

```bash
openclaw doctor
```

## System control (systemd user unit)

OpenClaw installs a systemd **user** service by default. Use a **system** service for shared or always-on servers. The full unit example and guidance live in the [Gateway runbook](../gateway.md). Minimal setup: Create `~/.config/systemd/user/openclaw-gateway[-].service`:

```ini
[Unit]
Description=OpenClaw Gateway (profile: <profile>, v<version>)
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/openclaw gateway --port 18789
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
```

Enable it:

```bash
systemctl --user enable --now openclaw-gateway[-<profile>].service
```

[macOS App](./macos.md)[Windows (WSL2)](./windows.md)
