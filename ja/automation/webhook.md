

  オートメーション

  
# Webhooks

ゲートウェイは、外部トリガーのための小さなHTTP Webhookエンドポイントを公開できます。

## 有効化

```json
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    // オプション: 明示的な `agentId` ルーティングをこの許可リストに制限します。
    // 省略するか "*" を含めると、任意のエージェントを許可します。
    // [] に設定すると、すべての明示的な `agentId` ルーティングを拒否します。
    allowedAgentIds: ["hooks", "main"],
  },
}
```

注意:

-   `hooks.token` は `hooks.enabled=true` の場合に必須です。
-   `hooks.path` のデフォルトは `/hooks` です。

## 認証

すべてのリクエストにはフックトークンを含める必要があります。ヘッダーを使用することを推奨します:

-   `Authorization: Bearer ` (推奨)
-   `x-openclaw-token: `
-   クエリストリングトークンは拒否されます (`?token=...` は `400` を返します)。

## エンドポイント

### POST /hooks/wake

ペイロード:

```json
{ "text": "System line", "mode": "now" }
```

-   `text` **必須** (文字列): イベントの説明 (例: "新規メール受信")。
-   `mode` オプション (`now` | `next-heartbeat`): 即時ハートビートをトリガーするか (デフォルト `now`)、次の定期チェックを待つか。

効果:

-   **メイン** セッションのシステムイベントをエンキューします
-   `mode=now` の場合、即時ハートビートをトリガーします

### POST /hooks/agent

ペイロード:

```json
{
  "message": "Run this",
  "name": "Email",
  "agentId": "hooks",
  "sessionKey": "hook:email:msg-123",
  "wakeMode": "now",
  "deliver": true,
  "channel": "last",
  "to": "+15551234567",
  "model": "openai/gpt-5.2-mini",
  "thinking": "low",
  "timeoutSeconds": 120
}
```

-   `message` **必須** (文字列): エージェントが処理するプロンプトまたはメッセージ。
-   `name` オプション (文字列): フックの人間が読める名前 (例: "GitHub")。セッション概要の接頭辞として使用されます。
-   `agentId` オプション (文字列): このフックを特定のエージェントにルーティングします。不明なIDはデフォルトエージェントにフォールバックします。設定すると、フックは解決されたエージェントのワークスペースと設定を使用して実行されます。
-   `sessionKey` オプション (文字列): エージェントのセッションを識別するために使用されるキー。デフォルトでは、このフィールドは `hooks.allowRequestSessionKey=true` でない限り拒否されます。
-   `wakeMode` オプション (`now` | `next-heartbeat`): 即時ハートビートをトリガーするか (デフォルト `now`)、次の定期チェックを待つか。
-   `deliver` オプション (ブール値): `true` の場合、エージェントの応答がメッセージングチャネルに送信されます。デフォルトは `true` です。ハートビート確認のみの応答は自動的にスキップされます。
-   `channel` オプション (文字列): 配信のためのメッセージングチャネル。次のいずれか: `last`, `whatsapp`, `telegram`, `discord`, `slack`, `mattermost` (プラグイン), `signal`, `imessage`, `msteams`。デフォルトは `last`。
-   `to` オプション (文字列): チャネルの受信者識別子 (例: WhatsApp/Signalの電話番号、TelegramのチャットID、Discord/Slack/Mattermost (プラグイン)のチャネルID、MS Teamsの会話ID)。デフォルトはメインセッションの最後の受信者です。
-   `model` オプション (文字列): モデルオーバーライド (例: `anthropic/claude-3-5-sonnet` またはエイリアス)。制限されている場合は許可モデルリストに含まれている必要があります。
-   `thinking` オプション (文字列): 思考レベルオーバーライド (例: `low`, `medium`, `high`)。
-   `timeoutSeconds` オプション (数値): エージェント実行の最大継続時間 (秒単位)。

効果:

-   **分離された** エージェントターンを実行します (独自のセッションキー)
-   常に概要を **メイン** セッションに投稿します
-   `wakeMode=now` の場合、即時ハートビートをトリガーします

## セッションキーポリシー (破壊的変更)

`/hooks/agent` ペイロードの `sessionKey` オーバーライドはデフォルトで無効です。

-   推奨: 固定の `hooks.defaultSessionKey` を設定し、リクエストオーバーライドはオフに保ちます。
-   オプション: 必要な場合のみリクエストオーバーライドを許可し、接頭辞を制限します。

推奨設定:

```json
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
  },
}
```

互換性設定 (レガシー動作):

```json
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    allowRequestSessionKey: true,
    allowedSessionKeyPrefixes: ["hook:"], // 強く推奨
  },
}
```

### POST /hooks/&lt;name&gt; (マッピング済み)

カスタムフック名は `hooks.mappings` 経由で解決されます (設定を参照)。マッピングは任意のペイロードを `wake` または `agent` アクションに変換でき、オプションでテンプレートやコード変換を含みます。マッピングオプション (概要):

-   `hooks.presets: ["gmail"]` は組み込みのGmailマッピングを有効にします。
-   `hooks.mappings` では、設定内で `match`, `action`, およびテンプレートを定義できます。
-   `hooks.transformsDir` + `transform.module` は、カスタムロジック用のJS/TSモジュールをロードします。
    -   `hooks.transformsDir` (設定されている場合) は、OpenClaw設定ディレクトリ (通常 `~/.openclaw/hooks/transforms`) 下の変換ルート内に留まる必要があります。
    -   `transform.module` は、有効な変換ディレクトリ内で解決される必要があります (トラバーサル/エスケープパスは拒否されます)。
-   `match.source` を使用して、汎用インジェストエンドポイント (ペイロード駆動ルーティング) を維持します。
-   TS変換には、TSローダー (例: `bun` または `tsx`) または実行時にプリコンパイルされた `.js` が必要です。
-   マッピングで `deliver: true` + `channel`/`to` を設定して、返信をチャット画面にルーティングします (`channel` はデフォルトで `last` で、WhatsAppにフォールバックします)。
-   `agentId` はフックを特定のエージェントにルーティングします; 不明なIDはデフォルトエージェントにフォールバックします。
-   `hooks.allowedAgentIds` は明示的な `agentId` ルーティングを制限します。省略するか (または `*` を含める) と任意のエージェントを許可します。`[]` に設定すると、明示的な `agentId` ルーティングを拒否します。
-   `hooks.defaultSessionKey` は、明示的なキーが提供されていない場合のフックエージェント実行のデフォルトセッションを設定します。
-   `hooks.allowRequestSessionKey` は、`/hooks/agent` ペイロードが `sessionKey` を設定できるかどうかを制御します (デフォルト: `false`)。
-   `hooks.allowedSessionKeyPrefixes` は、リクエストペイロードおよびマッピングからの明示的な `sessionKey` 値をオプションで制限します。
-   `allowUnsafeExternalContent: true` は、そのフックの外部コンテンツ安全性ラッパーを無効にします (危険; 信頼された内部ソースのみに使用)。
-   `openclaw webhooks gmail setup` は、`openclaw webhooks gmail run` 用の `hooks.gmail` 設定を書き込みます。完全なGmailウォッチフローについては [Gmail Pub/Sub](./gmail-pubsub.md) を参照してください。

## 応答

-   `/hooks/wake` に対して `200`
-   `/hooks/agent` に対して `200` (非同期実行受理)
-   認証失敗時に `401`
-   同じクライアントからの繰り返し認証失敗後に `429` (`Retry-After` を確認)
-   無効なペイロードに対して `400`
-   過大なペイロードに対して `413`

## 例

```bash
curl -X POST http://127.0.0.1:18789/hooks/wake \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"text":"New email received","mode":"now"}'
```

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","wakeMode":"next-heartbeat"}'
```

### 別のモデルを使用する

エージェントペイロード (またはマッピング) に `model` を追加して、その実行のモデルをオーバーライドします:

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","model":"openai/gpt-5.2-mini"}'
```

`agents.defaults.models` を強制する場合は、オーバーライドモデルがそこに含まれていることを確認してください。

```bash
curl -X POST http://127.0.0.1:18789/hooks/gmail \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"source":"gmail","messages":[{"from":"Ada","subject":"Hello","snippet":"Hi"}]}'
```

## セキュリティ

-   フックエンドポイントはループバック、テールネット、または信頼されたリバースプロキシの背後に保ちます。
-   専用のフックトークンを使用します; ゲートウェイ認証トークンを再利用しないでください。
-   繰り返しの認証失敗は、クライアントアドレスごとにレート制限され、ブルートフォース試行を遅らせます。
-   マルチエージェントルーティングを使用する場合は、`hooks.allowedAgentIds` を設定して明示的な `agentId` 選択を制限します。
-   呼び出し元が選択したセッションが必要でない限り、`hooks.allowRequestSessionKey=false` を保ちます。
-   リクエスト `sessionKey` を有効にする場合は、`hooks.allowedSessionKeyPrefixes` を制限します (例: `["hook:"]`)。
-   機密性の高い生のペイロードをWebhookログに含めないようにします。
-   フックペイロードは信頼できないものとして扱われ、デフォルトで安全性の境界でラップされます。特定のフックでこれを無効にする必要がある場合は、そのフックのマッピングで `allowUnsafeExternalContent: true` を設定します (危険)。

[オートメーション トラブルシューティング](./troubleshooting.md)[Gmail PubSub](./gmail-pubsub.md)