

  CLI commands

  
# devices

Manage device pairing requests and device-scoped tokens.

## Commands

### openclaw devices list

List pending pairing requests and paired devices.

```bash
openclaw devices list
openclaw devices list --json
```

### openclaw devices remove &lt;deviceId&gt;

Remove one paired device entry.

```bash
openclaw devices remove <deviceId>
openclaw devices remove <deviceId> --json
```

### openclaw devices clear --yes \[--pending\]

Clear paired devices in bulk.

```bash
openclaw devices clear --yes
openclaw devices clear --yes --pending
openclaw devices clear --yes --pending --json
```

### openclaw devices approve \[requestId\] \[--latest\]

Approve a pending device pairing request. If `requestId` is omitted, OpenClaw automatically approves the most recent pending request.

```bash
openclaw devices approve
openclaw devices approve <requestId>
openclaw devices approve --latest
```

### openclaw devices reject &lt;requestId&gt;

Reject a pending device pairing request.

```bash
openclaw devices reject <requestId>
```

### openclaw devices rotate --device &lt;id&gt; --role &lt;role&gt; \[--scope &lt;scope...&gt;\]

Rotate a device token for a specific role (optionally updating scopes).

```bash
openclaw devices rotate --device <deviceId> --role operator --scope operator.read --scope operator.write
```

### openclaw devices revoke --device &lt;id&gt; --role &lt;role&gt;

Revoke a device token for a specific role.

```bash
openclaw devices revoke --device <deviceId> --role node
```

## Common options

-   `--url `: Gateway WebSocket URL (defaults to `gateway.remote.url` when configured).
-   `--token `: Gateway token (if required).
-   `--password `: Gateway password (password auth).
-   `--timeout `: RPC timeout.
-   `--json`: JSON output (recommended for scripting).

Note: when you set `--url`, the CLI does not fall back to config or environment credentials. Pass `--token` or `--password` explicitly. Missing explicit credentials is an error.

## Notes

-   Token rotation returns a new token (sensitive). Treat it like a secret.
-   These commands require `operator.pairing` (or `operator.admin`) scope.
-   `devices clear` is intentionally gated by `--yes`.
-   If pairing scope is unavailable on local loopback (and no explicit `--url` is passed), list/approve can use a local pairing fallback.

[dashboard](./dashboard.md)[directory](./directory.md)
