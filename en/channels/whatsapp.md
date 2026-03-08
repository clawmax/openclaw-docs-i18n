

  Messaging platforms

  
# WhatsApp

Status: production-ready via WhatsApp Web (Baileys). Gateway owns linked session(s). 

## Quick setup

### Step 1: Configure WhatsApp access policy

```json
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing",
      allowFrom: ["+15551234567"],
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
  },
}
```

### Step 2: Link WhatsApp (QR)

```bash
openclaw channels login --channel whatsapp
```

For a specific account:

```bash
openclaw channels login --channel whatsapp --account work
```

### Step 3: Start the gateway

```bash
openclaw gateway
```

### Step 4: Approve first pairing request (if using pairing mode)

```bash
openclaw pairing list whatsapp
openclaw pairing approve whatsapp <CODE>
```

Pairing requests expire after 1 hour. Pending requests are capped at 3 per channel.

 

> **ℹ️** OpenClaw recommends running WhatsApp on a separate number when possible. (The channel metadata and onboarding flow are optimized for that setup, but personal-number setups are also supported.)

## Deployment patterns

This is the cleanest operational mode:

-   separate WhatsApp identity for OpenClaw
-   clearer DM allowlists and routing boundaries
-   lower chance of self-chat confusion

Minimal policy pattern:

```json
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"],
    },
  },
}
```

Onboarding supports personal-number mode and writes a self-chat-friendly baseline:

-   `dmPolicy: "allowlist"`
-   `allowFrom` includes your personal number
-   `selfChatMode: true`

In runtime, self-chat protections key off the linked self number and `allowFrom`.

The messaging platform channel is WhatsApp Web-based (`Baileys`) in current OpenClaw channel architecture.There is no separate Twilio WhatsApp messaging channel in the built-in chat-channel registry.

## Runtime model

-   Gateway owns the WhatsApp socket and reconnect loop.
-   Outbound sends require an active WhatsApp listener for the target account.
-   Status and broadcast chats are ignored (`@status`, `@broadcast`).
-   Direct chats use DM session rules (`session.dmScope`; default `main` collapses DMs to the agent main session).
-   Group sessions are isolated (`agent::whatsapp:group:`).

## Access control and activation

`channels.whatsapp.dmPolicy` controls direct chat access:

-   `pairing` (default)
-   `allowlist`
-   `open` (requires `allowFrom` to include `"*"`)
-   `disabled`

`allowFrom` accepts E.164-style numbers (normalized internally).Multi-account override: `channels.whatsapp.accounts..dmPolicy` (and `allowFrom`) take precedence over channel-level defaults for that account.Runtime behavior details:

-   pairings are persisted in channel allow-store and merged with configured `allowFrom`
-   if no allowlist is configured, the linked self number is allowed by default
-   outbound `fromMe` DMs are never auto-paired

Group access has two layers:

1.  **Group membership allowlist** (`channels.whatsapp.groups`)
    -   if `groups` is omitted, all groups are eligible
    -   if `groups` is present, it acts as a group allowlist (`"*"` allowed)
2.  **Group sender policy** (`channels.whatsapp.groupPolicy` + `groupAllowFrom`)
    -   `open`: sender allowlist bypassed
    -   `allowlist`: sender must match `groupAllowFrom` (or `*`)
    -   `disabled`: block all group inbound

Sender allowlist fallback:

-   if `groupAllowFrom` is unset, runtime falls back to `allowFrom` when available
-   sender allowlists are evaluated before mention/reply activation

Note: if no `channels.whatsapp` block exists at all, runtime group-policy fallback is `allowlist` (with a warning log), even if `channels.defaults.groupPolicy` is set.

Group replies require mention by default.Mention detection includes:

-   explicit WhatsApp mentions of the bot identity
-   configured mention regex patterns (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
-   implicit reply-to-bot detection (reply sender matches bot identity)

Security note:

-   quote/reply only satisfies mention gating; it does **not** grant sender authorization
-   with `groupPolicy: "allowlist"`, non-allowlisted senders are still blocked even if they reply to an allowlisted user’s message

Session-level activation command:

-   `/activation mention`
-   `/activation always`

`activation` updates session state (not global config). It is owner-gated.

## Personal-number and self-chat behavior

When the linked self number is also present in `allowFrom`, WhatsApp self-chat safeguards activate:

-   skip read receipts for self-chat turns
-   ignore mention-JID auto-trigger behavior that would otherwise ping yourself
-   if `messages.responsePrefix` is unset, self-chat replies default to `[{identity.name}]` or `[openclaw]`

## Message normalization and context

Incoming WhatsApp messages are wrapped in the shared inbound envelope.If a quoted reply exists, context is appended in this form:

```json
[Replying to <sender> id:<stanzaId>]
<quoted body or media placeholder>
[/Replying]
```

Reply metadata fields are also populated when available (`ReplyToId`, `ReplyToBody`, `ReplyToSender`, sender JID/E.164).

Media-only inbound messages are normalized with placeholders such as:

-   `<media:image>`
-   `<media:video>`
-   `<media:audio>`
-   `<media:document>`
-   `<media:sticker>`

Location and contact payloads are normalized into textual context before routing.

For groups, unprocessed messages can be buffered and injected as context when the bot is finally triggered.

-   default limit: `50`
-   config: `channels.whatsapp.historyLimit`
-   fallback: `messages.groupChat.historyLimit`
-   `0` disables

Injection markers:

-   `[Chat messages since your last reply - for context]`
-   `[Current message - respond to this]`

Read receipts are enabled by default for accepted inbound WhatsApp messages.Disable globally:

```json
{
  channels: {
    whatsapp: {
      sendReadReceipts: false,
    },
  },
}
```

Per-account override:

```json
{
  channels: {
    whatsapp: {
      accounts: {
        work: {
          sendReadReceipts: false,
        },
      },
    },
  },
}
```

Self-chat turns skip read receipts even when globally enabled.

## Delivery, chunking, and media

-   default chunk limit: `channels.whatsapp.textChunkLimit = 4000`
-   `channels.whatsapp.chunkMode = "length" | "newline"`
-   `newline` mode prefers paragraph boundaries (blank lines), then falls back to length-safe chunking

-   supports image, video, audio (PTT voice-note), and document payloads
-   `audio/ogg` is rewritten to `audio/ogg; codecs=opus` for voice-note compatibility
-   animated GIF playback is supported via `gifPlayback: true` on video sends
-   captions are applied to the first media item when sending multi-media reply payloads
-   media source can be HTTP(S), `file://`, or local paths

-   inbound media save cap: `channels.whatsapp.mediaMaxMb` (default `50`)
-   outbound media send cap: `channels.whatsapp.mediaMaxMb` (default `50`)
-   per-account overrides use `channels.whatsapp.accounts..mediaMaxMb`
-   images are auto-optimized (resize/quality sweep) to fit limits
-   on media send failure, first-item fallback sends text warning instead of dropping the response silently

## Acknowledgment reactions

WhatsApp supports immediate ack reactions on inbound receipt via `channels.whatsapp.ackReaction`.

```json
{
  channels: {
    whatsapp: {
      ackReaction: {
        emoji: "👀",
        direct: true,
        group: "mentions", // always | mentions | never
      },
    },
  },
}
```

Behavior notes:

-   sent immediately after inbound is accepted (pre-reply)
-   failures are logged but do not block normal reply delivery
-   group mode `mentions` reacts on mention-triggered turns; group activation `always` acts as bypass for this check
-   WhatsApp uses `channels.whatsapp.ackReaction` (legacy `messages.ackReaction` is not used here)

## Multi-account and credentials

-   account ids come from `channels.whatsapp.accounts`
-   default account selection: `default` if present, otherwise first configured account id (sorted)
-   account ids are normalized internally for lookup

-   current auth path: `~/.openclaw/credentials/whatsapp//creds.json`
-   backup file: `creds.json.bak`
-   legacy default auth in `~/.openclaw/credentials/` is still recognized/migrated for default-account flows

`openclaw channels logout --channel whatsapp [--account ]` clears WhatsApp auth state for that account.In legacy auth directories, `oauth.json` is preserved while Baileys auth files are removed.

## Tools, actions, and config writes

-   Agent tool support includes WhatsApp reaction action (`react`).
-   Action gates:
    -   `channels.whatsapp.actions.reactions`
    -   `channels.whatsapp.actions.polls`
-   Channel-initiated config writes are enabled by default (disable via `channels.whatsapp.configWrites=false`).

## Troubleshooting

Symptom: channel status reports not linked.Fix:

```bash
openclaw channels login --channel whatsapp
openclaw channels status
```

Symptom: linked account with repeated disconnects or reconnect attempts.Fix:

```bash
openclaw doctor
openclaw logs --follow
```

If needed, re-link with `channels login`.

Outbound sends fail fast when no active gateway listener exists for the target account.Make sure gateway is running and the account is linked.

Check in this order:

-   `groupPolicy`
-   `groupAllowFrom` / `allowFrom`
-   `groups` allowlist entries
-   mention gating (`requireMention` + mention patterns)
-   duplicate keys in `openclaw.json` (JSON5): later entries override earlier ones, so keep a single `groupPolicy` per scope

WhatsApp gateway runtime should use Node. Bun is flagged as incompatible for stable WhatsApp/Telegram gateway operation.

## Configuration reference pointers

Primary reference:

-   [Configuration reference - WhatsApp](../gateway/configuration-reference.md#whatsapp)

High-signal WhatsApp fields:

-   access: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`
-   delivery: `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `sendReadReceipts`, `ackReaction`
-   multi-account: `accounts..enabled`, `accounts..authDir`, account-level overrides
-   operations: `configWrites`, `debounceMs`, `web.enabled`, `web.heartbeatSeconds`, `web.reconnect.*`
-   session behavior: `session.dmScope`, `historyLimit`, `dmHistoryLimit`, `dms..historyLimit`

## Related

-   [Pairing](./pairing.md)
-   [Channel routing](./channel-routing.md)
-   [Multi-agent routing](../concepts/multi-agent.md)
-   [Troubleshooting](./troubleshooting.md)

[Twitch](./twitch.md)[Zalo](./zalo.md)
