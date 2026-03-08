

  CLI commands

  
# pairing

Approve or inspect DM pairing requests (for channels that support pairing). Related:

-   Pairing flow: [Pairing](../channels/pairing.md)

## Commands

```bash
openclaw pairing list telegram
openclaw pairing list --channel telegram --account work
openclaw pairing list telegram --json

openclaw pairing approve telegram <code>
openclaw pairing approve --channel telegram --account work <code> --notify
```

## Notes

-   Channel input: pass it positionally (`pairing list telegram`) or with `--channel `.
-   `pairing list` supports `--account ` for multi-account channels.
-   `pairing approve` supports `--account ` and `--notify`.
-   If only one pairing-capable channel is configured, `pairing approve ` is allowed.

[onboard](./onboard.md)[plugins](./plugins.md)
