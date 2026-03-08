

  Messaging platforms

  
# Telegram

Status: production-ready for bot DMs + groups via grammY. Long polling is the default mode; webhook mode is optional. 

## Quick setup

### Step 1: Create the bot token in BotFather

Open Telegram and chat with **@BotFather** (confirm the handle is exactly `@BotFather`).Run `/newbot`, follow prompts, and save the token.

### Step 2: Configure token and DM policy

```json
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```

Env fallback: `TELEGRAM_BOT_TOKEN=...` (default account only). Telegram does **not** use `openclaw channels login telegram`; configure token in config/env, then start gateway.

### Step 3: Start gateway and approve first DM

```bash
openclaw gateway
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

Pairing codes expire after 1 hour.

### Step 4: Add the bot to a group

Add the bot to your group, then set `channels.telegram.groups` and `groupPolicy` to match your access model.

 

> **ℹ️** Token resolution order is account-aware. In practice, config values win over env fallback, and `TELEGRAM_BOT_TOKEN` only applies to the default account.

## Telegram side settings

Telegram bots default to **Privacy Mode**, which limits what group messages they receive.If the bot must see all group messages, either:

-   disable privacy mode via `/setprivacy`, or
-   make the bot a group admin.

When toggling privacy mode, remove + re-add the bot in each group so Telegram applies the change.

Admin status is controlled in Telegram group settings.Admin bots receive all group messages, which is useful for always-on group behavior.

-   `/setjoingroups` to allow/deny group adds
-   `/setprivacy` for group visibility behavior

## Access control and activation

/getUpdates"', lang: 'bash' }, { label: 'Group policy and allowlists', code: '{\n  channels: {\n    telegram: {\n      groups: {\n        "-1001234567890": {\n          groupPolicy: "open",\n          requireMention: false,\n        },\n      },\n    },\n  },\n}', lang: 'json' }, { label: 'Mention behavior', code: '{\n  channels: {\n    telegram: {\n      groups: {\n        "*": { requireMention: false },\n      },\n    },\n  },\n}', lang: 'json' }]} />

## Runtime behavior

-   Telegram is owned by the gateway process.
-   Routing is deterministic: Telegram inbound replies back to Telegram (the model does not pick channels).
-   Inbound messages normalize into the shared channel envelope with reply metadata and media placeholders.
-   Group sessions are isolated by group ID. Forum topics append `:topic:` to keep topics isolated.
-   DM messages can carry `message_thread_id`; OpenClaw routes them with thread-aware session keys and preserves thread ID for replies.
-   Long polling uses grammY runner with per-chat/per-thread sequencing. Overall runner sink concurrency uses `agents.defaults.maxConcurrent`.
-   Telegram Bot API has no read-receipt support (`sendReadReceipts` does not apply).

## Feature reference

OpenClaw can stream partial replies in real time:

-   direct chats: Telegram native draft streaming via `sendMessageDraft`
-   groups/topics: preview message + `editMessageText`

Requirement:

-   `channels.telegram.streaming` is `off | partial | block | progress` (default: `partial`)
-   `progress` maps to `partial` on Telegram (compat with cross-channel naming)
-   legacy `channels.telegram.streamMode` and boolean `streaming` values are auto-mapped

Telegram enabled `sendMessageDraft` for all bots in Bot API 9.5 (March 1, 2026).For text-only replies:

-   DM: OpenClaw updates the draft in place (no extra preview message)
-   group/topic: OpenClaw keeps the same preview message and performs a final edit in place (no second message)

For complex replies (for example media payloads), OpenClaw falls back to normal final delivery and then cleans up the preview message.Preview streaming is separate from block streaming. When block streaming is explicitly enabled for Telegram, OpenClaw skips the preview stream to avoid double-streaming.If native draft transport is unavailable/rejected, OpenClaw automatically falls back to `sendMessage` + `editMessageText`.Telegram-only reasoning stream:

-   `/reasoning stream` sends reasoning to the live preview while generating
-   final answer is sent without reasoning text

Outbound text uses Telegram `parse_mode: "HTML"`.

-   Markdown-ish text is rendered to Telegram-safe HTML.
-   Raw model HTML is escaped to reduce Telegram parse failures.
-   If Telegram rejects parsed HTML, OpenClaw retries as plain text.

Link previews are enabled by default and can be disabled with `channels.telegram.linkPreview: false`.

Telegram command menu registration is handled at startup with `setMyCommands`.Native command defaults:

-   `commands.native: "auto"` enables native commands for Telegram

Add custom command menu entries:

```json
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
    },
  },
}
```

Rules:

-   names are normalized (strip leading `/`, lowercase)
-   valid pattern: `a-z`, `0-9`, `_`, length `1..32`
-   custom commands cannot override native commands
-   conflicts/duplicates are skipped and logged

Notes:

-   custom commands are menu entries only; they do not auto-implement behavior
-   plugin/skill commands can still work when typed even if not shown in Telegram menu

If native commands are disabled, built-ins are removed. Custom/plugin commands may still register if configured.Common setup failure:

-   `setMyCommands failed` usually means outbound DNS/HTTPS to `api.telegram.org` is blocked.

### Device pairing commands (device-pair plugin)

When the `device-pair` plugin is installed:

1.  `/pair` generates setup code
2.  paste code in iOS app
3.  `/pair approve` approves latest pending request

More details: [Pairing](./pairing.md#pair-via-telegram-recommended-for-ios).

Configure inline keyboard scope:

```json
{
  channels: {
    telegram: {
      capabilities: {
        inlineButtons: "allowlist",
      },
    },
  },
}
```

Per-account override:

```json
{
  channels: {
    telegram: {
      accounts: {
        main: {
          capabilities: {
            inlineButtons: "allowlist",
          },
        },
      },
    },
  },
}
```

Scopes:

-   `off`
-   `dm`
-   `group`
-   `all`
-   `allowlist` (default)

Legacy `capabilities: ["inlineButtons"]` maps to `inlineButtons: "all"`.Message action example:

```json
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  message: "Choose an option:",
  buttons: [
    [
      { text: "Yes", callback_data: "yes" },
      { text: "No", callback_data: "no" },
    ],
    [{ text: "Cancel", callback_data: "cancel" }],
  ],
}
```

Callback clicks are passed to the agent as text: `callback_data: `

Telegram tool actions include:

-   `sendMessage` (`to`, `content`, optional `mediaUrl`, `replyToMessageId`, `messageThreadId`)
-   `react` (`chatId`, `messageId`, `emoji`)
-   `deleteMessage` (`chatId`, `messageId`)
-   `editMessage` (`chatId`, `messageId`, `content`)
-   `createForumTopic` (`chatId`, `name`, optional `iconColor`, `iconCustomEmojiId`)

Channel message actions expose ergonomic aliases (`send`, `react`, `delete`, `edit`, `sticker`, `sticker-search`, `topic-create`).Gating controls:

-   `channels.telegram.actions.sendMessage`
-   `channels.telegram.actions.deleteMessage`
-   `channels.telegram.actions.reactions`
-   `channels.telegram.actions.sticker` (default: disabled)

Note: `edit` and `topic-create` are currently enabled by default and do not have separate `channels.telegram.actions.*` toggles.Reaction removal semantics: [/tools/reactions](../tools/reactions.md)

Telegram supports explicit reply threading tags in generated output:

-   `[[reply_to_current]]` replies to the triggering message
-   `[[reply_to:]]` replies to a specific Telegram message ID

`channels.telegram.replyToMode` controls handling:

-   `off` (default)
-   `first`
-   `all`

Note: `off` disables implicit reply threading. Explicit `[[reply_to_*]]` tags are still honored.

Forum supergroups:

-   topic session keys append `:topic:`
-   replies and typing target the topic thread
-   topic config path: `channels.telegram.groups..topics.`

General topic (`threadId=1`) special-case:

-   message sends omit `message_thread_id` (Telegram rejects `sendMessage(...thread_id=1)`)
-   typing actions still include `message_thread_id`

Topic inheritance: topic entries inherit group settings unless overridden (`requireMention`, `allowFrom`, `skills`, `systemPrompt`, `enabled`, `groupPolicy`). `agentId` is topic-only and does not inherit from group defaults.**Per-topic agent routing**: Each topic can route to a different agent by setting `agentId` in the topic config. This gives each topic its own isolated workspace, memory, and session. Example:

```json
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          topics: {
            "1": { agentId: "main" },      // General topic → main agent
            "3": { agentId: "zu" },        // Dev topic → zu agent
            "5": { agentId: "coder" }      // Code review → coder agent
          }
        }
      }
    }
  }
}
```

Each topic then has its own session key: `agent:zu:telegram:group:-1001234567890:topic:3`**Persistent ACP topic binding**: Forum topics can pin ACP harness sessions through top-level typed ACP bindings:

-   `bindings[]` with `type: "acp"` and `match.channel: "telegram"`

Example:

```json
{
  agents: {
    list: [
      {
        id: "codex",
        runtime: {
          type: "acp",
          acp: {
            agent: "codex",
            backend: "acpx",
            mode: "persistent",
            cwd: "/workspace/openclaw",
          },
        },
      },
    ],
  },
  bindings: [
    {
      type: "acp",
      agentId: "codex",
      match: {
        channel: "telegram",
        accountId: "default",
        peer: { kind: "group", id: "-1001234567890:topic:42" },
      },
    },
  ],
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          topics: {
            "42": {
              requireMention: false,
            },
          },
        },
      },
    },
  },
}
```

This is currently scoped to forum topics in groups and supergroups.**Thread-bound ACP spawn from chat**:

-   `/acp spawn  --thread here|auto` can bind the current Telegram topic to a new ACP session.
-   Follow-up topic messages route to the bound ACP session directly (no `/acp steer` required).
-   OpenClaw pins the spawn confirmation message in-topic after a successful bind.
-   Requires `channels.telegram.threadBindings.spawnAcpSessions=true`.

Template context includes:

-   `MessageThreadId`
-   `IsForum`

DM thread behavior:

-   private chats with `message_thread_id` keep DM routing but use thread-aware session keys/reply targets.

### Audio messages

Telegram distinguishes voice notes vs audio files.

-   default: audio file behavior
-   tag `[[audio_as_voice]]` in agent reply to force voice-note send

Message action example:

```json
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

### Video messages

Telegram distinguishes video files vs video notes.Message action example:

```json
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

Video notes do not support captions; provided message text is sent separately.

### Stickers

Inbound sticker handling:

-   static WEBP: downloaded and processed (placeholder `<media:sticker>`)
-   animated TGS: skipped
-   video WEBM: skipped

Sticker context fields:

-   `Sticker.emoji`
-   `Sticker.setName`
-   `Sticker.fileId`
-   `Sticker.fileUniqueId`
-   `Sticker.cachedDescription`

Sticker cache file:

-   `~/.openclaw/telegram/sticker-cache.json`

Stickers are described once (when possible) and cached to reduce repeated vision calls.Enable sticker actions:

```json
{
  channels: {
    telegram: {
      actions: {
        sticker: true,
      },
    },
  },
}
```

Send sticker action:

```json
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

Search cached stickers:

```json
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

Telegram reactions arrive as `message_reaction` updates (separate from message payloads).When enabled, OpenClaw enqueues system events like:

-   `Telegram reaction added: 👍 by Alice (@alice) on msg 42`

Config:

-   `channels.telegram.reactionNotifications`: `off | own | all` (default: `own`)
-   `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` (default: `minimal`)

Notes:

-   `own` means user reactions to bot-sent messages only (best-effort via sent-message cache).
-   Reaction events still respect Telegram access controls (`dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`); unauthorized senders are dropped.
-   Telegram does not provide thread IDs in reaction updates.
    -   non-forum groups route to group chat session
    -   forum groups route to the group general-topic session (`:topic:1`), not the exact originating topic

`allowed_updates` for polling/webhook include `message_reaction` automatically.

`ackReaction` sends an acknowledgement emoji while OpenClaw is processing an inbound message.Resolution order:

-   `channels.telegram.accounts..ackReaction`
-   `channels.telegram.ackReaction`
-   `messages.ackReaction`
-   agent identity emoji fallback (`agents.list[].identity.emoji`, else ”👀”)

Notes:

-   Telegram expects unicode emoji (for example ”👀”).
-   Use `""` to disable the reaction for a channel or account.

Channel config writes are enabled by default (`configWrites !== false`).Telegram-triggered writes include:

-   group migration events (`migrate_to_chat_id`) to update `channels.telegram.groups`
-   `/config set` and `/config unset` (requires command enablement)

Disable:

```json
{
  channels: {
    telegram: {
      configWrites: false,
    },
  },
}
```

Default: long polling.Webhook mode:

-   set `channels.telegram.webhookUrl`
-   set `channels.telegram.webhookSecret` (required when webhook URL is set)
-   optional `channels.telegram.webhookPath` (default `/telegram-webhook`)
-   optional `channels.telegram.webhookHost` (default `127.0.0.1`)
-   optional `channels.telegram.webhookPort` (default `8787`)

Default local listener for webhook mode binds to `127.0.0.1:8787`.If your public endpoint differs, place a reverse proxy in front and point `webhookUrl` at the public URL. Set `webhookHost` (for example `0.0.0.0`) when you intentionally need external ingress.

-   `channels.telegram.textChunkLimit` default is 4000.
-   `channels.telegram.chunkMode="newline"` prefers paragraph boundaries (blank lines) before length splitting.
-   `channels.telegram.mediaMaxMb` (default 100) caps inbound and outbound Telegram media size.
-   `channels.telegram.timeoutSeconds` overrides Telegram API client timeout (if unset, grammY default applies).
-   group context history uses `channels.telegram.historyLimit` or `messages.groupChat.historyLimit` (default 50); `0` disables.
-   DM history controls:
    -   `channels.telegram.dmHistoryLimit`
    -   `channels.telegram.dms["<user_id>"].historyLimit`
-   `channels.telegram.retry` config applies to Telegram send helpers (CLI/tools/actions) for recoverable outbound API errors.

CLI send target can be numeric chat ID or username:

```bash
openclaw message send --channel telegram --target 123456789 --message "hi"
openclaw message send --channel telegram --target @name --message "hi"
```

Telegram polls use `openclaw message poll` and support forum topics:

```bash
openclaw message poll --channel telegram --target 123456789 \
  --poll-question "Ship it?" --poll-option "Yes" --poll-option "No"
openclaw message poll --channel telegram --target -1001234567890:topic:42 \
  --poll-question "Pick a time" --poll-option "10am" --poll-option "2pm" \
  --poll-duration-seconds 300 --poll-public
```

Telegram-only poll flags:

-   `--poll-duration-seconds` (5-600)
-   `--poll-anonymous`
-   `--poll-public`
-   `--thread-id` for forum topics (or use a `:topic:` target)

Action gating:

-   `channels.telegram.actions.sendMessage=false` disables outbound Telegram messages, including polls
-   `channels.telegram.actions.poll=false` disables Telegram poll creation while leaving regular sends enabled

## Troubleshooting

-   If `requireMention=false`, Telegram privacy mode must allow full visibility.
    -   BotFather: `/setprivacy` -> Disable
    -   then remove + re-add bot to group
-   `openclaw channels status` warns when config expects unmentioned group messages.
-   `openclaw channels status --probe` can check explicit numeric group IDs; wildcard `"*"` cannot be membership-probed.
-   quick session test: `/activation always`.

-   when `channels.telegram.groups` exists, group must be listed (or include `"*"`)
-   verify bot membership in group
-   review logs: `openclaw logs --follow` for skip reasons

-   authorize your sender identity (pairing and/or numeric `allowFrom`)
-   command authorization still applies even when group policy is `open`
-   `setMyCommands failed` usually indicates DNS/HTTPS reachability issues to `api.telegram.org`

-   Node 22+ + custom fetch/proxy can trigger immediate abort behavior if AbortSignal types mismatch.
-   Some hosts resolve `api.telegram.org` to IPv6 first; broken IPv6 egress can cause intermittent Telegram API failures.
-   If logs include `TypeError: fetch failed` or `Network request for 'getUpdates' failed!`, OpenClaw now retries these as recoverable network errors.
-   On VPS hosts with unstable direct egress/TLS, route Telegram API calls through `channels.telegram.proxy`:

```yaml
channels:
  telegram:
    proxy: socks5://<user>:<password>@proxy-host:1080
```

-   Node 22+ defaults to `autoSelectFamily=true` (except WSL2) and `dnsResultOrder=ipv4first`.
-   If your host is WSL2 or explicitly works better with IPv4-only behavior, force family selection:

```yaml
channels:
  telegram:
    network:
      autoSelectFamily: false
```

-   Environment overrides (temporary):
    -   `OPENCLAW_TELEGRAM_DISABLE_AUTO_SELECT_FAMILY=1`
    -   `OPENCLAW_TELEGRAM_ENABLE_AUTO_SELECT_FAMILY=1`
    -   `OPENCLAW_TELEGRAM_DNS_RESULT_ORDER=ipv4first`
-   Validate DNS answers:

```bash
dig +short api.telegram.org A
dig +short api.telegram.org AAAA
```

 More help: [Channel troubleshooting](./troubleshooting.md).

## Telegram config reference pointers

Primary reference:

-   `channels.telegram.enabled`: enable/disable channel startup.
-   `channels.telegram.botToken`: bot token (BotFather).
-   `channels.telegram.tokenFile`: read token from file path.
-   `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (default: pairing).
-   `channels.telegram.allowFrom`: DM allowlist (numeric Telegram user IDs). `allowlist` requires at least one sender ID. `open` requires `"*"`. `openclaw doctor --fix` can resolve legacy `@username` entries to IDs and can recover allowlist entries from pairing-store files in allowlist migration flows.
-   `channels.telegram.actions.poll`: enable or disable Telegram poll creation (default: enabled; still requires `sendMessage`).
-   `channels.telegram.defaultTo`: default Telegram target used by CLI `--deliver` when no explicit `--reply-to` is provided.
-   `channels.telegram.groupPolicy`: `open | allowlist | disabled` (default: allowlist).
-   `channels.telegram.groupAllowFrom`: group sender allowlist (numeric Telegram user IDs). `openclaw doctor --fix` can resolve legacy `@username` entries to IDs. Non-numeric entries are ignored at auth time. Group auth does not use DM pairing-store fallback (`2026.2.25+`).
-   Multi-account precedence:
    -   When two or more account IDs are configured, set `channels.telegram.defaultAccount` (or include `channels.telegram.accounts.default`) to make default routing explicit.
    -   If neither is set, OpenClaw falls back to the first normalized account ID and `openclaw doctor` warns.
    -   `channels.telegram.accounts.default.allowFrom` and `channels.telegram.accounts.default.groupAllowFrom` apply only to the `default` account.
    -   Named accounts inherit `channels.telegram.allowFrom` and `channels.telegram.groupAllowFrom` when account-level values are unset.
    -   Named accounts do not inherit `channels.telegram.accounts.default.allowFrom` / `groupAllowFrom`.
-   `channels.telegram.groups`: per-group defaults + allowlist (use `"*"` for global defaults).
    -   `channels.telegram.groups..groupPolicy`: per-group override for groupPolicy (`open | allowlist | disabled`).
    -   `channels.telegram.groups..requireMention`: mention gating default.
    -   `channels.telegram.groups..skills`: skill filter (omit = all skills, empty = none).
    -   `channels.telegram.groups..allowFrom`: per-group sender allowlist override.
    -   `channels.telegram.groups..systemPrompt`: extra system prompt for the group.
    -   `channels.telegram.groups..enabled`: disable the group when `false`.
    -   `channels.telegram.groups..topics..*`: per-topic overrides (group fields + topic-only `agentId`).
    -   `channels.telegram.groups..topics..agentId`: route this topic to a specific agent (overrides group-level and binding routing).
    -   `channels.telegram.groups..topics..groupPolicy`: per-topic override for groupPolicy (`open | allowlist | disabled`).
    -   `channels.telegram.groups..topics..requireMention`: per-topic mention gating override.
    -   top-level `bindings[]` with `type: "acp"` and canonical topic id `chatId:topic:topicId` in `match.peer.id`: persistent ACP topic binding fields (see [ACP Agents](../tools/acp-agents.md#channel-specific-settings)).
    -   `channels.telegram.direct..topics..agentId`: route DM topics to a specific agent (same behavior as forum topics).
-   `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist` (default: allowlist).
-   `channels.telegram.accounts..capabilities.inlineButtons`: per-account override.
-   `channels.telegram.commands.nativeSkills`: enable/disable Telegram native skills commands.
-   `channels.telegram.replyToMode`: `off | first | all` (default: `off`).
-   `channels.telegram.textChunkLimit`: outbound chunk size (chars).
-   `channels.telegram.chunkMode`: `length` (default) or `newline` to split on blank lines (paragraph boundaries) before length chunking.
-   `channels.telegram.linkPreview`: toggle link previews for outbound messages (default: true).
-   `channels.telegram.streaming`: `off | partial | block | progress` (live stream preview; default: `partial`; `progress` maps to `partial`; `block` is legacy preview mode compatibility). In DMs, `partial` uses native `sendMessageDraft` when available.
-   `channels.telegram.mediaMaxMb`: inbound/outbound Telegram media cap (MB, default: 100).
-   `channels.telegram.retry`: retry policy for Telegram send helpers (CLI/tools/actions) on recoverable outbound API errors (attempts, minDelayMs, maxDelayMs, jitter).
-   `channels.telegram.network.autoSelectFamily`: override Node autoSelectFamily (true=enable, false=disable). Defaults to enabled on Node 22+, with WSL2 defaulting to disabled.
-   `channels.telegram.network.dnsResultOrder`: override DNS result order (`ipv4first` or `verbatim`). Defaults to `ipv4first` on Node 22+.
-   `channels.telegram.proxy`: proxy URL for Bot API calls (SOCKS/HTTP).
-   `channels.telegram.webhookUrl`: enable webhook mode (requires `channels.telegram.webhookSecret`).
-   `channels.telegram.webhookSecret`: webhook secret (required when webhookUrl is set).
-   `channels.telegram.webhookPath`: local webhook path (default `/telegram-webhook`).
-   `channels.telegram.webhookHost`: local webhook bind host (default `127.0.0.1`).
-   `channels.telegram.webhookPort`: local webhook bind port (default `8787`).
-   `channels.telegram.actions.reactions`: gate Telegram tool reactions.
-   `channels.telegram.actions.sendMessage`: gate Telegram tool message sends.
-   `channels.telegram.actions.deleteMessage`: gate Telegram tool message deletes.
-   `channels.telegram.actions.sticker`: gate Telegram sticker actions — send and search (default: false).
-   `channels.telegram.reactionNotifications`: `off | own | all` — control which reactions trigger system events (default: `own` when not set).
-   `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — control agent’s reaction capability (default: `minimal` when not set).
-   [Configuration reference - Telegram](../gateway/configuration-reference.md#telegram)

Telegram-specific high-signal fields:

-   startup/auth: `enabled`, `botToken`, `tokenFile`, `accounts.*`
-   access control: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`, `groups.*.topics.*`, top-level `bindings[]` (`type: "acp"`)
-   command/menu: `commands.native`, `commands.nativeSkills`, `customCommands`
-   threading/replies: `replyToMode`
-   streaming: `streaming` (preview), `blockStreaming`
-   formatting/delivery: `textChunkLimit`, `chunkMode`, `linkPreview`, `responsePrefix`
-   media/network: `mediaMaxMb`, `timeoutSeconds`, `retry`, `network.autoSelectFamily`, `proxy`
-   webhook: `webhookUrl`, `webhookSecret`, `webhookPath`, `webhookHost`
-   actions/capabilities: `capabilities.inlineButtons`, `actions.sendMessage|editMessage|deleteMessage|reactions|sticker`
-   reactions: `reactionNotifications`, `reactionLevel`
-   writes/history: `configWrites`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`

## Related

-   [Pairing](./pairing.md)
-   [Channel routing](./channel-routing.md)
-   [Multi-agent routing](../concepts/multi-agent.md)
-   [Troubleshooting](./troubleshooting.md)

[Slack](./slack.md)[Tlon](./tlon.md)
