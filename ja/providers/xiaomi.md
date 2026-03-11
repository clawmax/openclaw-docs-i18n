

  プロバイダー

  
# Xiaomi MiMo

Xiaomi MiMo は、**MiMo** モデルのための API プラットフォームです。OpenAI および Anthropic フォーマットと互換性のある REST API を提供し、認証には API キーを使用します。API キーは [Xiaomi MiMo コンソール](https://platform.xiaomimimo.com/#/console/api-keys) で作成してください。OpenClaw は Xiaomi MiMo API キーと共に `xiaomi` プロバイダーを使用します。

## モデル概要

-   **mimo-v2-flash**: 262144 トークンのコンテキストウィンドウ、Anthropic Messages API 互換。
-   ベース URL: `https://api.xiaomimimo.com/anthropic`
-   認証: `Bearer $XIAOMI_API_KEY`

## CLI セットアップ

```bash
openclaw onboard --auth-choice xiaomi-api-key
# または非対話型
openclaw onboard --auth-choice xiaomi-api-key --xiaomi-api-key "$XIAOMI_API_KEY"
```

## 設定スニペット

```json
{
  env: { XIAOMI_API_KEY: "your-key" },
  agents: { defaults: { model: { primary: "xiaomi/mimo-v2-flash" } } },
  models: {
    mode: "merge",
    providers: {
      xiaomi: {
        baseUrl: "https://api.xiaomimimo.com/anthropic",
        api: "anthropic-messages",
        apiKey: "XIAOMI_API_KEY",
        models: [
          {
            id: "mimo-v2-flash",
            name: "Xiaomi MiMo V2 Flash",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 262144,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## 注意事項

-   モデル参照: `xiaomi/mimo-v2-flash`。
-   `XIAOMI_API_KEY` が設定されている（または認証プロファイルが存在する）場合、プロバイダーは自動的に注入されます。
-   プロバイダーのルールについては、[/concepts/model-providers](../concepts/model-providers.md) を参照してください。

[vLLM](./vllm.md)[Z.AI](./zai.md)