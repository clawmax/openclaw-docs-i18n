

  プロバイダー

  
# Litellm

[LiteLLM](https://litellm.ai) は、100以上のモデルプロバイダーに統一されたAPIを提供するオープンソースのLLMゲートウェイです。OpenClawをLiteLLM経由でルーティングすることで、一元化されたコスト追跡、ロギング、そしてOpenClawの設定を変更することなくバックエンドを切り替える柔軟性を得られます。

## OpenClawでLiteLLMを使用する理由

-   **コスト追跡** — OpenClawがすべてのモデルに費やした金額を正確に把握
-   **モデルルーティング** — Claude、GPT-4、Gemini、Bedrockなどを設定変更なしで切り替え
-   **仮想キー** — OpenClaw用の使用制限付きキーを作成
-   **ロギング** — デバッグのための完全なリクエスト/レスポンスログ
-   **フォールバック** — プライマリプロバイダーがダウンした場合の自動フェイルオーバー

## クイックスタート

### オンボーディング経由

```bash
openclaw onboard --auth-choice litellm-api-key
```

### 手動セットアップ

1.  LiteLLM Proxyを起動:

```bash
pip install 'litellm[proxy]'
litellm --model claude-opus-4-6
```

2.  OpenClawをLiteLLMに向ける:

```bash
export LITELLM_API_KEY="your-litellm-key"

openclaw
```

以上です。OpenClawはLiteLLM経由でルーティングされるようになりました。

## 設定

### 環境変数

```bash
export LITELLM_API_KEY="sk-litellm-key"
```

### 設定ファイル

```json
{
  models: {
    providers: {
      litellm: {
        baseUrl: "http://localhost:4000",
        apiKey: "${LITELLM_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "claude-opus-4-6",
            name: "Claude Opus 4.6",
            reasoning: true,
            input: ["text", "image"],
            contextWindow: 200000,
            maxTokens: 64000,
          },
          {
            id: "gpt-4o",
            name: "GPT-4o",
            reasoning: false,
            input: ["text", "image"],
            contextWindow: 128000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "litellm/claude-opus-4-6" },
    },
  },
}
```

## 仮想キー

使用制限付きのOpenClaw専用キーを作成:

```bash
curl -X POST "http://localhost:4000/key/generate" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "key_alias": "openclaw",
    "max_budget": 50.00,
    "budget_duration": "monthly"
  }'
```

生成されたキーを `LITELLM_API_KEY` として使用します。

## モデルルーティング

LiteLLMはモデルリクエストを異なるバックエンドにルーティングできます。LiteLLMの `config.yaml` で設定:

```yaml
model_list:
  - model_name: claude-opus-4-6
    litellm_params:
      model: claude-opus-4-6
      api_key: os.environ/ANTHROPIC_API_KEY

  - model_name: gpt-4o
    litellm_params:
      model: gpt-4o
      api_key: os.environ/OPENAI_API_KEY
```

OpenClawは `claude-opus-4-6` をリクエストし続けます — ルーティングはLiteLLMが処理します。

## 使用状況の確認

LiteLLMのダッシュボードまたはAPIで確認:

```bash
# キー情報
curl "http://localhost:4000/key/info" \
  -H "Authorization: Bearer sk-litellm-key"

# 使用量ログ
curl "http://localhost:4000/spend/logs" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY"
```

## 注意点

-   LiteLLMはデフォルトで `http://localhost:4000` で実行されます
-   OpenClawはOpenAI互換の `/v1/chat/completions` エンドポイント経由で接続します
-   OpenClawのすべての機能はLiteLLM経由で動作します — 制限はありません

## 関連項目

-   [LiteLLM ドキュメント](https://docs.litellm.ai)
-   [モデルプロバイダー](../concepts/model-providers.md)

[Kilocode](./kilocode.md)[GLM Models](./glm.md)