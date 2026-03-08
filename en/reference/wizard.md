

  Technical reference

  
# Onboarding Wizard Reference

This is the full reference for the `openclaw onboard` CLI wizard. For a high-level overview, see [Onboarding Wizard](../start/wizard.md).

## Flow details (local mode)

### Step 1: Existing config detection

-   If `~/.openclaw/openclaw.json` exists, choose **Keep / Modify / Reset**.
-   Re-running the wizard does **not** wipe anything unless you explicitly choose **Reset** (or pass `--reset`).
-   CLI `--reset` defaults to `config+creds+sessions`; use `--reset-scope full` to also remove workspace.
-   If the config is invalid or contains legacy keys, the wizard stops and asks you to run `openclaw doctor` before continuing.
-   Reset uses `trash` (never `rm`) and offers scopes:
    -   Config only
    -   Config + credentials + sessions
    -   Full reset (also removes workspace)

### Step 2: Model/Auth

-   **Anthropic API key**: uses `ANTHROPIC_API_KEY` if present or prompts for a key, then saves it for daemon use.
-   **Anthropic OAuth (Claude Code CLI)**: on macOS the wizard checks Keychain item “Claude Code-credentials” (choose “Always Allow” so launchd starts don’t block); on Linux/Windows it reuses `~/.claude/.credentials.json` if present.
-   **Anthropic token (paste setup-token)**: run `claude setup-token` on any machine, then paste the token (you can name it; blank = default).
-   **OpenAI Code (Codex) subscription (Codex CLI)**: if `~/.codex/auth.json` exists, the wizard can reuse it.
-   **OpenAI Code (Codex) subscription (OAuth)**: browser flow; paste the `code#state`.
    -   Sets `agents.defaults.model` to `openai-codex/gpt-5.2` when model is unset or `openai/*`.
-   **OpenAI API key**: uses `OPENAI_API_KEY` if present or prompts for a key, then stores it in auth profiles.
-   **xAI (Grok) API key**: prompts for `XAI_API_KEY` and configures xAI as a model provider.
-   **OpenCode Zen (multi-model proxy)**: prompts for `OPENCODE_API_KEY` (or `OPENCODE_ZEN_API_KEY`, get it at [https://opencode.ai/auth](https://opencode.ai/auth)).
-   **API key**: stores the key for you.
-   **Vercel AI Gateway (multi-model proxy)**: prompts for `AI_GATEWAY_API_KEY`.
-   More detail: [Vercel AI Gateway](../providers/vercel-ai-gateway.md)
-   **Cloudflare AI Gateway**: prompts for Account ID, Gateway ID, and `CLOUDFLARE_AI_GATEWAY_API_KEY`.
-   More detail: [Cloudflare AI Gateway](../providers/cloudflare-ai-gateway.md)
-   **MiniMax M2.5**: config is auto-written.
-   More detail: [MiniMax](../providers/minimax.md)
-   **Synthetic (Anthropic-compatible)**: prompts for `SYNTHETIC_API_KEY`.
-   More detail: [Synthetic](../providers/synthetic.md)
-   **Moonshot (Kimi K2)**: config is auto-written.
-   **Kimi Coding**: config is auto-written.
-   More detail: [Moonshot AI (Kimi + Kimi Coding)](../providers/moonshot.md)
-   **Skip**: no auth configured yet.
-   Pick a default model from detected options (or enter provider/model manually). For best quality and lower prompt-injection risk, choose the strongest latest-generation model available in your provider stack.
-   Wizard runs a model check and warns if the configured model is unknown or missing auth.
-   API key storage mode defaults to plaintext auth-profile values. Use `--secret-input-mode ref` to store env-backed refs instead (for example `keyRef: { source: "env", provider: "default", id: "OPENAI_API_KEY" }`).
-   OAuth credentials live in `~/.openclaw/credentials/oauth.json`; auth profiles live in `~/.openclaw/agents//agent/auth-profiles.json` (API keys + OAuth).
-   More detail: [/concepts/oauth](../concepts/oauth.md)

> **ℹ️** Headless/server tip: complete OAuth on a machine with a browser, then copy `~/.openclaw/credentials/oauth.json` (or `$OPENCLAW_STATE_DIR/credentials/oauth.json`) to the gateway host.

### Step 3: Workspace

-   Default `~/.openclaw/workspace` (configurable).
-   Seeds the workspace files needed for the agent bootstrap ritual.
-   Full workspace layout + backup guide: [Agent workspace](../concepts/agent-workspace.md)

### Step 4: Gateway

-   Port, bind, auth mode, tailscale exposure.
-   Auth recommendation: keep **Token** even for loopback so local WS clients must authenticate.
-   In token mode, interactive onboarding offers:
    -   **Generate/store plaintext token** (default)
    -   **Use SecretRef** (opt-in)
    -   Quickstart reuses existing `gateway.auth.token` SecretRefs across `env`, `file`, and `exec` providers for onboarding probe/dashboard bootstrap.
    -   If that SecretRef is configured but cannot be resolved, onboarding fails early with a clear fix message instead of silently degrading runtime auth.
-   In password mode, interactive onboarding also supports plaintext or SecretRef storage.
-   Non-interactive token SecretRef path: `--gateway-token-ref-env <ENV_VAR>`.
    -   Requires a non-empty env var in the onboarding process environment.
    -   Cannot be combined with `--gateway-token`.
-   Disable auth only if you fully trust every local process.
-   Non‑loopback binds still require auth.

### Step 5: Channels

-   [WhatsApp](../channels/whatsapp.md): optional QR login.
-   [Telegram](../channels/telegram.md): bot token.
-   [Discord](../channels/discord.md): bot token.
-   [Google Chat](../channels/googlechat.md): service account JSON + webhook audience.
-   [Mattermost](../channels/mattermost.md) (plugin): bot token + base URL.
-   [Signal](../channels/signal.md): optional `signal-cli` install + account config.
-   [BlueBubbles](../channels/bluebubbles.md): **recommended for iMessage**; server URL + password + webhook.
-   [iMessage](../channels/imessage.md): legacy `imsg` CLI path + DB access.
-   DM security: default is pairing. First DM sends a code; approve via `openclaw pairing approve  ` or use allowlists.

### Step 6: Web search

-   Pick a provider: Perplexity, Brave, Gemini, Grok, or Kimi (or skip).
-   Paste your API key (QuickStart auto-detects keys from env vars or existing config).
-   Skip with `--skip-search`.
-   Configure later: `openclaw configure --section web`.

### Step 7: Daemon install

-   macOS: LaunchAgent
    -   Requires a logged-in user session; for headless, use a custom LaunchDaemon (not shipped).
-   Linux (and Windows via WSL2): systemd user unit
    -   Wizard attempts to enable lingering via `loginctl enable-linger ` so the Gateway stays up after logout.
    -   May prompt for sudo (writes `/var/lib/systemd/linger`); it tries without sudo first.
-   **Runtime selection:** Node (recommended; required for WhatsApp/Telegram). Bun is **not recommended**.
-   If token auth requires a token and `gateway.auth.token` is SecretRef-managed, daemon install validates it but does not persist resolved plaintext token values into supervisor service environment metadata.
-   If token auth requires a token and the configured token SecretRef is unresolved, daemon install is blocked with actionable guidance.
-   If both `gateway.auth.token` and `gateway.auth.password` are configured and `gateway.auth.mode` is unset, daemon install is blocked until mode is set explicitly.

### Step 8: Health check

-   Starts the Gateway (if needed) and runs `openclaw health`.
-   Tip: `openclaw status --deep` adds gateway health probes to status output (requires a reachable gateway).

### Step 9: Skills (recommended)

-   Reads the available skills and checks requirements.
-   Lets you choose a node manager: **npm / pnpm** (bun not recommended).
-   Installs optional dependencies (some use Homebrew on macOS).

### Step 10: Finish

-   Summary + next steps, including iOS/Android/macOS apps for extra features.

 

> **ℹ️** If no GUI is detected, the wizard prints SSH port-forward instructions for the Control UI instead of opening a browser. If the Control UI assets are missing, the wizard attempts to build them; fallback is `pnpm ui:build` (auto-installs UI deps).

## Non-interactive mode

Use `--non-interactive` to automate or script onboarding:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

Add `--json` for a machine‑readable summary. Gateway token SecretRef in non-interactive mode:

```bash
export OPENCLAW_GATEWAY_TOKEN="your-token"
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice skip \
  --gateway-auth token \
  --gateway-token-ref-env OPENCLAW_GATEWAY_TOKEN
```

`--gateway-token` and `--gateway-token-ref-env` are mutually exclusive. 

> **ℹ️** `--json` does **not** imply non-interactive mode. Use `--non-interactive` (and `--workspace`) for scripts.

 

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice gemini-api-key \
  --gemini-api-key "$GEMINI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice zai-api-key \
  --zai-api-key "$ZAI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice cloudflare-ai-gateway-api-key \
  --cloudflare-ai-gateway-account-id "your-account-id" \
  --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
  --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice moonshot-api-key \
  --moonshot-api-key "$MOONSHOT_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice synthetic-api-key \
  --synthetic-api-key "$SYNTHETIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice opencode-zen \
  --opencode-zen-api-key "$OPENCODE_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

### Add agent (non-interactive)

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

## Gateway wizard RPC

The Gateway exposes the wizard flow over RPC (`wizard.start`, `wizard.next`, `wizard.cancel`, `wizard.status`). Clients (macOS app, Control UI) can render steps without re‑implementing onboarding logic.

## Signal setup (signal-cli)

The wizard can install `signal-cli` from GitHub releases:

-   Downloads the appropriate release asset.
-   Stores it under `~/.openclaw/tools/signal-cli//`.
-   Writes `channels.signal.cliPath` to your config.

Notes:

-   JVM builds require **Java 21**.
-   Native builds are used when available.
-   Windows uses WSL2; signal-cli install follows the Linux flow inside WSL.

## What the wizard writes

Typical fields in `~/.openclaw/openclaw.json`:

-   `agents.defaults.workspace`
-   `agents.defaults.model` / `models.providers` (if Minimax chosen)
-   `tools.profile` (local onboarding defaults to `"coding"` when unset; existing explicit values are preserved)
-   `gateway.*` (mode, bind, auth, tailscale)
-   `session.dmScope` (behavior details: [CLI Onboarding Reference](../start/wizard-cli-reference.md#outputs-and-internals))
-   `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
-   Channel allowlists (Slack/Discord/Matrix/Microsoft Teams) when you opt in during the prompts (names resolve to IDs when possible).
-   `skills.install.nodeManager`
-   `wizard.lastRunAt`
-   `wizard.lastRunVersion`
-   `wizard.lastRunCommit`
-   `wizard.lastRunCommand`
-   `wizard.lastRunMode`

`openclaw agents add` writes `agents.list[]` and optional `bindings`. WhatsApp credentials go under `~/.openclaw/credentials/whatsapp//`. Sessions are stored under `~/.openclaw/agents//sessions/`. Some channels are delivered as plugins. When you pick one during onboarding, the wizard will prompt to install it (npm or a local path) before it can be configured.

## Related docs

-   Wizard overview: [Onboarding Wizard](../start/wizard.md)
-   macOS app onboarding: [Onboarding](../start/onboarding.md)
-   Config reference: [Gateway configuration](../gateway/configuration.md)
-   Providers: [WhatsApp](../channels/whatsapp.md), [Telegram](../channels/telegram.md), [Discord](../channels/discord.md), [Google Chat](../channels/googlechat.md), [Signal](../channels/signal.md), [BlueBubbles](../channels/bluebubbles.md) (iMessage), [iMessage](../channels/imessage.md) (legacy)
-   Skills: [Skills](../tools/skills.md), [Skills config](../tools/skills-config.md)

[USER](./templates/USER.md)[Token Use and Costs](./token-use.md)
