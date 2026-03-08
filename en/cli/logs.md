

  CLI commands

  
# logs

Tail Gateway file logs over RPC (works in remote mode). Related:

-   Logging overview: [Logging](../logging.md)

## Examples

```bash
openclaw logs
openclaw logs --follow
openclaw logs --json
openclaw logs --limit 500
openclaw logs --local-time
openclaw logs --follow --local-time
```

Use `--local-time` to render timestamps in your local timezone.

[hooks](./hooks.md)[memory](./memory.md)
