

  プロバイダー

  
# Vercel AI Gateway

[Vercel AI Gateway](https://vercel.com/ai-gateway) は、単一のエンドポイントを通じて数百のモデルにアクセスするための統一 API を提供します。

-   プロバイダー: `vercel-ai-gateway`
-   認証: `AI_GATEWAY_API_KEY`
-   API: Anthropic Messages 互換

## クイックスタート

1.  API キーを設定します（推奨: Gateway 用に保存します）:

```bash
openclaw onboard --auth-choice ai-gateway-api-key
```

2.  デフォルトモデルを設定します:

```json
{
  agents: {
    defaults: {
      model: { primary: "vercel-ai-gateway/anthropic/claude-opus-4.6" },
    },
  },
}
```

## 非対話型の例

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY"
```

## 環境に関する注意

Gateway がデーモン (launchd/systemd) として実行される場合、`AI_GATEWAY_API_KEY` がそのプロセスで利用可能であることを確認してください（例: `~/.openclaw/.env` 内または `env.shellEnv` 経由）。

## モデル ID 短縮名

OpenClaw は Vercel Claude の短縮モデル参照を受け付け、実行時に正規化します:

-   `vercel-ai-gateway/claude-opus-4.6` -> `vercel-ai-gateway/anthropic/claude-opus-4.6`
-   `vercel-ai-gateway/opus-4.6` -> `vercel-ai-gateway/anthropic/claude-opus-4-6`

[Together](./together.md)[Venice AI](./venice.md)