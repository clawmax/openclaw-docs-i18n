

  プロバイダー

  
# OpenRouter

OpenRouter は、単一のエンドポイントと API キーの背後で多くのモデルにリクエストをルーティングする **統合 API** を提供します。OpenAI 互換であるため、ベース URL を切り替えるだけでほとんどの OpenAI SDK が動作します。

## CLI セットアップ

```bash
openclaw onboard --auth-choice apiKey --token-provider openrouter --token "$OPENROUTER_API_KEY"
```

## 設定スニペット

```json
{
  env: { OPENROUTER_API_KEY: "sk-or-..." },
  agents: {
    defaults: {
      model: { primary: "openrouter/anthropic/claude-sonnet-4-5" },
    },
  },
}
```

## 注意点

-   モデル参照は `openrouter/<プロバイダー>/<モデル>` の形式です。
-   より多くのモデル/プロバイダーオプションについては、[/concepts/model-providers](../concepts/model-providers.md) を参照してください。
-   OpenRouter は、内部で API キーを使用した Bearer トークンを利用します。

[OpenCode Zen](./opencode.md)[Qianfan](./qianfan.md)