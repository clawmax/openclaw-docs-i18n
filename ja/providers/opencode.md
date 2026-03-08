

  プロバイダー

  
# OpenCode Zen

OpenCode Zenは、OpenCodeチームがコーディングエージェント向けに推奨する**厳選されたモデルリスト**です。これは、APIキーと`opencode`プロバイダーを使用するオプションのホスト型モデルアクセスパスです。Zenは現在ベータ版です。

## CLIセットアップ

```bash
openclaw onboard --auth-choice opencode-zen
# または非対話型
openclaw onboard --opencode-zen-api-key "$OPENCODE_API_KEY"
```

## 設定スニペット

```json
{
  env: { OPENCODE_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-6" } } },
}
```

## 注意事項

-   `OPENCODE_ZEN_API_KEY`もサポートされています。
-   Zenにサインインし、支払い情報を追加し、APIキーをコピーします。
-   OpenCode Zenはリクエストごとに課金されます。詳細はOpenCodeダッシュボードで確認してください。

[OpenAI](./openai.md)[OpenRouter](./openrouter.md)

---