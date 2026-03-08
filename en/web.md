

  Web interfaces

  
# Web

The Gateway serves a small **browser Control UI** (Vite + Lit) from the same port as the Gateway WebSocket:

-   default: `http://:18789/`
-   optional prefix: set `gateway.controlUi.basePath` (e.g. `/openclaw`)

Capabilities live in [Control UI](./web/control-ui.md). This page focuses on bind modes, security, and web-facing surfaces.

## Webhooks

When `hooks.enabled=true`, the Gateway also exposes a small webhook endpoint on the same HTTP server. See [Gateway configuration](./gateway/configuration.md) → `hooks` for auth + payloads.

## Config (default-on)

The Control UI is **enabled by default** when assets are present (`dist/control-ui`). You can control it via config:

```json
{
  gateway: {
    controlUi: { enabled: true, basePath: "/openclaw" }, // basePath optional
  },
}
```

## Tailscale access

### Integrated Serve (recommended)

Keep the Gateway on loopback and let Tailscale Serve proxy it:

```json
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" },
  },
}
```

Then start the gateway:

```bash
openclaw gateway
```

Open:

-   `https:///` (or your configured `gateway.controlUi.basePath`)

### Tailnet bind + token

```json
{
  gateway: {
    bind: "tailnet",
    controlUi: { enabled: true },
    auth: { mode: "token", token: "your-token" },
  },
}
```

Then start the gateway (token required for non-loopback binds):

```bash
openclaw gateway
```

Open:

-   `http://<tailscale-ip>:18789/` (or your configured `gateway.controlUi.basePath`)

### Public internet (Funnel)

```json
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password" }, // or OPENCLAW_GATEWAY_PASSWORD
  },
}
```

## Security notes

-   Gateway auth is required by default (token/password or Tailscale identity headers).
-   Non-loopback binds still **require** a shared token/password (`gateway.auth` or env).
-   The wizard generates a gateway token by default (even on loopback).
-   The UI sends `connect.params.auth.token` or `connect.params.auth.password`.
-   For non-loopback Control UI deployments, set `gateway.controlUi.allowedOrigins` explicitly (full origins). Without it, gateway startup is refused by default.
-   `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback=true` enables Host-header origin fallback mode, but is a dangerous security downgrade.
-   With Serve, Tailscale identity headers can satisfy Control UI/WebSocket auth when `gateway.auth.allowTailscale` is `true` (no token/password required). HTTP API endpoints still require token/password. Set `gateway.auth.allowTailscale: false` to require explicit credentials. See [Tailscale](./gateway/tailscale.md) and [Security](./gateway/security.md). This tokenless flow assumes the gateway host is trusted.
-   `gateway.tailscale.mode: "funnel"` requires `gateway.auth.mode: "password"` (shared password).

## Building the UI

The Gateway serves static files from `dist/control-ui`. Build them with:

```bash
pnpm ui:build # auto-installs UI deps on first run
```

[CONTRIBUTING THREAT MODEL](./security/CONTRIBUTING-THREAT-MODEL.md)[Control UI](./web/control-ui.md)
