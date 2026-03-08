

  CLI commands

  
# tui

Open the terminal UI connected to the Gateway. Related:

-   TUI guide: [TUI](../web/tui.md)

Notes:

-   `tui` resolves configured gateway auth SecretRefs for token/password auth when possible (`env`/`file`/`exec` providers).

## Examples

```bash
openclaw tui
openclaw tui --url ws://127.0.0.1:18789 --token <token>
openclaw tui --session main --deliver
```

[system](./system.md)[uninstall](./uninstall.md)
