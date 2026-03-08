

  CLI コマンド

  
# voicecall

`voicecall` はプラグインによって提供されるコマンドです。音声通話プラグインがインストールされ有効になっている場合にのみ表示されます。主なドキュメント:

-   音声通話プラグイン: [Voice Call](../plugins/voice-call.md)

## 一般的なコマンド

```bash
openclaw voicecall status --call-id <id>
openclaw voicecall call --to "+15555550123" --message "Hello" --mode notify
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall end --call-id <id>
```

## Webhookの公開 (Tailscale)

```bash
openclaw voicecall expose --mode serve
openclaw voicecall expose --mode funnel
openclaw voicecall expose --mode off
```

セキュリティ上の注意: Webhookエンドポイントは信頼できるネットワークにのみ公開してください。可能な場合は、FunnelよりもTailscale Serveを優先してください。

[update](./update.md)[webhooks](./webhooks.md)

---