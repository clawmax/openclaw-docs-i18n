

  CLI commands

  
# health

Fetch health from the running Gateway.

```bash
openclaw health
openclaw health --json
openclaw health --verbose
```

Notes:

-   `--verbose` runs live probes and prints per-account timings when multiple accounts are configured.
-   Output includes per-agent session stores when multiple agents are configured.

[gateway](./gateway.md)[hooks](./hooks.md)
