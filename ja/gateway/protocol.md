

  プロトコルとAPI

  
# Gateway Protocol

Gateway WSプロトコルは、OpenClawの**単一のコントロールプレーン + ノードトランスポート**です。すべてのクライアント（CLI、Web UI、macOSアプリ、iOS/Androidノード、ヘッドレスノード）はWebSocket経由で接続し、ハンドシェイク時に**ロール**と**スコープ**を宣言します。

## トランスポート

-   WebSocket、JSONペイロードを持つテキストフレーム。
-   最初のフレームは`connect`リクエストで**なければなりません**。

## ハンドシェイク (connect)

Gateway → クライアント (接続前チャレンジ):

```json
{
  "type": "event",
  "event": "connect.challenge",
  "payload": { "nonce": "…", "ts": 1737264000000 }
}
```

クライアント → Gateway:

```json
{
  "type": "req",
  "id": "…",
  "method": "connect",
  "params": {
    "minProtocol": 3,
    "maxProtocol": 3,
    "client": {
      "id": "cli",
      "version": "1.2.3",
      "platform": "macos",
      "mode": "operator"
    },
    "role": "operator",
    "scopes": ["operator.read", "operator.write"],
    "caps": [],
    "commands": [],
    "permissions": {},
    "auth": { "token": "…" },
    "locale": "en-US",
    "userAgent": "openclaw-cli/1.2.3",
    "device": {
      "id": "device_fingerprint",
      "publicKey": "…",
      "signature": "…",
      "signedAt": 1737264000000,
      "nonce": "…"
    }
  }
}
```

Gateway → クライアント:

```json
{
  "type": "res",
  "id": "…",
  "ok": true,
  "payload": { "type": "hello-ok", "protocol": 3, "policy": { "tickIntervalMs": 15000 } }
}
```

デバイストークンが発行された場合、`hello-ok`には以下も含まれます:

```json
{
  "auth": {
    "deviceToken": "…",
    "role": "operator",
    "scopes": ["operator.read", "operator.write"]
  }
}
```

### ノードの例

```json
{
  "type": "req",
  "id": "…",
  "method": "connect",
  "params": {
    "minProtocol": 3,
    "maxProtocol": 3,
    "client": {
      "id": "ios-node",
      "version": "1.2.3",
      "platform": "ios",
      "mode": "node"
    },
    "role": "node",
    "scopes": [],
    "caps": ["camera", "canvas", "screen", "location", "voice"],
    "commands": ["camera.snap", "canvas.navigate", "screen.record", "location.get"],
    "permissions": { "camera.capture": true, "screen.record": false },
    "auth": { "token": "…" },
    "locale": "en-US",
    "userAgent": "openclaw-ios/1.2.3",
    "device": {
      "id": "device_fingerprint",
      "publicKey": "…",
      "signature": "…",
      "signedAt": 1737264000000,
      "nonce": "…"
    }
  }
}
```

## フレーミング

-   **リクエスト**: `{type:"req", id, method, params}`
-   **レスポンス**: `{type:"res", id, ok, payload|error}`
-   **イベント**: `{type:"event", event, payload, seq?, stateVersion?}`

副作用を伴うメソッドには**べき等性キー**が必要です（スキーマを参照）。

## ロール + スコープ

### ロール

-   `operator` = コントロールプレーンクライアント (CLI/UI/自動化)。
-   `node` = 機能ホスト (カメラ/画面/キャンバス/system.run)。

### スコープ (operator)

一般的なスコープ:

-   `operator.read`
-   `operator.write`
-   `operator.admin`
-   `operator.approvals`
-   `operator.pairing`

メソッドスコープは最初のゲートに過ぎません。`chat.send`を通じて到達する一部のスラッシュコマンドは、さらに厳格なコマンドレベルのチェックを適用します。例えば、永続的な`/config set`および`/config unset`の書き込みには`operator.admin`が必要です。

### 機能/コマンド/パーミッション (node)

ノードは接続時に機能の主張を宣言します:

-   `caps`: 高レベルの機能カテゴリ。
-   `commands`: 呼び出しのためのコマンド許可リスト。
-   `permissions`: 詳細なトグル (例: `screen.record`, `camera.capture`)。

Gatewayはこれらを**主張**として扱い、サーバーサイドの許可リストを強制します。

## プレゼンス

-   `system-presence`は、デバイスIDをキーとしたエントリを返します。
-   プレゼンスエントリには`deviceId`、`roles`、`scopes`が含まれるため、UIはデバイスが**operator**と**node**の両方として接続する場合でも、デバイスごとに単一の行を表示できます。

### ノードヘルパーメソッド

-   ノードは`skills.bins`を呼び出して、自動許可チェック用の現在のスキル実行ファイルのリストを取得できます。

### オペレーターヘルパーメソッド

-   オペレーターは`tools.catalog` (`operator.read`) を呼び出して、エージェントのランタイムツールカタログを取得できます。レスポンスにはグループ化されたツールと出所メタデータが含まれます:
    -   `source`: `core` または `plugin`
    -   `pluginId`: `source="plugin"`の場合のプラグイン所有者
    -   `optional`: プラグインツールがオプションかどうか

## 実行承認

-   実行リクエストに承認が必要な場合、ゲートウェイは`exec.approval.requested`をブロードキャストします。
-   オペレータークライアントは`exec.approval.resolve`を呼び出して解決します (`operator.approvals`スコープが必要)。
-   `host=node`の場合、`exec.approval.request`には`systemRunPlan` (正規の`argv`/`cwd`/`rawCommand`/セッションメタデータ) を含める**必要があります**。`systemRunPlan`が欠けているリクエストは拒否されます。

## バージョニング

-   `PROTOCOL_VERSION`は`src/gateway/protocol/schema.ts`にあります。
-   クライアントは`minProtocol` + `maxProtocol`を送信します。サーバーは不一致を拒否します。
-   スキーマ + モデルはTypeBox定義から生成されます:
    -   `pnpm protocol:gen`
    -   `pnpm protocol:gen:swift`
    -   `pnpm protocol:check`

## 認証

-   `OPENCLAW_GATEWAY_TOKEN` (または `--token`) が設定されている場合、`connect.params.auth.token`はそれと一致するか、ソケットが閉じられます。
-   ペアリング後、Gatewayは接続ロール + スコープにスコープされた**デバイストークン**を発行します。これは`hello-ok.auth.deviceToken`で返され、クライアントは将来の接続のために永続化する必要があります。
-   デバイストークンは`device.token.rotate`および`device.token.revoke` ( `operator.pairing`スコープが必要) でローテーション/取り消しできます。

## デバイスID + ペアリング

-   ノードは、キーペアのフィンガープリントから導出された安定したデバイスID (`device.id`) を含めるべきです。
-   ゲートウェイはデバイス + ロールごとにトークンを発行します。
-   ローカルの自動承認が有効でない限り、新しいデバイスIDにはペアリング承認が必要です。
-   **ローカル**接続には、ループバックとゲートウェイホスト自身のTailnetアドレスが含まれます (同じホストのTailnetバインドでも自動承認できるように)。
-   すべてのWSクライアントは`connect`中に`device` IDを含める必要があります (operator + node)。コントロールUIは、ブレークグラス用途のために`gateway.controlUi.dangerouslyDisableDeviceAuth`が有効な場合**のみ**、これを省略できます。
-   すべての接続は、サーバー提供の`connect.challenge` nonceに署名する必要があります。

### デバイス認証移行診断

まだチャレンジ前署名動作を使用しているレガシークライアントのために、`connect`は安定した`error.details.reason`を持つ`error.details.code`の下に`DEVICE_AUTH_*`詳細コードを返すようになりました。一般的な移行失敗:

| メッセージ | details.code | details.reason | 意味 |
| --- | --- | --- | --- |
| `device nonce required` | `DEVICE_AUTH_NONCE_REQUIRED` | `device-nonce-missing` | クライアントが`device.nonce`を省略した (または空白を送信した)。 |
| `device nonce mismatch` | `DEVICE_AUTH_NONCE_MISMATCH` | `device-nonce-mismatch` | クライアントが古い/間違ったnonceで署名した。 |
| `device signature invalid` | `DEVICE_AUTH_SIGNATURE_INVALID` | `device-signature` | 署名ペイロードがv2ペイロードと一致しない。 |
| `device signature expired` | `DEVICE_AUTH_SIGNATURE_EXPIRED` | `device-signature-stale` | 署名されたタイムスタンプが許容されるずれの範囲外。 |
| `device identity mismatch` | `DEVICE_AUTH_DEVICE_ID_MISMATCH` | `device-id-mismatch` | `device.id`が公開鍵フィンガープリントと一致しない。 |
| `device public key invalid` | `DEVICE_AUTH_PUBLIC_KEY_INVALID` | `device-public-key` | 公開鍵のフォーマット/正規化に失敗した。 |

移行ターゲット:

-   常に`connect.challenge`を待ちます。
-   サーバーnonceを含むv2ペイロードに署名します。
-   同じnonceを`connect.params.device.nonce`で送信します。
-   推奨される署名ペイロードは`v3`で、デバイス/クライアント/ロール/スコープ/トークン/nonceフィールドに加えて`platform`と`deviceFamily`をバインドします。
-   レガシー`v2`署名は互換性のために引き続き受け入れられますが、ペアリング済みデバイスのメタデータピニングは再接続時のコマンドポリシーを制御します。

## TLS + ピニング

-   WS接続でTLSがサポートされています。
-   クライアントはオプションでゲートウェイ証明書フィンガープリントをピン留めできます (`gateway.tls`設定と`gateway.remote.tlsFingerprint`またはCLIの`--tls-fingerprint`を参照)。

## スコープ

このプロトコルは**完全なゲートウェイAPI** (ステータス、チャネル、モデル、チャット、エージェント、セッション、ノード、承認など) を公開します。正確な表面は`src/gateway/protocol/schema.ts`のTypeBoxスキーマで定義されています。

[Sandbox vs Tool Policy vs Elevated](./sandbox-vs-tool-policy-vs-elevated.md)[Bridge Protocol](./bridge-protocol.md)