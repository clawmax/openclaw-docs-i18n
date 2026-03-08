

  CLI commands

  
# agent

Run an agent turn via the Gateway (use `--local` for embedded). Use `--agent ` to target a configured agent directly. Related:

-   Agent send tool: [Agent send](../tools/agent-send.md)

## Examples

```bash
openclaw agent --to +15555550123 --message "status update" --deliver
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```

## Notes

-   When this command triggers `models.json` regeneration, SecretRef-managed provider credentials are persisted as non-secret markers (for example env var names or `secretref-managed`), not resolved secret plaintext.

[acp](./acp.md)[agents](./agents.md)
