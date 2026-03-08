

  Configuration and operations

  
# Configuration

OpenClaw reads an optional **JSON5** config from `~/.openclaw/openclaw.json`. If the file is missing, OpenClaw uses safe defaults. Common reasons to add a config:

-   Connect channels and control who can message the bot
-   Set models, tools, sandboxing, or automation (cron, hooks)
-   Tune sessions, media, networking, or UI

See the [full reference](./configuration-reference.md) for every available field. 

> **💡** **New to configuration?** Start with `openclaw onboard` for interactive setup, or check out the [Configuration Examples](./configuration-examples.md) guide for complete copy-paste configs.

## Minimal config

```
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## Editing config

```bash
openclaw onboard       # full setup wizard
openclaw configure     # config wizard
```

```bash
openclaw config get agents.defaults.workspace
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config unset tools.web.search.apiKey
```

Open [http://127.0.0.1:18789](http://127.0.0.1:18789) and use the **Config** tab. The Control UI renders a form from the config schema, with a **Raw JSON** editor as an escape hatch.

Edit `~/.openclaw/openclaw.json` directly. The Gateway watches the file and applies changes automatically (see [hot reload](#config-hot-reload)).

## Strict validation

> **⚠️** OpenClaw only accepts configurations that fully match the schema. Unknown keys, malformed types, or invalid values cause the Gateway to **refuse to start**. The only root-level exception is `$schema` (string), so editors can attach JSON Schema metadata.

 When validation fails:

-   The Gateway does not boot
-   Only diagnostic commands work (`openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`)
-   Run `openclaw doctor` to see exact issues
-   Run `openclaw doctor --fix` (or `--yes`) to apply repairs

## Common tasks

Each channel has its own config section under `channels.`. See the dedicated channel page for setup steps:

-   [WhatsApp](../channels/whatsapp.md) — `channels.whatsapp`
-   [Telegram](../channels/telegram.md) — `channels.telegram`
-   [Discord](../channels/discord.md) — `channels.discord`
-   [Slack](../channels/slack.md) — `channels.slack`
-   [Signal](../channels/signal.md) — `channels.signal`
-   [iMessage](../channels/imessage.md) — `channels.imessage`
-   [Google Chat](../channels/googlechat.md) — `channels.googlechat`
-   [Mattermost](../channels/mattermost.md) — `channels.mattermost`
-   [MS Teams](../channels/msteams.md) — `channels.msteams`

All channels share the same DM policy pattern:

```json
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",   // pairing | allowlist | open | disabled
      allowFrom: ["tg:123"], // only for allowlist/open
    },
  },
}
```

Set the primary model and optional fallbacks:

```json
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["openai/gpt-5.2"],
      },
      models: {
        "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
        "openai/gpt-5.2": { alias: "GPT" },
      },
    },
  },
}
```

-   `agents.defaults.models` defines the model catalog and acts as the allowlist for `/model`.
-   Model refs use `provider/model` format (e.g. `anthropic/claude-opus-4-6`).
-   `agents.defaults.imageMaxDimensionPx` controls transcript/tool image downscaling (default `1200`); lower values usually reduce vision-token usage on screenshot-heavy runs.
-   See [Models CLI](../concepts/models.md) for switching models in chat and [Model Failover](../concepts/model-failover.md) for auth rotation and fallback behavior.
-   For custom/self-hosted providers, see [Custom providers](./configuration-reference.md#custom-providers-and-base-urls) in the reference.

DM access is controlled per channel via `dmPolicy`:

-   `"pairing"` (default): unknown senders get a one-time pairing code to approve
-   `"allowlist"`: only senders in `allowFrom` (or the paired allow store)
-   `"open"`: allow all inbound DMs (requires `allowFrom: ["*"]`)
-   `"disabled"`: ignore all DMs

For groups, use `groupPolicy` + `groupAllowFrom` or channel-specific allowlists.See the [full reference](./configuration-reference.md#dm-and-group-access) for per-channel details.

Group messages default to **require mention**. Configure patterns per agent:

```json
{
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          mentionPatterns: ["@openclaw", "openclaw"],
        },
      },
    ],
  },
  channels: {
    whatsapp: {
      groups: { "*": { requireMention: true } },
    },
  },
}
```

-   **Metadata mentions**: native @-mentions (WhatsApp tap-to-mention, Telegram @bot, etc.)
-   **Text patterns**: regex patterns in `mentionPatterns`
-   See [full reference](./configuration-reference.md#group-chat-mention-gating) for per-channel overrides and self-chat mode.

Sessions control conversation continuity and isolation:

```json
{
  session: {
    dmScope: "per-channel-peer",  // recommended for multi-user
    threadBindings: {
      enabled: true,
      idleHours: 24,
      maxAgeHours: 0,
    },
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 120,
    },
  },
}
```

-   `dmScope`: `main` (shared) | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
-   `threadBindings`: global defaults for thread-bound session routing (Discord supports `/focus`, `/unfocus`, `/agents`, `/session idle`, and `/session max-age`).
-   See [Session Management](../concepts/session.md) for scoping, identity links, and send policy.
-   See [full reference](./configuration-reference.md#session) for all fields.

Run agent sessions in isolated Docker containers:

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",  // off | non-main | all
        scope: "agent",    // session | agent | shared
      },
    },
  },
}
```

Build the image first: `scripts/sandbox-setup.sh`See [Sandboxing](./sandboxing.md) for the full guide and [full reference](./configuration-reference.md#sandbox) for all options.

```json
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last",
      },
    },
  },
}
```

-   `every`: duration string (`30m`, `2h`). Set `0m` to disable.
-   `target`: `last` | `whatsapp` | `telegram` | `discord` | `none`
-   `directPolicy`: `allow` (default) or `block` for DM-style heartbeat targets
-   See [Heartbeat](./heartbeat.md) for the full guide.

```json
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
    sessionRetention: "24h",
    runLog: {
      maxBytes: "2mb",
      keepLines: 2000,
    },
  },
}
```

-   `sessionRetention`: prune completed isolated run sessions from `sessions.json` (default `24h`; set `false` to disable).
-   `runLog`: prune `cron/runs/.jsonl` by size and retained lines.
-   See [Cron jobs](../automation/cron-jobs.md) for feature overview and CLI examples.

Enable HTTP webhook endpoints on the Gateway:

```json
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        agentId: "main",
        deliver: true,
      },
    ],
  },
}
```

Security note:

-   Treat all hook/webhook payload content as untrusted input.
-   Keep unsafe-content bypass flags disabled (`hooks.gmail.allowUnsafeExternalContent`, `hooks.mappings[].allowUnsafeExternalContent`) unless doing tightly scoped debugging.
-   For hook-driven agents, prefer strong modern model tiers and strict tool policy (for example messaging-only plus sandboxing where possible).

See [full reference](./configuration-reference.md#hooks) for all mapping options and Gmail integration.

Run multiple isolated agents with separate workspaces and sessions:

```json
{
  agents: {
    list: [
      { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
      { id: "work", workspace: "~/.openclaw/workspace-work" },
    ],
  },
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
  ],
}
```

See [Multi-Agent](../concepts/multi-agent.md) and [full reference](./configuration-reference.md#multi-agent-routing) for binding rules and per-agent access profiles.

Use `$include` to organize large configs:

```
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },
  agents: { $include: "./agents.json5" },
  broadcast: {
    $include: ["./clients/a.json5", "./clients/b.json5"],
  },
}
```

-   **Single file**: replaces the containing object
-   **Array of files**: deep-merged in order (later wins)
-   **Sibling keys**: merged after includes (override included values)
-   **Nested includes**: supported up to 10 levels deep
-   **Relative paths**: resolved relative to the including file
-   **Error handling**: clear errors for missing files, parse errors, and circular includes

## Config hot reload

The Gateway watches `~/.openclaw/openclaw.json` and applies changes automatically — no manual restart needed for most settings.

### Reload modes

| Mode | Behavior |
| --- | --- |
| **`hybrid`** (default) | Hot-applies safe changes instantly. Automatically restarts for critical ones. |
| **`hot`** | Hot-applies safe changes only. Logs a warning when a restart is needed — you handle it. |
| **`restart`** | Restarts the Gateway on any config change, safe or not. |
| **`off`** | Disables file watching. Changes take effect on the next manual restart. |

```json
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

### What hot-applies vs what needs a restart

Most fields hot-apply without downtime. In `hybrid` mode, restart-required changes are handled automatically.

| Category | Fields | Restart needed? |
| --- | --- | --- |
| Channels | `channels.*`, `web` (WhatsApp) — all built-in and extension channels | No |
| Agent & models | `agent`, `agents`, `models`, `routing` | No |
| Automation | `hooks`, `cron`, `agent.heartbeat` | No |
| Sessions & messages | `session`, `messages` | No |
| Tools & media | `tools`, `browser`, `skills`, `audio`, `talk` | No |
| UI & misc | `ui`, `logging`, `identity`, `bindings` | No |
| Gateway server | `gateway.*` (port, bind, auth, tailscale, TLS, HTTP) | **Yes** |
| Infrastructure | `discovery`, `canvasHost`, `plugins` | **Yes** |

> **ℹ️** `gateway.reload` and `gateway.remote` are exceptions — changing them does **not** trigger a restart.

## Config RPC (programmatic updates)

> **ℹ️** Control-plane write RPCs (`config.apply`, `config.patch`, `update.run`) are rate-limited to **3 requests per 60 seconds** per `deviceId+clientIp`. When limited, the RPC returns `UNAVAILABLE` with `retryAfterMs`.

 

Validates + writes the full config and restarts the Gateway in one step.

`config.apply` replaces the **entire config**. Use `config.patch` for partial updates, or `openclaw config set` for single keys.

Params:

-   `raw` (string) — JSON5 payload for the entire config
-   `baseHash` (optional) — config hash from `config.get` (required when config exists)
-   `sessionKey` (optional) — session key for the post-restart wake-up ping
-   `note` (optional) — note for the restart sentinel
-   `restartDelayMs` (optional) — delay before restart (default 2000)

Restart requests are coalesced while one is already pending/in-flight, and a 30-second cooldown applies between restart cycles.

```bash
openclaw gateway call config.get --params '{}'  # capture payload.hash
openclaw gateway call config.apply --params '{
  "raw": "{ agents: { defaults: { workspace: \"~/.openclaw/workspace\" } } }",
  "baseHash": "<hash>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123"
}'
```

Merges a partial update into the existing config (JSON merge patch semantics):

-   Objects merge recursively
-   `null` deletes a key
-   Arrays replace

Params:

-   `raw` (string) — JSON5 with just the keys to change
-   `baseHash` (required) — config hash from `config.get`
-   `sessionKey`, `note`, `restartDelayMs` — same as `config.apply`

Restart behavior matches `config.apply`: coalesced pending restarts plus a 30-second cooldown between restart cycles.

```bash
openclaw gateway call config.patch --params '{
  "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
  "baseHash": "<hash>"
}'
```

## Environment variables

OpenClaw reads env vars from the parent process plus:

-   `.env` from the current working directory (if present)
-   `~/.openclaw/.env` (global fallback)

Neither file overrides existing env vars. You can also set inline env vars in config:

```json
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

If enabled and expected keys aren’t set, OpenClaw runs your login shell and imports only the missing keys:

```json
{
  env: {
    shellEnv: { enabled: true, timeoutMs: 15000 },
  },
}
```

Env var equivalent: `OPENCLAW_LOAD_SHELL_ENV=1`

 

Reference env vars in any config string value with `${VAR_NAME}`:

```json
{
  gateway: { auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" } },
  models: { providers: { custom: { apiKey: "${CUSTOM_API_KEY}" } } },
}
```

Rules:

-   Only uppercase names matched: `[A-Z_][A-Z0-9_]*`
-   Missing/empty vars throw an error at load time
-   Escape with `$${VAR}` for literal output
-   Works inside `$include` files
-   Inline substitution: `"${BASE}/v1"` → `"https://api.example.com/v1"`

 

For fields that support SecretRef objects, you can use:

```json
{
  models: {
    providers: {
      openai: { apiKey: { source: "env", provider: "default", id: "OPENAI_API_KEY" } },
    },
  },
  skills: {
    entries: {
      "nano-banana-pro": {
        apiKey: {
          source: "file",
          provider: "filemain",
          id: "/skills/entries/nano-banana-pro/apiKey",
        },
      },
    },
  },
  channels: {
    googlechat: {
      serviceAccountRef: {
        source: "exec",
        provider: "vault",
        id: "channels/googlechat/serviceAccount",
      },
    },
  },
}
```

SecretRef details (including `secrets.providers` for `env`/`file`/`exec`) are in [Secrets Management](./secrets.md). Supported credential paths are listed in [SecretRef Credential Surface](../reference/secretref-credential-surface.md).

 See [Environment](../help/environment.md) for full precedence and sources.

## Full reference

For the complete field-by-field reference, see **[Configuration Reference](./configuration-reference.md)**.

* * *

*Related: [Configuration Examples](./configuration-examples.md) · [Configuration Reference](./configuration-reference.md) · [Doctor](./doctor.md)*

[Gateway Runbook](../gateway.md)[Configuration Reference](./configuration-reference.md)
