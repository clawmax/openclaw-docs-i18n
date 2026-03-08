

  Webインターフェース

  
# Control UI

Control UIは、ゲートウェイによって提供される小さな **Vite + Lit** シングルページアプリケーションです：

-   デフォルト: `http://:18789/`
-   オプションのプレフィックス: `gateway.controlUi.basePath` を設定（例: `/openclaw`）

同じポートで**ゲートウェイWebSocketに直接**通信します。

## クイックオープン（ローカル）

ゲートウェイが同じコンピュータで実行されている場合、以下を開きます：

-   [http://127.0.0.1:18789/](http://127.0.0.1:18789/) (または [http://localhost:18789/](http://localhost:18789/))

ページが読み込まれない場合は、まずゲートウェイを起動します：`openclaw gateway`。認証はWebSocketハンドシェイク時に以下を介して提供されます：

-   `connect.params.auth.token`
-   `connect.params.auth.password` ダッシュボードの設定パネルではトークンを保存できます。パスワードは永続化されません。オンボーディングウィザードはデフォルトでゲートウェイトークンを生成するため、初回接続時にここに貼り付けます。

## デバイスペアリング（初回接続）

新しいブラウザやデバイスからControl UIに接続する場合、ゲートウェイは**一度限りのペアリング承認**を要求します — たとえ `gateway.auth.allowTailscale: true` で同じTailnet上にいてもです。これは不正アクセスを防ぐためのセキュリティ対策です。**表示される内容:** "disconnected (1008): pairing required" **デバイスを承認するには:**

```bash
# 保留中のリクエストを一覧表示
openclaw devices list

# リクエストIDで承認
openclaw devices approve <requestId>
```

一度承認されると、デバイスは記憶され、`openclaw devices revoke --device  --role ` で取り消さない限り、再承認は不要です。トークンのローテーションと取り消しについては [Devices CLI](../cli/devices.md) を参照してください。**注意点:**

-   ローカル接続（`127.0.0.1`）は自動承認されます。
-   リモート接続（LAN、Tailnetなど）は明示的な承認が必要です。
-   各ブラウザプロファイルは一意のデバイスIDを生成するため、ブラウザを切り替えたりブラウザデータをクリアしたりすると、再ペアリングが必要になります。

## 言語サポート

Control UIは、初回読み込み時にブラウザのロケールに基づいて自身をローカライズでき、後からAccessカードの言語ピッカーで上書きできます。

-   サポートされているロケール: `en`, `zh-CN`, `zh-TW`, `pt-BR`, `de`, `es`
-   英語以外の翻訳はブラウザで遅延読み込みされます。
-   選択されたロケールはブラウザストレージに保存され、将来の訪問時に再利用されます。
-   翻訳キーが見つからない場合は英語にフォールバックします。

## できること（現在）

-   ゲートウェイWS経由でモデルとチャット（`chat.history`, `chat.send`, `chat.abort`, `chat.inject`）
-   チャット内でのツール呼び出しのストリーミング + ライブツール出力カード（エージェントイベント）
-   チャネル: WhatsApp/Telegram/Discord/Slack + プラグインチャネル（Mattermostなど）のステータス + QRログイン + チャネルごとの設定（`channels.status`, `web.login.*`, `config.patch`）
-   インスタンス: プレゼンスリスト + 更新（`system-presence`）
-   セッション: 一覧 + セッションごとの思考/詳細オーバーライド（`sessions.list`, `sessions.patch`）
-   Cronジョブ: 一覧/追加/編集/実行/有効化/無効化 + 実行履歴（`cron.*`）
-   スキル: ステータス、有効化/無効化、インストール、APIキー更新（`skills.*`）
-   ノード: 一覧 + キャップ（`node.list`）
-   Exec承認: ゲートウェイまたはノードの許可リストの編集 + `exec host=gateway/node` のポリシー要求（`exec.approvals.*`）
-   設定: `~/.openclaw/openclaw.json` の表示/編集（`config.get`, `config.set`）
-   設定: 適用 + 検証付き再起動（`config.apply`）および最後のアクティブセッションのウェイクアップ
-   設定書き込みには、同時編集の衝突を防ぐためのベースハッシュガードが含まれます
-   設定スキーマ + フォームレンダリング（`config.schema`、プラグイン + チャネルスキーマを含む）；Raw JSONエディタも利用可能
-   デバッグ: ステータス/ヘルス/モデルスナップショット + イベントログ + 手動RPC呼び出し（`status`, `health`, `models.list`）
-   ログ: ゲートウェイファイルログのライブテール表示とフィルタ/エクスポート（`logs.tail`）
-   更新: パッケージ/git更新の実行 + 再起動（`update.run`）と再起動レポート

Cronジョブパネルの注意点:

-   分離ジョブの場合、配信はデフォルトでサマリーを通知します。内部のみの実行にしたい場合は、noneに切り替えることができます。
-   チャネル/ターゲットフィールドは、通知が選択されたときに表示されます。
-   Webhookモードは `delivery.mode = "webhook"` を使用し、`delivery.to` に有効なHTTP(S) webhook URLを設定します。
-   メインセッションジョブの場合、webhookとnoneの配信モードが利用可能です。
-   高度な編集コントロールには、実行後削除、エージェントオーバーライドのクリア、cronの正確/ずらしオプション、エージェントモデル/思考オーバーライド、ベストエフォート配信の切り替えが含まれます。
-   フォーム検証はインラインでフィールドレベルのエラーを表示します。無効な値は修正されるまで保存ボタンを無効にします。
-   `cron.webhookToken` を設定して専用のベアラートークンを送信します。省略された場合、webhookは認証ヘッダーなしで送信されます。
-   非推奨のフォールバック: `notify: true` のレガシージョブは、移行されるまで `cron.webhook` を引き続き使用できます。

## チャットの動作

-   `chat.send` は**ノンブロッキング**です：`{ runId, status: "started" }` で即座にACKし、応答は `chat` イベントを介してストリーミングされます。
-   同じ `idempotencyKey` で再送信すると、実行中は `{ status: "in_flight" }` を返し、完了後は `{ status: "ok" }` を返します。
-   `chat.history` の応答はUIの安全性のためにサイズ制限されています。トランスクリプトエントリが大きすぎる場合、ゲートウェイは長いテキストフィールドを切り詰めたり、重いメタデータブロックを省略したり、大きすぎるメッセージをプレースホルダー（`[chat.history omitted: message too large]`）に置き換えたりする場合があります。
-   `chat.inject` はセッショントランスクリプトにアシスタントノートを追加し、UIのみの更新（エージェント実行なし、チャネル配信なし）のために `chat` イベントをブロードキャストします。
-   停止:
    -   **Stop** をクリック（`chat.abort` を呼び出す）
    -   `/stop` を入力（または `stop`, `stop action`, `stop run`, `stop openclaw`, `please stop` などのスタンドアロン中止フレーズ）でアウトオブバンド中止
    -   `chat.abort` は `{ sessionKey }`（`runId` なし）をサポートし、そのセッションのすべてのアクティブな実行を中止します
-   中止部分の保持:
    -   実行が中止された場合、部分的なアシスタントテキストをUIに表示できます
    -   ゲートウェイは、バッファリングされた出力が存在する場合、中止された部分的なアシスタントテキストをトランスクリプト履歴に永続化します
    -   永続化されたエントリには中止メタデータが含まれるため、トランスクリプトの利用者は中止部分を通常の完了出力と区別できます

## Tailnetアクセス（推奨）

### 統合Tailscale Serve（推奨）

ゲートウェイをループバックに維持し、Tailscale ServeにHTTPSでプロキシさせます：

```bash
openclaw gateway --tailscale serve
```

開く:

-   `https:///` (または設定した `gateway.controlUi.basePath`)

デフォルトでは、Control UI/WebSocket Serveリクエストは、`gateway.auth.allowTailscale` が `true` の場合、Tailscale IDヘッダー（`tailscale-user-login`）を介して認証できます。OpenClawは、`x-forwarded-for` アドレスを `tailscale whois` で解決し、ヘッダーと照合することでIDを検証し、リクエストがTailscaleの `x-forwarded-*` ヘッダー付きでループバックに到達した場合にのみこれらを受け入れます。Serveトラフィックに対してもトークン/パスワードを要求したい場合は、`gateway.auth.allowTailscale: false`（または `gateway.auth.mode: "password"` を強制）を設定します。トークンレスServe認証は、ゲートウェイホストが信頼されていることを前提としています。そのホスト上で信頼できないローカルコードが実行される可能性がある場合は、トークン/パスワード認証を要求してください。

### Tailnetにバインド + トークン

```bash
openclaw gateway --bind tailnet --token "$(openssl rand -hex 32)"
```

次に開く:

-   `http://<tailscale-ip>:18789/` (または設定した `gateway.controlUi.basePath`)

トークンをUI設定に貼り付けます（`connect.params.auth.token` として送信されます）。

## 安全でないHTTP

ダッシュボードをプレーンHTTP（`http://<lan-ip>` または `http://<tailscale-ip>`）で開くと、ブラウザは**非セキュアコンテキスト**で実行され、WebCryptoをブロックします。デフォルトでは、OpenClawはデバイスIDなしのControl UI接続を**ブロック**します。**推奨される修正:** HTTPS（Tailscale Serve）を使用するか、UIをローカルで開きます：

-   `https:///` (Serve)
-   `http://127.0.0.1:18789/` (ゲートウェイホスト上)

**安全でない認証トグル動作:**

```json
{
  gateway: {
    controlUi: { allowInsecureAuth: true },
    bind: "tailnet",
    auth: { mode: "token", token: "replace-me" },
  },
}
```

`allowInsecureAuth` は、Control UIのデバイスIDまたはペアリングチェックをバイパスしません。**緊急時のみ:**

```json
{
  gateway: {
    controlUi: { dangerouslyDisableDeviceAuth: true },
    bind: "tailnet",
    auth: { mode: "token", token: "replace-me" },
  },
}
```

`dangerouslyDisableDeviceAuth` はControl UIのデバイスIDチェックを無効にし、深刻なセキュリティ低下です。緊急使用後は速やかに元に戻してください。HTTPSセットアップのガイダンスについては [Tailscale](../gateway/tailscale.md) を参照してください。

## UIのビルド

ゲートウェイは `dist/control-ui` から静的ファイルを提供します。以下でビルドします：

```bash
pnpm ui:build # 初回実行時にUI依存関係を自動インストール
```

オプションの絶対ベース（固定アセットURLが必要な場合）：

```
OPENCLAW_CONTROL_UI_BASE_PATH=/openclaw/ pnpm ui:build
```

ローカル開発用（別々の開発サーバー）：

```bash
pnpm ui:dev # 初回実行時にUI依存関係を自動インストール
```

次に、UIをゲートウェイWS URL（例: `ws://127.0.0.1:18789`）に向けます。

## デバッグ/テスト: 開発サーバー + リモートゲートウェイ

Control UIは静的ファイルです。WebSocketターゲットは設定可能で、HTTPオリジンとは異なる場合があります。これは、Vite開発サーバーをローカルで実行し、ゲートウェイを別の場所で実行したい場合に便利です。

1.  UI開発サーバーを起動: `pnpm ui:dev`
2.  以下のようなURLを開く：

```
http://localhost:5173/?gatewayUrl=ws://<gateway-host>:18789
```

オプションのワンタイム認証（必要な場合）：

```
http://localhost:5173/?gatewayUrl=wss://<gateway-host>:18789#token=<gateway-token>
```

注意点:

-   `gatewayUrl` は読み込み後にlocalStorageに保存され、URLから削除されます。
-   `token` は現在のタブのメモリにインポートされ、URLから取り除かれます。localStorageには保存されません。
-   `password` はメモリ内のみに保持されます。
-   `gatewayUrl` が設定されている場合、UIは設定や環境認証情報にフォールバックしません。`token`（または `password`）を明示的に提供してください。明示的な認証情報がない場合はエラーです。
-   ゲートウェイがTLS（Tailscale Serve、HTTPSプロキシなど）の背後にある場合は `wss://` を使用します。
-   `gatewayUrl` はクリックジャッキングを防ぐため、トップレベルウィンドウ（埋め込みではない）でのみ受け入れられます。
-   非ループバックのControl UIデプロイメントでは、`gateway.controlUi.allowedOrigins` を明示的に（完全なオリジンで）設定する必要があります。これにはリモート開発セットアップも含まれます。
-   `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback=true` はHostヘッダーオリジンフォールバックモードを有効にしますが、危険なセキュリティモードです。

例：

```json
{
  gateway: {
    controlUi: {
      allowedOrigins: ["http://localhost:5173"],
    },
  },
}
```

リモートアクセスセットアップの詳細: [Remote access](../gateway/remote.md)。

[Web](../web.md)[ダッシュボード](./dashboard.md)