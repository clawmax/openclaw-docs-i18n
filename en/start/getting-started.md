

  First steps

  
# Getting Started

Goal: go from zero to a first working chat with minimal setup. 

> **ℹ️** Fastest chat: open the Control UI (no channel setup needed). Run `openclaw dashboard` and chat in the browser, or open `http://127.0.0.1:18789/` on the gateway host. Docs: [Dashboard](../web/dashboard.md) and [Control UI](../web/control-ui.md).

## Prereqs

-   Node 22 or newer

> **💡** Check your Node version with `node --version` if you are unsure.

## Quick setup (CLI)

### Step 1: Install OpenClaw (recommended)

> **ℹ️** Other install methods and requirements: [Install](../install.md).

### Step 2: Run the onboarding wizard

```bash
openclaw onboard --install-daemon
```

The wizard configures auth, gateway settings, and optional channels. See [Onboarding Wizard](./wizard.md) for details.

### Step 3: Check the Gateway

If you installed the service, it should already be running:

```bash
openclaw gateway status
```

### Step 4: Open the Control UI

```bash
openclaw dashboard
```

 

> **✅** If the Control UI loads, your Gateway is ready for use.

## Optional checks and extras

Useful for quick tests or troubleshooting.

```bash
openclaw gateway --port 18789
```

Requires a configured channel.

```bash
openclaw message send --target +15555550123 --message "Hello from OpenClaw"
```

## Useful environment variables

If you run OpenClaw as a service account or want custom config/state locations:

-   `OPENCLAW_HOME` sets the home directory used for internal path resolution.
-   `OPENCLAW_STATE_DIR` overrides the state directory.
-   `OPENCLAW_CONFIG_PATH` overrides the config file path.

Full environment variable reference: [Environment vars](../help/environment.md).

## Go deeper

## What you will have

-   A running Gateway
-   Auth configured
-   Control UI access or a connected channel

## Next steps

-   DM safety and approvals: [Pairing](../channels/pairing.md)
-   Connect more channels: [Channels](../channels.md)
-   Advanced workflows and from source: [Setup](./setup.md)

[Features](../concepts/features.md)[Onboarding Overview](./onboarding-overview.md)
