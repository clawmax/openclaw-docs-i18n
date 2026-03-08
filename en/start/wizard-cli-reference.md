

  Guides

  
# CLI Onboarding Reference

This page is the full reference for `openclaw onboard`. For the short guide, see [Onboarding Wizard (CLI)](./wizard.md).

## What the wizard does

Local mode (default) walks you through:

-   Model and auth setup (OpenAI Code subscription OAuth, Anthropic API key or setup token, plus MiniMax, GLM, Moonshot, and AI Gateway options)
-   Workspace location and bootstrap files
-   Gateway settings (port, bind, auth, tailscale)
-   Channels and providers (Telegram, WhatsApp, Discord, Google Chat, Mattermost plugin, Signal)
-   Daemon install (LaunchAgent or systemd user unit)
-   Health check
-   Skills setup

Remote mode configures this machine to connect to a gateway elsewhere. It does not install or modify anything on the remote host.

## Local flow details

### Step 1: Existing config detection

-   If `~/.openclaw/openclaw.json` exists, choose Keep, Modify, or Reset.
-   Re-running the wizard does not wipe anything unless you explicitly choose Reset (or pass `--reset`).
-   CLI `--reset` defaults to `config+creds+sessions`; use `--reset-scope full` to also remove workspace.
-   If config is invalid or contains legacy keys, the wizard stops and asks you to run `openclaw doctor` before continuing.
-   Reset uses `trash` and offers scopes:
    -   Config only
    -   Config + credentials + sessions
    -   Full reset (also removes workspace)

### Step 2: Model and auth

-   Full option matrix is in [Auth and model options](#auth-and-model-options).

### Step 3: Workspace

-   Default `~/.openclaw/workspace` (configurable).
-   Seeds workspace files needed for first-run bootstrap ritual.
-   Workspace layout: [Agent workspace](../concepts/agent-workspace.md).

### Step 4: Gateway

-   Prompts for port, bind, auth mode, and tailscale exposure.
-   Recommended: keep token auth enabled even for loopback so local WS clients must authenticate.
-   In token mode, interactive onboarding offers:
    -   **Generate/store plaintext token** (default)
    -   **Use SecretRef** (opt-in)
-   In password mode, interactive onboarding also supports plaintext or SecretRef storage.
-   Non-interactive token SecretRef path: `--gateway-token-ref-env <ENV_VAR>`.
    -   Requires a non-empty env var in the onboarding process environment.
    -   Cannot be combined with `--gateway-token`.
-   Disable auth only if you fully trust every local process.
-   Non-loopback binds still require auth.

### Step 5: Channels

-   [WhatsApp](../channels/whatsapp.md): optional QR login
-   [Telegram](../channels/telegram.md): bot token
-   [Discord](../channels/discord.md): bot token
-   [Google Chat](../channels/googlechat.md): service account JSON + webhook audience
-   [Mattermost](../channels/mattermost.md) plugin: bot token + base URL
-   [Signal](../channels/signal.md): optional `signal-cli` install + account config
-   [BlueBubbles](../channels/bluebubbles.md): recommended for iMessage; server URL + password + webhook
-   [iMessage](../channels/imessage.md): legacy `imsg` CLI path + DB access
-   DM security: default is pairing. First DM sends a code; approve via `openclaw pairing approve  ` or use allowlists.

### Step 6: Daemon install

-   macOS: LaunchAgent
    -   Requires logged-in user session; for headless, use a custom LaunchDaemon (not shipped).
-   Linux and Windows via WSL2: systemd user unit
    -   Wizard attempts `loginctl enable-linger ` so gateway stays up after logout.
    -   May prompt for sudo (writes `/var/lib/systemd/linger`); it tries without sudo first.
-   Runtime selection: Node (recommended; required for WhatsApp and Telegram). Bun is not recommended.

### Step 7: Health check

-   Starts gateway (if needed) and runs `openclaw health`.
-   `openclaw status --deep` adds gateway health probes to status output.

### Step 8: Skills

-   Reads available skills and checks requirements.
-   Lets you choose node manager: npm or pnpm (bun not recommended).
-   Installs optional dependencies (some use Homebrew on macOS).

### Step 9: Finish

-   Summary and next steps, including iOS, Android, and macOS app options.

 

> **ℹ️** If no GUI is detected, the wizard prints SSH port-forward instructions for the Control UI instead of opening a browser. If Control UI assets are missing, the wizard attempts to build them; fallback is `pnpm ui:build` (auto-installs UI deps).

## Remote mode details

Remote mode configures this machine to connect to a gateway elsewhere. 

> **ℹ️** Remote mode does not install or modify anything on the remote host.

 What you set:

-   Remote gateway URL (`ws://...`)
-   Token if remote gateway auth is required (recommended)

> **ℹ️** -   If gateway is loopback-only, use SSH tunneling or a tailnet.
> -   Discovery hints:
>     -   macOS: Bonjour (`dns-sd`)
>     -   Linux: Avahi (`avahi-browse`)

## Auth and model options

Uses `ANTHROPIC_API_KEY` if present or prompts for a key, then saves it for daemon use.

-   macOS: checks Keychain item “Claude Code-credentials”
-   Linux and Windows: reuses `~/.claude/.credentials.json` if present

On macOS, choose “Always Allow” so launchd starts do not block.

Run `claude setup-token` on any machine, then paste the token. You can name it; blank uses default.

If `~/.codex/auth.json` exists, the wizard can reuse it.

Browser flow; paste `code#state`.Sets `agents.defaults.model` to `openai-codex/gpt-5.4` when model is unset or `openai/*`.

Uses `OPENAI_API_KEY` if present or prompts for a key, then stores the credential in auth profiles.Sets `agents.defaults.model` to `openai/gpt-5.1-codex` when model is unset, `openai/*`, or `openai-codex/*`.

Prompts for `XAI_API_KEY` and configures xAI as a model provider.

Prompts for `OPENCODE_API_KEY` (or `OPENCODE_ZEN_API_KEY`). Setup URL: [opencode.ai/auth](https://opencode.ai/auth).

Stores the key for you.

Prompts for `AI_GATEWAY_API_KEY`. More detail: [Vercel AI Gateway](../providers/vercel-ai-gateway.md).

Prompts for account ID, gateway ID, and `CLOUDFLARE_AI_GATEWAY_API_KEY`. More detail: [Cloudflare AI Gateway](../providers/cloudflare-ai-gateway.md).

Config is auto-written. More detail: [MiniMax](../providers/minimax.md).

Prompts for `SYNTHETIC_API_KEY`. More detail: [Synthetic](../providers/synthetic.md).

Moonshot (Kimi K2) and Kimi Coding configs are auto-written. More detail: [Moonshot AI (Kimi + Kimi Coding)](../providers/moonshot.md).

Works with OpenAI-compatible and Anthropic-compatible endpoints.Interactive onboarding supports the same API key storage choices as other provider API key flows:

-   **Paste API key now** (plaintext)
-   **Use secret reference** (env ref or configured provider ref, with preflight validation)

Non-interactive flags:

-   `--auth-choice custom-api-key`
-   `--custom-base-url`
-   `--custom-model-id`
-   `--custom-api-key` (optional; falls back to `CUSTOM_API_KEY`)
-   `--custom-provider-id` (optional)
-   `--custom-compatibility <openai|anthropic>` (optional; default `openai`)

Leaves auth unconfigured.

 Model behavior:

-   Pick default model from detected options, or enter provider and model manually.
-   Wizard runs a model check and warns if the configured model is unknown or missing auth.

Credential and profile paths:

-   OAuth credentials: `~/.openclaw/credentials/oauth.json`
-   Auth profiles (API keys + OAuth): `~/.openclaw/agents//agent/auth-profiles.json`

Credential storage mode:

-   Default onboarding behavior persists API keys as plaintext values in auth profiles.
-   `--secret-input-mode ref` enables reference mode instead of plaintext key storage. In interactive onboarding, you can choose either:
    -   environment variable ref (for example `keyRef: { source: "env", provider: "default", id: "OPENAI_API_KEY" }`)
    -   configured provider ref (`file` or `exec`) with provider alias + id
-   Interactive reference mode runs a fast preflight validation before saving.
    -   Env refs: validates variable name + non-empty value in the current onboarding environment.
    -   Provider refs: validates provider config and resolves the requested id.
    -   If preflight fails, onboarding shows the error and lets you retry.
-   In non-interactive mode, `--secret-input-mode ref` is env-backed only.
    -   Set the provider env var in the onboarding process environment.
    -   Inline key flags (for example `--openai-api-key`) require that env var to be set; otherwise onboarding fails fast.
    -   For custom providers, non-interactive `ref` mode stores `models.providers..apiKey` as `{ source: "env", provider: "default", id: "CUSTOM_API_KEY" }`.
    -   In that custom-provider case, `--custom-api-key` requires `CUSTOM_API_KEY` to be set; otherwise onboarding fails fast.
-   Gateway auth credentials support plaintext and SecretRef choices in interactive onboarding:
    -   Token mode: **Generate/store plaintext token** (default) or **Use SecretRef**.
    -   Password mode: plaintext or SecretRef.
-   Non-interactive token SecretRef path: `--gateway-token-ref-env <ENV_VAR>`.
-   Existing plaintext setups continue to work unchanged.

> **ℹ️** Headless and server tip: complete OAuth on a machine with a browser, then copy `~/.openclaw/credentials/oauth.json` (or `$OPENCLAW_STATE_DIR/credentials/oauth.json`) to the gateway host.

## Outputs and internals

Typical fields in `~/.openclaw/openclaw.json`:

-   `agents.defaults.workspace`
-   `agents.defaults.model` / `models.providers` (if Minimax chosen)
-   `tools.profile` (local onboarding defaults to `"coding"` when unset; existing explicit values are preserved)
-   `gateway.*` (mode, bind, auth, tailscale)
-   `session.dmScope` (local onboarding defaults this to `per-channel-peer` when unset; existing explicit values are preserved)
-   `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
-   Channel allowlists (Slack, Discord, Matrix, Microsoft Teams) when you opt in during prompts (names resolve to IDs when possible)
-   `skills.install.nodeManager`
-   `wizard.lastRunAt`
-   `wizard.lastRunVersion`
-   `wizard.lastRunCommit`
-   `wizard.lastRunCommand`
-   `wizard.lastRunMode`

`openclaw agents add` writes `agents.list[]` and optional `bindings`. WhatsApp credentials go under `~/.openclaw/credentials/whatsapp//`. Sessions are stored under `~/.openclaw/agents//sessions/`. 

> **ℹ️** Some channels are delivered as plugins. When selected during onboarding, the wizard prompts to install the plugin (npm or local path) before channel configuration.

 Gateway wizard RPC:

-   `wizard.start`
-   `wizard.next`
-   `wizard.cancel`
-   `wizard.status`

Clients (macOS app and Control UI) can render steps without re-implementing onboarding logic. Signal setup behavior:

-   Downloads the appropriate release asset
-   Stores it under `~/.openclaw/tools/signal-cli//`
-   Writes `channels.signal.cliPath` in config
-   JVM builds require Java 21
-   Native builds are used when available
-   Windows uses WSL2 and follows Linux signal-cli flow inside WSL

## Related docs

-   Onboarding hub: [Onboarding Wizard (CLI)](./wizard.md)
-   Automation and scripts: [CLI Automation](./wizard-cli-automation.md)
-   Command reference: [`openclaw onboard`](../cli/onboard.md)

[Personal Assistant Setup](./openclaw.md)[CLI automation](./wizard-cli-automation.md)
