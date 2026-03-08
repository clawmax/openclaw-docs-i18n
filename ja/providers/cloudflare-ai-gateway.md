

  プロバイダー

  
# Cloudflare AI Gateway

Cloudflare AI Gateway はプロバイダー API の前に配置され、分析、キャッシュ、制御を追加できます。Anthropic の場合、OpenClaw はあなたの Gateway エンドポイントを通じて Anthropic Messages API を使用します。

-   プロバイダー: `cloudflare-ai-gateway`
-   ベース URL: `https://gateway.ai.cloudflare.com/v1/<account_id>/<gateway_id>/anthropic`
-   デフォルトモデル: `cloudflare-ai-gateway/claude-sonnet-4-5`
-   API キー: `CLOUDFLARE_AI_GATEWAY_API_KEY` (Gateway 経由のリクエスト用のプロバイダー API キー)

Anthropic モデルの場合は、あなたの Anthropic API キーを使用してください。

## クイックスタート

1.  プロバイダー API キーと Gateway の詳細を設定します:

```bash
openclaw onboard --auth-choice cloudflare-ai-gateway-api-key
```

2.  デフォルトモデルを設定します:

```json
{
  agents: {
    defaults: {
      model: { primary: "cloudflare-ai-gateway/claude-sonnet-4-5" },
    },
  },
}
```

## 非対話型の例

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice cloudflare-ai-gateway-api-key \
  --cloudflare-ai-gateway-account-id "your-account-id" \
  --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
  --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY"
```

## 認証付きゲートウェイ

Cloudflare で Gateway 認証を有効にした場合は、`cf-aig-authorization` ヘッダーを追加してください (これはプロバイダー API キーに加えて追加されます)。

```json
{
  models: {
    providers: {
      "cloudflare-ai-gateway": {
        headers: {
          "cf-aig-authorization": "Bearer <cloudflare-ai-gateway-token>",
        },
      },
    },
  },
}
```

## 環境に関する注意

Gateway がデーモン (launchd/systemd) として実行される場合、`CLOUDFLARE_AI_GATEWAY_API_KEY` がそのプロセスから利用可能であることを確認してください (例: `~/.openclaw/.env` または `env.shellEnv` 経由)。

[Amazon Bedrock](./bedrock.md)[Claude Max API Proxy](./claude-max-api-proxy.md)