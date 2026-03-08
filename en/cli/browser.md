

  CLI commands

  
# browser

Manage OpenClaw’s browser control server and run browser actions (tabs, snapshots, screenshots, navigation, clicks, typing). Related:

-   Browser tool + API: [Browser tool](../tools/browser.md)
-   Chrome extension relay: [Chrome extension](../tools/chrome-extension.md)

## Common flags

-   `--url `: Gateway WebSocket URL (defaults to config).
-   `--token `: Gateway token (if required).
-   `--timeout `: request timeout (ms).
-   `--browser-profile `: choose a browser profile (default from config).
-   `--json`: machine-readable output (where supported).

## Quick start (local)

```bash
openclaw browser --browser-profile chrome tabs
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

## Profiles

Profiles are named browser routing configs. In practice:

-   `openclaw`: launches/attaches to a dedicated OpenClaw-managed Chrome instance (isolated user data dir).
-   `chrome`: controls your existing Chrome tab(s) via the Chrome extension relay.

```bash
openclaw browser profiles
openclaw browser create-profile --name work --color "#FF5A36"
openclaw browser delete-profile --name work
```

Use a specific profile:

```bash
openclaw browser --browser-profile work tabs
```

## Tabs

```bash
openclaw browser tabs
openclaw browser open https://docs.openclaw.ai
openclaw browser focus <targetId>
openclaw browser close <targetId>
```

## Snapshot / screenshot / actions

Snapshot:

```bash
openclaw browser snapshot
```

Screenshot:

```bash
openclaw browser screenshot
```

Navigate/click/type (ref-based UI automation):

```bash
openclaw browser navigate https://example.com
openclaw browser click <ref>
openclaw browser type <ref> "hello"
```

## Chrome extension relay (attach via toolbar button)

This mode lets the agent control an existing Chrome tab that you attach manually (it does not auto-attach). Install the unpacked extension to a stable path:

```bash
openclaw browser extension install
openclaw browser extension path
```

Then Chrome → `chrome://extensions` → enable “Developer mode” → “Load unpacked” → select the printed folder. Full guide: [Chrome extension](../tools/chrome-extension.md)

## Remote browser control (node host proxy)

If the Gateway runs on a different machine than the browser, run a **node host** on the machine that has Chrome/Brave/Edge/Chromium. The Gateway will proxy browser actions to that node (no separate browser control server required). Use `gateway.nodes.browser.mode` to control auto-routing and `gateway.nodes.browser.node` to pin a specific node if multiple are connected. Security + remote setup: [Browser tool](../tools/browser.md), [Remote access](../gateway/remote.md), [Tailscale](../gateway/tailscale.md), [Security](../gateway/security.md)

[approvals](./approvals.md)[channels](./channels.md)
