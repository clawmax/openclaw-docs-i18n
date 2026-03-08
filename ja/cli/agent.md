

  CLI コマンド

  
# agent

ゲートウェイ経由でエージェントターンを実行します（組み込みの場合は `--local` を使用）。設定済みのエージェントを直接ターゲットにするには `--agent ` を使用します。関連項目:

-   エージェント送信ツール: [Agent send](../tools/agent-send.md)

## 例

```bash
openclaw agent --to +15555550123 --message "status update" --deliver
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```

## 注意事項

-   このコマンドが `models.json` の再生成をトリガーする場合、SecretRef で管理されるプロバイダー認証情報は、解決された秘密の平文ではなく、非秘密のマーカー（例: 環境変数名や `secretref-managed`）として保持されます。

[acp](./acp.md)[agents](./agents.md)

---