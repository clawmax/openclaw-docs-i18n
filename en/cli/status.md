

  CLI commands

  
# status

Diagnostics for channels + sessions.

```bash
openclaw status
openclaw status --all
openclaw status --deep
openclaw status --usage
```

Notes:

-   `--deep` runs live probes (WhatsApp Web + Telegram + Discord + Google Chat + Slack + Signal).
-   Output includes per-agent session stores when multiple agents are configured.
-   Overview includes Gateway + node host service install/runtime status when available.
-   Overview includes update channel + git SHA (for source checkouts).
-   Update info surfaces in the Overview; if an update is available, status prints a hint to run `openclaw update` (see [Updating](../install/updating.md)).
-   Read-only status surfaces (`status`, `status --json`, `status --all`) resolve supported SecretRefs for their targeted config paths when possible.
-   If a supported channel SecretRef is configured but unavailable in the current command path, status stays read-only and reports degraded output instead of crashing. Human output shows warnings such as “configured token unavailable in this command path”, and JSON output includes `secretDiagnostics`.

[skills](./skills.md)[system](./system.md)
