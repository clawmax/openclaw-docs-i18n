

  Platforms overview

  
# Platforms

OpenClaw core is written in TypeScript. **Node is the recommended runtime**. Bun is not recommended for the Gateway (WhatsApp/Telegram bugs). Companion apps exist for macOS (menu bar app) and mobile nodes (iOS/Android). Windows and Linux companion apps are planned, but the Gateway is fully supported today. Native companion apps for Windows are also planned; the Gateway is recommended via WSL2.

## Choose your OS

-   macOS: [macOS](./platforms/macos.md)
-   iOS: [iOS](./platforms/ios.md)
-   Android: [Android](./platforms/android.md)
-   Windows: [Windows](./platforms/windows.md)
-   Linux: [Linux](./platforms/linux.md)

## VPS & hosting

-   VPS hub: [VPS hosting](./vps.md)
-   Fly.io: [Fly.io](./install/fly.md)
-   Hetzner (Docker): [Hetzner](./install/hetzner.md)
-   GCP (Compute Engine): [GCP](./install/gcp.md)
-   exe.dev (VM + HTTPS proxy): [exe.dev](./install/exe-dev.md)

## Common links

-   Install guide: [Getting Started](./start/getting-started.md)
-   Gateway runbook: [Gateway](./gateway.md)
-   Gateway configuration: [Configuration](./gateway/configuration.md)
-   Service status: `openclaw gateway status`

## Gateway service install (CLI)

Use one of these (all supported):

-   Wizard (recommended): `openclaw onboard --install-daemon`
-   Direct: `openclaw gateway install`
-   Configure flow: `openclaw configure` → select **Gateway service**
-   Repair/migrate: `openclaw doctor` (offers to install or fix the service)

The service target depends on OS:

-   macOS: LaunchAgent (`ai.openclaw.gateway` or `ai.openclaw.`; legacy `com.openclaw.*`)
-   Linux/WSL2: systemd user service (`openclaw-gateway[-].service`)

[macOS App](./platforms/macos.md)
