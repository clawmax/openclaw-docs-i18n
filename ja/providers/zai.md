

  プロバイダー

  
# Z.AI

Z.AIは、**GLM**モデルのためのAPIプラットフォームです。GLMのREST APIを提供し、認証にはAPIキーを使用します。Z.AIコンソールでAPIキーを作成してください。OpenClawは、Z.AI APIキーと共に`zai`プロバイダーを使用します。

## CLIセットアップ

```bash
openclaw onboard --auth-choice zai-api-key
# または非対話型
openclaw onboard --zai-api-key "$ZAI_API_KEY"
```

## 設定スニペット

```json
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-5" } } },
}
```

## 注意事項

-   GLMモデルは `zai/<モデル名>` として利用可能です（例: `zai/glm-5`）。
-   Z.AIのツールコールストリーミングのために、`tool_stream`はデフォルトで有効です。無効にするには、`agents.defaults.models["zai/<モデル名>"].params.tool_stream` を `false` に設定してください。
-   モデルファミリーの概要については、[/providers/glm](./glm.md) を参照してください。
-   Z.AIは、あなたのAPIキーを使用したBearer認証を使用します。

[Xiaomi MiMo](./xiaomi.md)

---