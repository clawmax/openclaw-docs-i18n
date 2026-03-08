

  Gateway

  
# Gateway Runbook

Use this page for day-1 startup and day-2 operations of the Gateway service. 

## 5-minute local startup

### Step 1: Start the Gateway

```bash
openclaw gateway --port 18789
# debug/trace mirrored to stdio
openclaw gateway --port 18789 --verbose
# force-kill listener on selected port, then start
openclaw gateway --force
```

### Step 2: Verify service health

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
```

Healthy baseline: `Runtime: running` and `RPC probe: ok`.

### Step 3: Validate channel readiness

```bash
openclaw channels status --probe
```

 

> **ℹ️** Gateway config reload watches the active config file path (resolved from profile/state defaults, or `OPENCLAW_CONFIG_PATH` when set). Default mode is `gateway.reload.mode="hybrid"`.

## Runtime model

-   One always-on process for routing, control plane, and channel connections.
-   Single multiplexed port for:
    -   WebSocket control/RPC
    -   HTTP APIs (OpenAI-compatible, Responses, tools invoke)
    -   Control UI and hooks
-   Default bind mode: `loopback`.
-   Auth is required by default (`gateway.auth.token` / `gateway.auth.password`, or `OPENCLAW_GATEWAY_TOKEN` / `OPENCLAW_GATEWAY_PASSWORD`).

### Port and bind precedence

| Setting | Resolution order |
| --- | --- |
| Gateway port | `--port` → `OPENCLAW_GATEWAY_PORT` → `gateway.port` → `18789` |
| Bind mode | CLI/override → `gateway.bind` → `loopback` |

### Hot reload modes

| `gateway.reload.mode` | Behavior |
| --- | --- |
| `off` | No config reload |
| `hot` | Apply only hot-safe changes |
| `restart` | Restart on reload-required changes |
| `hybrid` (default) | Hot-apply when safe, restart when required |

## Operator command set

```bash
openclaw gateway status
openclaw gateway status --deep
openclaw gateway status --json
openclaw gateway install
openclaw gateway restart
openclaw gateway stop
openclaw secrets reload
openclaw logs --follow
openclaw doctor
```

## Remote access

Preferred: Tailscale/VPN. Fallback: SSH tunnel.

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

Then connect clients to `ws://127.0.0.1:18789` locally. 

> **⚠️** If gateway auth is configured, clients still must send auth (`token`/`password`) even over SSH tunnels.

 See: [Remote Gateway](./gateway/remote.md), [Authentication](./gateway/authentication.md), [Tailscale](./gateway/tailscale.md).

## Supervision and service lifecycle

Use supervised runs for production-like reliability. 

].service\nopenclaw gateway status', lang: 'bash' }, { label: 'Linux (system service)', code: 'sudo systemctl daemon-reload\nsudo systemctl enable --now openclaw-gateway[-].service', lang: 'bash' }]} />

## Multiple gateways on one host

Most setups should run **one** Gateway. Use multiple only for strict isolation/redundancy (for example a rescue profile). Checklist per instance:

-   Unique `gateway.port`
-   Unique `OPENCLAW_CONFIG_PATH`
-   Unique `OPENCLAW_STATE_DIR`
-   Unique `agents.defaults.workspace`

Example:

```
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

See: [Multiple gateways](./gateway/multiple-gateways.md).

### Dev profile quick path

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
openclaw --dev status
```

Defaults include isolated state/config and base gateway port `19001`.

## Protocol quick reference (operator view)

-   First client frame must be `connect`.
-   Gateway returns `hello-ok` snapshot (`presence`, `health`, `stateVersion`, `uptimeMs`, limits/policy).
-   Requests: `req(method, params)` → `res(ok/payload|error)`.
-   Common events: `connect.challenge`, `agent`, `chat`, `presence`, `tick`, `health`, `heartbeat`, `shutdown`.

Agent runs are two-stage:

1.  Immediate accepted ack (`status:"accepted"`)
2.  Final completion response (`status:"ok"|"error"`), with streamed `agent` events in between.

See full protocol docs: [Gateway Protocol](./gateway/protocol.md).

## Operational checks

### Liveness

-   Open WS and send `connect`.
-   Expect `hello-ok` response with snapshot.

### Readiness

```bash
openclaw gateway status
openclaw channels status --probe
openclaw health
```

### Gap recovery

Events are not replayed. On sequence gaps, refresh state (`health`, `system-presence`) before continuing.

## Common failure signatures

| Signature | Likely issue |
| --- | --- |
| `refusing to bind gateway ... without auth` | Non-loopback bind without token/password |
| `another gateway instance is already listening` / `EADDRINUSE` | Port conflict |
| `Gateway start blocked: set gateway.mode=local` | Config set to remote mode |
| `unauthorized` during connect | Auth mismatch between client and gateway |

For full diagnosis ladders, use [Gateway Troubleshooting](./gateway/troubleshooting.md).

## Safety guarantees

-   Gateway protocol clients fail fast when Gateway is unavailable (no implicit direct-channel fallback).
-   Invalid/non-connect first frames are rejected and closed.
-   Graceful shutdown emits `shutdown` event before socket close.

* * *

Related:

-   [Troubleshooting](./gateway/troubleshooting.md)
-   [Background Process](./gateway/background-process.md)
-   [Configuration](./gateway/configuration.md)
-   [Health](./gateway/health.md)
-   [Doctor](./gateway/doctor.md)
-   [Authentication](./gateway/authentication.md)

[Configuration](./gateway/configuration.md)
