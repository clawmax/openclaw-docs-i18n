

  Messaging platforms

  
# iMessage

> **⚠️** For new iMessage deployments, use [BlueBubbles](./bluebubbles.md).The `imsg` integration is legacy and may be removed in a future release.

 Status: legacy external CLI integration. Gateway spawns `imsg rpc` and communicates over JSON-RPC on stdio (no separate daemon/port). 

## Quick setup

## Requirements and permissions (macOS)

-   Messages must be signed in on the Mac running `imsg`.
-   Full Disk Access is required for the process context running OpenClaw/`imsg` (Messages DB access).
-   Automation permission is required to send messages through Messages.app.

> **💡** Permissions are granted per process context. If gateway runs headless (LaunchAgent/SSH), run a one-time interactive command in that same context to trigger prompts:
> 
> Copy
> 
> ```
> imsg chats --limit 1
> # or
> imsg send <handle> "test"
> ```

## Access control and routing

`channels.imessage.dmPolicy` controls direct messages:

-   `pairing` (default)
-   `allowlist`
-   `open` (requires `allowFrom` to include `"*"`)
-   `disabled`

Allowlist field: `channels.imessage.allowFrom`.Allowlist entries can be handles or chat targets (`chat_id:*`, `chat_guid:*`, `chat_identifier:*`).

`channels.imessage.groupPolicy` controls group handling:

-   `allowlist` (default when configured)
-   `open`
-   `disabled`

Group sender allowlist: `channels.imessage.groupAllowFrom`.Runtime fallback: if `groupAllowFrom` is unset, iMessage group sender checks fall back to `allowFrom` when available. Runtime note: if `channels.imessage` is completely missing, runtime falls back to `groupPolicy="allowlist"` and logs a warning (even if `channels.defaults.groupPolicy` is set).Mention gating for groups:

-   iMessage has no native mention metadata
-   mention detection uses regex patterns (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
-   with no configured patterns, mention gating cannot be enforced

Control commands from authorized senders can bypass mention gating in groups.

-   DMs use direct routing; groups use group routing.
-   With default `session.dmScope=main`, iMessage DMs collapse into the agent main session.
-   Group sessions are isolated (`agent::imessage:group:<chat_id>`).
-   Replies route back to iMessage using originating channel/target metadata.

Group-ish thread behavior:Some multi-participant iMessage threads can arrive with `is_group=false`. If that `chat_id` is explicitly configured under `channels.imessage.groups`, OpenClaw treats it as group traffic (group gating + group session isolation).

## Deployment patterns

Use a dedicated Apple ID and macOS user so bot traffic is isolated from your personal Messages profile.Typical flow:

1.  Create/sign in a dedicated macOS user.
2.  Sign into Messages with the bot Apple ID in that user.
3.  Install `imsg` in that user.
4.  Create SSH wrapper so OpenClaw can run `imsg` in that user context.
5.  Point `channels.imessage.accounts..cliPath` and `.dbPath` to that user profile.

First run may require GUI approvals (Automation + Full Disk Access) in that bot user session.

Common topology:

-   gateway runs on Linux/VM
-   iMessage + `imsg` runs on a Mac in your tailnet
-   `cliPath` wrapper uses SSH to run `imsg`
-   `remoteHost` enables SCP attachment fetches

Example:

```json
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db",
    },
  },
}
```

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

Use SSH keys so both SSH and SCP are non-interactive. Ensure the host key is trusted first (for example `ssh bot@mac-mini.tailnet-1234.ts.net`) so `known_hosts` is populated.

iMessage supports per-account config under `channels.imessage.accounts`.Each account can override fields such as `cliPath`, `dbPath`, `allowFrom`, `groupPolicy`, `mediaMaxMb`, history settings, and attachment root allowlists.

## Media, chunking, and delivery targets

-   inbound attachment ingestion is optional: `channels.imessage.includeAttachments`
-   remote attachment paths can be fetched via SCP when `remoteHost` is set
-   attachment paths must match allowed roots:
    -   `channels.imessage.attachmentRoots` (local)
    -   `channels.imessage.remoteAttachmentRoots` (remote SCP mode)
    -   default root pattern: `/Users/*/Library/Messages/Attachments`
-   SCP uses strict host-key checking (`StrictHostKeyChecking=yes`)
-   outbound media size uses `channels.imessage.mediaMaxMb` (default 16 MB)

-   text chunk limit: `channels.imessage.textChunkLimit` (default 4000)
-   chunk mode: `channels.imessage.chunkMode`
    -   `length` (default)
    -   `newline` (paragraph-first splitting)

Preferred explicit targets:

-   `chat_id:123` (recommended for stable routing)
-   `chat_guid:...`
-   `chat_identifier:...`

Handle targets are also supported:

-   `imessage:+1555...`
-   `sms:+1555...`
-   `user@example.com`

```bash
imsg chats --limit 20
```

## Config writes

iMessage allows channel-initiated config writes by default (for `/config set|unset` when `commands.config: true`). Disable:

```json
{
  channels: {
    imessage: {
      configWrites: false,
    },
  },
}
```

## Troubleshooting

Validate the binary and RPC support:

```bash
imsg rpc --help
openclaw channels status --probe
```

If probe reports RPC unsupported, update `imsg`.

Check:

-   `channels.imessage.dmPolicy`
-   `channels.imessage.allowFrom`
-   pairing approvals (`openclaw pairing list imessage`)

Check:

-   `channels.imessage.groupPolicy`
-   `channels.imessage.groupAllowFrom`
-   `channels.imessage.groups` allowlist behavior
-   mention pattern configuration (`agents.list[].groupChat.mentionPatterns`)

Check:

-   `channels.imessage.remoteHost`
-   `channels.imessage.remoteAttachmentRoots`
-   SSH/SCP key auth from the gateway host
-   host key exists in `~/.ssh/known_hosts` on the gateway host
-   remote path readability on the Mac running Messages

Re-run in an interactive GUI terminal in the same user/session context and approve prompts:

```bash
imsg chats --limit 1
imsg send <handle> "test"
```

Confirm Full Disk Access + Automation are granted for the process context that runs OpenClaw/`imsg`.

## Configuration reference pointers

-   [Configuration reference - iMessage](../gateway/configuration-reference.md#imessage)
-   [Gateway configuration](../gateway/configuration.md)
-   [Pairing](./pairing.md)
-   [BlueBubbles](./bluebubbles.md)

[Google Chat](./googlechat.md)[IRC](./irc.md)
