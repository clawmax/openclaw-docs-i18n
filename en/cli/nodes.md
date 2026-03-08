

  CLI commands

  
# nodes

Manage paired nodes (devices) and invoke node capabilities. Related:

-   Nodes overview: [Nodes](../nodes.md)
-   Camera: [Camera nodes](../nodes/camera.md)
-   Images: [Image nodes](../nodes/images.md)

Common options:

-   `--url`, `--token`, `--timeout`, `--json`

## Common commands

```bash
openclaw nodes list
openclaw nodes list --connected
openclaw nodes list --last-connected 24h
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes status
openclaw nodes status --connected
openclaw nodes status --last-connected 24h
```

`nodes list` prints pending/paired tables. Paired rows include the most recent connect age (Last Connect). Use `--connected` to only show currently-connected nodes. Use `--last-connected ` to filter to nodes that connected within a duration (e.g. `24h`, `7d`).

## Invoke / run

```bash
openclaw nodes invoke --node <id|name|ip> --command <command> --params <json>
openclaw nodes run --node <id|name|ip> <command...>
openclaw nodes run --raw "git status"
openclaw nodes run --agent main --node <id|name|ip> --raw "git status"
```

Invoke flags:

-   `--params `: JSON object string (default `{}`).
-   `--invoke-timeout `: node invoke timeout (default `15000`).
-   `--idempotency-key `: optional idempotency key.

### Exec-style defaults

`nodes run` mirrors the model’s exec behavior (defaults + approvals):

-   Reads `tools.exec.*` (plus `agents.list[].tools.exec.*` overrides).
-   Uses exec approvals (`exec.approval.request`) before invoking `system.run`.
-   `--node` can be omitted when `tools.exec.node` is set.
-   Requires a node that advertises `system.run` (macOS companion app or headless node host).

Flags:

-   `--cwd `: working directory.
-   `--env <key=val>`: env override (repeatable). Note: node hosts ignore `PATH` overrides (and `tools.exec.pathPrepend` is not applied to node hosts).
-   `--command-timeout `: command timeout.
-   `--invoke-timeout `: node invoke timeout (default `30000`).
-   `--needs-screen-recording`: require screen recording permission.
-   `--raw `: run a shell string (`/bin/sh -lc` or `cmd.exe /c`). In allowlist mode on Windows node hosts, `cmd.exe /c` shell-wrapper runs require approval (allowlist entry alone does not auto-allow the wrapper form).
-   `--agent `: agent-scoped approvals/allowlists (defaults to configured agent).
-   `--ask <off|on-miss|always>`, `--security <deny|allowlist|full>`: overrides.

[node](./node.md)[onboard](./onboard.md)
