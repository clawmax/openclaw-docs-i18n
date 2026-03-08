

  First steps

  
# Onboarding (macOS App)

This doc describes the **current** first‑run onboarding flow. The goal is a smooth “day 0” experience: pick where the Gateway runs, connect auth, run the wizard, and let the agent bootstrap itself. For a general overview of onboarding paths, see [Onboarding Overview](./onboarding-overview.md). 

### Step 1: Approve macOS warning

![](../images/start-01-macos-warning.jpeg.md)

### Step 2: Approve find local networks

![](../images/start-02-local-networks.jpeg.md)

### Step 3: Welcome and security notice

![](../images/start-03-security-notice.png.md)

Security trust model:

-   By default, OpenClaw is a personal agent: one trusted operator boundary.
-   Shared/multi-user setups require lock-down (split trust boundaries, keep tool access minimal, and follow [Security](../gateway/security.md)).
-   Local onboarding now defaults new configs to `tools.profile: "coding"` so fresh local setups keep filesystem/runtime tools without forcing the unrestricted `full` profile.
-   If hooks/webhooks or other untrusted content feeds are enabled, use a strong modern model tier and keep strict tool policy/sandboxing.

### Step 4: Local vs Remote

![](../images/start-04-choose-gateway.png.md)

Where does the **Gateway** run?

-   **This Mac (Local only):** onboarding can configure auth and write credentials locally.
-   **Remote (over SSH/Tailnet):** onboarding does **not** configure local auth; credentials must exist on the gateway host.
-   **Configure later:** skip setup and leave the app unconfigured.

> **💡** **Gateway auth tip:**
> 
> -   The wizard now generates a **token** even for loopback, so local WS clients must authenticate.
> -   If you disable auth, any local process can connect; use that only on fully trusted machines.
> -   Use a **token** for multi‑machine access or non‑loopback binds.

### Step 5: Permissions

![](../images/start-05-permissions.png.md)

Onboarding requests TCC permissions needed for:

-   Automation (AppleScript)
-   Notifications
-   Accessibility
-   Screen Recording
-   Microphone
-   Speech Recognition
-   Camera
-   Location

### Step 6: CLI

> **ℹ️** This step is optional

 The app can install the global `openclaw` CLI via npm/pnpm so terminal workflows and launchd tasks work out of the box.

### Step 7: Onboarding Chat (dedicated session)

After setup, the app opens a dedicated onboarding chat session so the agent can introduce itself and guide next steps. This keeps first‑run guidance separate from your normal conversation. See [Bootstrapping](./bootstrapping.md) for what happens on the gateway host during the first agent run.

[Onboarding: CLI](./wizard.md)[Personal Assistant Setup](./openclaw.md)
