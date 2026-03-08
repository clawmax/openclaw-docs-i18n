

  プロトコルとAPI

  
# ツール呼び出しAPI

OpenClawのGatewayは、単一のツールを直接呼び出すためのシンプルなHTTPエンドポイントを公開しています。これは常に有効ですが、Gateway認証とツールポリシーによって制御されます。

-   `POST /tools/invoke`
-   Gatewayと同じポート（WS + HTTP多重化）: `http://<gateway-host>:/tools/invoke`

デフォルトの最大ペイロードサイズは2 MBです。

## 認証

Gatewayの認証設定を使用します。ベアラートークンを送信します:

-   `Authorization: Bearer `

注意:

-   `gateway.auth.mode="token"`の場合、`gateway.auth.token`（または`OPENCLAW_GATEWAY_TOKEN`）を使用します。
-   `gateway.auth.mode="password"`の場合、`gateway.auth.password`（または`OPENCLAW_GATEWAY_PASSWORD`）を使用します。
-   `gateway.auth.rateLimit`が設定されており、認証失敗が多すぎる場合、エンドポイントは`Retry-After`を伴う`429`を返します。

## リクエストボディ

```json
{
  "tool": "sessions_list",
  "action": "json",
  "args": {},
  "sessionKey": "main",
  "dryRun": false
}
```

フィールド:

-   `tool` (文字列, 必須): 呼び出すツール名。
-   `action` (文字列, オプション): ツールスキーマが`action`をサポートし、argsペイロードがそれを省略している場合、argsにマッピングされます。
-   `args` (オブジェクト, オプション): ツール固有の引数。
-   `sessionKey` (文字列, オプション): ターゲットセッションキー。省略または`"main"`の場合、Gatewayは設定されたメインセッションキーを使用します（`session.mainKey`とデフォルトエージェント、またはグローバルスコープの`global`を尊重します）。
-   `dryRun` (ブール値, オプション): 将来の使用のために予約済み。現在は無視されます。

## ポリシー + ルーティング動作

ツールの可用性は、Gatewayエージェントで使用されるのと同じポリシーチェーンによってフィルタリングされます:

-   `tools.profile` / `tools.byProvider.profile`
-   `tools.allow` / `tools.byProvider.allow`
-   `agents..tools.allow` / `agents..tools.byProvider.allow`
-   グループポリシー（セッションキーがグループまたはチャネルにマッピングされる場合）
-   サブエージェントポリシー（サブエージェントセッションキーで呼び出す場合）

ツールがポリシーによって許可されていない場合、エンドポイントは**404**を返します。Gateway HTTPは、デフォルトで（セッションポリシーがツールを許可している場合でも）ハードな拒否リストも適用します:

-   `sessions_spawn`
-   `sessions_send`
-   `gateway`
-   `whatsapp_login`

この拒否リストは`gateway.tools`を介してカスタマイズできます:

```json
{
  gateway: {
    tools: {
      // HTTP /tools/invoke でブロックする追加ツール
      deny: ["browser"],
      // デフォルトの拒否リストからツールを削除
      allow: ["gateway"],
    },
  },
}
```

グループポリシーがコンテキストを解決するのを支援するために、オプションで設定できます:

-   `x-openclaw-message-channel: ` (例: `slack`, `telegram`)
-   `x-openclaw-account-id: ` (複数のアカウントが存在する場合)

## レスポンス

-   `200` → `{ ok: true, result }`
-   `400` → `{ ok: false, error: { type, message } }` (無効なリクエストまたはツール入力エラー)
-   `401` → 認証失敗
-   `429` → 認証レート制限 (`Retry-After`設定)
-   `404` → ツールが利用不可（見つからない、または許可リストにない）
-   `405` → メソッドが許可されていない
-   `500` → `{ ok: false, error: { type, message } }` (予期しないツール実行エラー; サニタイズされたメッセージ)

## 例

```bash
curl -sS http://127.0.0.1:18789/tools/invoke \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
    "tool": "sessions_list",
    "action": "json",
    "args": {}
  }'
```

[OpenResponses API](./openresponses-http-api.md)[CLIバックエンド](./cli-backends.md)