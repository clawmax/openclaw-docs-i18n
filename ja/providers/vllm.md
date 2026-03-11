

  プロバイダー

  
# vLLM

vLLM は、**OpenAI互換**のHTTP APIを介してオープンソース（および一部のカスタム）モデルを提供できます。OpenClaw は `openai-completions` API を使用して vLLM に接続できます。また、OpenClaw は `VLLM_API_KEY` を設定し（サーバーが認証を強制しない場合は任意の値で動作）、明示的な `models.providers.vllm` エントリを定義しない場合、vLLM から利用可能なモデルを**自動検出**することもできます。

## クイックスタート

1.  OpenAI互換サーバーで vLLM を起動します。

ベースURLは `/v1` エンドポイント（例: `/v1/models`, `/v1/chat/completions`）を公開している必要があります。vLLM は通常以下で実行されます:

-   `http://127.0.0.1:8000/v1`

2.  オプトイン（認証が設定されていない場合、任意の値で動作します）:

```bash
export VLLM_API_KEY="vllm-local"
```

3.  モデルを選択します（vLLM のモデルIDのいずれかに置き換えてください）:

```json
{
  agents: {
    defaults: {
      model: { primary: "vllm/your-model-id" },
    },
  },
}
```

## モデル検出（暗黙的プロバイダー）

`VLLM_API_KEY` が設定されている（または認証プロファイルが存在する）状態で、`models.providers.vllm` を**定義しない**場合、OpenClaw は以下をクエリします:

-   `GET http://127.0.0.1:8000/v1/models`

…そして返されたIDをモデルエントリに変換します。明示的に `models.providers.vllm` を設定すると、自動検出はスキップされ、モデルを手動で定義する必要があります。

## 明示的な構成（手動モデル）

以下の場合に明示的な構成を使用します:

-   vLLM が異なるホスト/ポートで実行されている。
-   `contextWindow`/`maxTokens` の値を固定したい。
-   サーバーが実際のAPIキーを必要とする（またはヘッダーを制御したい）。

```json
{
  models: {
    providers: {
      vllm: {
        baseUrl: "http://127.0.0.1:8000/v1",
        apiKey: "${VLLM_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "your-model-id",
            name: "ローカル vLLM モデル",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## トラブルシューティング

-   サーバーが到達可能か確認します:

```bash
curl http://127.0.0.1:8000/v1/models
```

-   認証エラーでリクエストが失敗する場合は、サーバー構成と一致する実際の `VLLM_API_KEY` を設定するか、`models.providers.vllm` の下でプロバイダーを明示的に構成してください。

[Venice AI](./venice.md)[Xiaomi MiMo](./xiaomi.md)