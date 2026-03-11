

  プロトコルと API

  
# OpenAI Chat Completions

OpenClaw の Gateway は、小規模な OpenAI 互換の Chat Completions エンドポイントを提供できます。このエンドポイントは**デフォルトで無効**です。まず設定で有効化してください。

-   `POST /v1/chat/completions`
-   Gateway (WS + HTTP 多重化) と同じポート: `http://<gateway-host>:/v1/chat/completions`

内部では、リクエストは通常の Gateway エージェント実行 (`openclaw agent` と同じコードパス) として実行されるため、ルーティング/権限/設定は Gateway のものと一致します。

## 認証

Gateway の認証設定を使用します。ベアラートークンを送信してください:

-   `Authorization: Bearer `

注意:

-   `gateway.auth.mode="token"` の場合、`gateway.auth.token` (または `OPENCLAW_GATEWAY_TOKEN`) を使用します。
-   `gateway.auth.mode="password"` の場合、`gateway.auth.password` (または `OPENCLAW_GATEWAY_PASSWORD`) を使用します。
-   `gateway.auth.rateLimit` が設定されており、認証失敗が多すぎる場合、エンドポイントは `Retry-After` 付きで `429` を返します。

## セキュリティ境界 (重要)

このエンドポイントは、Gateway インスタンスへの**完全なオペレーターアクセス**を提供する表面として扱ってください。

-   ここでの HTTP ベアラー認証は、狭いユーザー単位のスコープモデルではありません。
-   このエンドポイントに対する有効な Gateway トークン/パスワードは、所有者/オペレーターの認証情報と同様に扱うべきです。
-   リクエストは、信頼されたオペレーターアクションと同じ制御プレーンエージェントパスを経由して実行されます。
-   このエンドポイントには、所有者以外/ユーザー単位のツール境界は別途存在しません。呼び出し元がここで Gateway 認証を通過すると、OpenClaw はその呼び出し元をこの Gateway の信頼されたオペレーターとして扱います。
-   ターゲットエージェントのポリシーが機密性の高いツールを許可している場合、このエンドポイントはそれらを使用できます。
-   このエンドポイントはループバック/Tailnet/プライベートイングレスのみで使用し、直接パブリックインターネットに公開しないでください。

[セキュリティ](./security.md) と [リモートアクセス](./remote.md) を参照してください。

## エージェントの選択

カスタムヘッダーは不要です: OpenAI の `model` フィールドにエージェント ID をエンコードします:

-   `model: "openclaw:"` (例: `"openclaw:main"`, `"openclaw:beta"`)
-   `model: "agent:"` (エイリアス)

または、ヘッダーで特定の OpenClaw エージェントをターゲットにします:

-   `x-openclaw-agent-id: ` (デフォルト: `main`)

高度な設定:

-   `x-openclaw-session-key: ` を指定して、セッションルーティングを完全に制御します。

## エンドポイントの有効化

`gateway.http.endpoints.chatCompletions.enabled` を `true` に設定します:

```json
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: true },
      },
    },
  },
}
```

## エンドポイントの無効化

`gateway.http.endpoints.chatCompletions.enabled` を `false` に設定します:

```json
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: false },
      },
    },
  },
}
```

## セッションの動作

デフォルトでは、エンドポイントは**リクエストごとにステートレス**です (新しいセッションキーが呼び出しごとに生成されます)。リクエストに OpenAI の `user` 文字列が含まれている場合、Gateway はそこから安定したセッションキーを導出するため、繰り返しの呼び出しでエージェントセッションを共有できます。

## ストリーミング (SSE)

`stream: true` を設定して Server-Sent Events (SSE) を受信します:

-   `Content-Type: text/event-stream`
-   各イベント行は `data: `
-   ストリームは `data: [DONE]` で終了します

## 例

非ストリーミング:

```bash
curl -sS http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "messages": [{"role":"user","content":"hi"}]
  }'
```

ストリーミング:

```bash
curl -N http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "stream": true,
    "messages": [{"role":"user","content":"hi"}]
  }'
```

[ブリッジプロトコル](./bridge-protocol.md)[OpenResponses API](./openresponses-http-api.md)