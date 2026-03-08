

  メッセージングプラットフォーム

  
# Mattermost

ステータス: プラグイン経由でサポート (ボットトークン + WebSocket イベント)。チャンネル、グループ、DM がサポートされています。Mattermost はセルフホスト可能なチームメッセージングプラットフォームです。製品の詳細とダウンロードについては公式サイト [mattermost.com](https://mattermost.com) を参照してください。

## プラグイン必須

Mattermost はプラグインとして提供され、コアインストールには同梱されていません。CLI (npm レジストリ) 経由でインストールします:

```bash
openclaw plugins install @openclaw/mattermost
```

ローカルチェックアウト (git リポジトリから実行する場合):

```bash
openclaw plugins install ./extensions/mattermost
```

configure/オンボーディング中に Mattermost を選択し、git チェックアウトが検出された場合、OpenClaw は自動的にローカルインストールパスを提供します。詳細: [プラグイン](../tools/plugin.md)

## クイックセットアップ

1.  Mattermost プラグインをインストールします。
2.  Mattermost ボットアカウントを作成し、**ボットトークン**をコピーします。
3.  Mattermost の**ベース URL** (例: `https://chat.example.com`) をコピーします。
4.  OpenClaw を設定し、ゲートウェイを起動します。

最小限の設定:

```json
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
    },
  },
}
```

## ネイティブスラッシュコマンド

ネイティブスラッシュコマンドはオプトインです。有効にすると、OpenClaw は Mattermost API 経由で `oc_*` スラッシュコマンドを登録し、ゲートウェイ HTTP サーバー上でコールバック POST を受け取ります。

```json
{
  channels: {
    mattermost: {
      commands: {
        native: true,
        nativeSkills: true,
        callbackPath: "/api/channels/mattermost/command",
        // Mattermost がゲートウェイに直接到達できない場合 (リバースプロキシ/公開URL) に使用します。
        callbackUrl: "https://gateway.example.com/api/channels/mattermost/command",
      },
    },
  },
}
```

注意点:

-   `native: "auto"` は Mattermost ではデフォルトで無効です。有効にするには `native: true` を設定します。
-   `callbackUrl` が省略された場合、OpenClaw はゲートウェイのホスト/ポート + `callbackPath` から URL を導出します。
-   マルチアカウント設定では、`commands` はトップレベルまたは `channels.mattermost.accounts..commands` の下に設定できます (アカウントの値がトップレベルのフィールドを上書きします)。
-   コマンドコールバックはコマンドごとのトークンで検証され、トークンチェックが失敗した場合は閉じた状態で失敗します。
-   到達可能性要件: コールバックエンドポイントは Mattermost サーバーから到達可能でなければなりません。
    -   Mattermost が OpenClaw と同じホスト/ネットワーク名前空間で実行されていない限り、`callbackUrl` を `localhost` に設定しないでください。
    -   その URL が `/api/channels/mattermost/command` を OpenClaw にリバースプロキシしない限り、`callbackUrl` を Mattermost のベース URL に設定しないでください。
    -   簡単なチェック: `curl https://<gateway-host>/api/channels/mattermost/command`; GET リクエストは OpenClaw から `405 Method Not Allowed` を返すべきで、`404` ではありません。
-   Mattermost エグレス許可リスト要件:
    -   コールバックターゲットがプライベート/Tailnet/内部アドレスの場合、Mattermost の `ServiceSettings.AllowedUntrustedInternalConnections` にコールバックホスト/ドメインを含めるように設定します。
    -   完全な URL ではなく、ホスト/ドメインエントリを使用します。
        -   良い例: `gateway.tailnet-name.ts.net`
        -   悪い例: `https://gateway.tailnet-name.ts.net`

## 環境変数 (デフォルトアカウント)

環境変数を好む場合は、ゲートウェイホスト上でこれらを設定します:

-   `MATTERMOST_BOT_TOKEN=...`
-   `MATTERMOST_URL=https://chat.example.com`

環境変数は**デフォルト**アカウント (`default`) にのみ適用されます。他のアカウントは設定値を使用する必要があります。

## チャットモード

Mattermost は DM に自動的に応答します。チャンネルの動作は `chatmode` で制御されます:

-   `oncall` (デフォルト): チャンネルで @メンションされた場合のみ応答します。
-   `onmessage`: すべてのチャンネルメッセージに応答します。
-   `onchar`: メッセージがトリガープレフィックスで始まる場合に応答します。

設定例:

```json
{
  channels: {
    mattermost: {
      chatmode: "onchar",
      oncharPrefixes: [">", "!"],
    },
  },
}
```

注意点:

-   `onchar` でも明示的な @メンションには応答します。
-   レガシー設定では `channels.mattermost.requireMention` が尊重されますが、`chatmode` が推奨されます。

## アクセス制御 (DM)

-   デフォルト: `channels.mattermost.dmPolicy = "pairing"` (不明な送信者にはペアリングコードが送られます)。
-   承認方法:
    -   `openclaw pairing list mattermost`
    -   `openclaw pairing approve mattermost `
-   公開 DM: `channels.mattermost.dmPolicy="open"` に加えて `channels.mattermost.allowFrom=["*"]`。

## チャンネル (グループ)

-   デフォルト: `channels.mattermost.groupPolicy = "allowlist"` (メンションゲート)。
-   `channels.mattermost.groupAllowFrom` で送信者を許可リストに追加します (ユーザー ID を推奨)。
-   `@username` マッチングは可変であり、`channels.mattermost.dangerouslyAllowNameMatching: true` の場合にのみ有効になります。
-   オープンチャンネル: `channels.mattermost.groupPolicy="open"` (メンションゲート)。
-   実行時注意: `channels.mattermost` が完全に欠落している場合、ランタイムはグループチェックのために `groupPolicy="allowlist"` にフォールバックします (`channels.defaults.groupPolicy` が設定されていても)。

## アウトバウンド配信のターゲット

`openclaw message send` または cron/webhook で使用するターゲット形式:

-   チャンネルの場合: `channel:`
-   DM の場合: `user:`
-   DM の場合: `@username` (Mattermost API 経由で解決)

ベア ID はチャンネルとして扱われます。

## リアクション (メッセージツール)

-   `channel=mattermost` で `message action=react` を使用します。
-   `messageId` は Mattermost の投稿 ID です。
-   `emoji` は `thumbsup` や `:+1:` のような名前を受け付けます (コロンはオプション)。
-   リアクションを削除するには `remove=true` (ブール値) を設定します。
-   リアクションの追加/削除イベントは、ルーティングされたエージェントセッションにシステムイベントとして転送されます。

例:

```bash
message action=react channel=mattermost target=channel:<channelId> messageId=<postId> emoji=thumbsup
message action=react channel=mattermost target=channel:<channelId> messageId=<postId> emoji=thumbsup remove=true
```

設定:

-   `channels.mattermost.actions.reactions`: リアクションアクションの有効/無効 (デフォルト true)。
-   アカウントごとの上書き: `channels.mattermost.accounts..actions.reactions`。

## インタラクティブボタン (メッセージツール)

クリック可能なボタン付きのメッセージを送信します。ユーザーがボタンをクリックすると、エージェントが選択を受け取り、応答できます。ボタンを有効にするには、チャンネル機能に `inlineButtons` を追加します:

```json
{
  channels: {
    mattermost: {
      capabilities: ["inlineButtons"],
    },
  },
}
```

`buttons` パラメータを指定して `message action=send` を使用します。ボタンは 2 次元配列 (ボタンの行) です:

```bash
message action=send channel=mattermost target=channel:<channelId> buttons=[[{"text":"Yes","callback_data":"yes"},{"text":"No","callback_data":"no"}]]
```

ボタンフィールド:

-   `text` (必須): 表示ラベル。
-   `callback_data` (必須): クリック時に送り返される値 (アクション ID として使用)。
-   `style` (オプション): `"default"`、`"primary"`、`"danger"`。

ユーザーがボタンをクリックすると:

1.  すべてのボタンが確認行に置き換えられます (例: ”✓ **Yes** selected by @user”)。
2.  エージェントは選択をインバウンドメッセージとして受け取り、応答します。

注意点:

-   ボタンコールバックは HMAC-SHA256 検証を使用します (自動、設定不要)。
-   Mattermost はセキュリティ機能として API レスポンスからコールバックデータを削除するため、クリック時にすべてのボタンが削除されます — 部分的な削除は不可能です。
-   ハイフンやアンダースコアを含むアクション ID は自動的にサニタイズされます (Mattermost ルーティング制限)。

設定:

-   `channels.mattermost.capabilities`: 機能文字列の配列。エージェントシステムプロンプトにボタンツールの説明を表示するには `"inlineButtons"` を追加します。
-   `channels.mattermost.interactions.callbackBaseUrl`: ボタンコールバック用のオプションの外部ベース URL (例: `https://gateway.example.com`)。Mattermost がゲートウェイのバインドホストに直接到達できない場合に使用します。
-   マルチアカウント設定では、同じフィールドを `channels.mattermost.accounts..interactions.callbackBaseUrl` の下に設定することもできます。
-   `interactions.callbackBaseUrl` が省略された場合、OpenClaw は `gateway.customBindHost` + `gateway.port` からコールバック URL を導出し、次に `http://localhost:` にフォールバックします。
-   到達可能性ルール: ボタンコールバック URL は Mattermost サーバーから到達可能でなければなりません。`localhost` は Mattermost と OpenClaw が同じホスト/ネットワーク名前空間で実行されている場合にのみ機能します。
-   コールバックターゲットがプライベート/Tailnet/内部の場合は、そのホスト/ドメインを Mattermost の `ServiceSettings.AllowedUntrustedInternalConnections` に追加してください。

### 直接 API 連携 (外部スクリプト)

外部スクリプトや Webhook は、エージェントの `message` ツールを経由する代わりに、Mattermost REST API を介して直接ボタンを投稿できます。可能な限り拡張機能の `buildButtonAttachments()` を使用してください。生の JSON を投稿する場合は、以下のルールに従ってください: **ペイロード構造:**

```json
{
  channel_id: "<channelId>",
  message: "Choose an option:",
  props: {
    attachments: [
      {
        actions: [
          {
            id: "mybutton01", // 英数字のみ — 下記参照
            type: "button", // 必須、ないとクリックが黙って無視されます
            name: "Approve", // 表示ラベル
            style: "primary", // オプション: "default", "primary", "danger"
            integration: {
              url: "https://gateway.example.com/mattermost/interactions/default",
              context: {
                action_id: "mybutton01", // ボタン id と一致する必要あり (名前参照用)
                action: "approve",
                // ... 任意のカスタムフィールド ...
                _token: "<hmac>", // HMAC セクションを参照
              },
            },
          },
        ],
      },
    ],
  },
}
```

**重要なルール:**

1.  添付ファイルはトップレベルの `attachments` ではなく `props.attachments` に入れます (黙って無視されます)。
2.  すべてのアクションに `type: "button"` が必要です — これがないと、クリックは黙って飲み込まれます。
3.  すべてのアクションに `id` フィールドが必要です — Mattermost は ID のないアクションを無視します。
4.  アクション `id` は**英数字のみ** (`[a-zA-Z0-9]`) でなければなりません。ハイフンやアンダースコアは Mattermost のサーバーサイドアクションルーティングを壊します (404 を返します)。使用前に削除してください。
5.  `context.action_id` はボタンの `id` と一致する必要があり、確認メッセージが生の ID ではなくボタン名 (例: "Approve") を表示できるようにします。
6.  `context.action_id` は必須です — これがないとインタラクションハンドラは 400 を返します。

**HMAC トークン生成:** ゲートウェイは HMAC-SHA256 でボタンクリックを検証します。外部スクリプトはゲートウェイの検証ロジックと一致するトークンを生成する必要があります:

1.  ボットトークンから秘密鍵を導出: `HMAC-SHA256(key="openclaw-mattermost-interactions", data=botToken)`
2.  `_token` を**除く**すべてのフィールドを含むコンテキストオブジェクトを構築します。
3.  **ソートされたキー**と**スペースなし**でシリアライズします (ゲートウェイはソートされたキーで `JSON.stringify` を使用し、コンパクトな出力を生成します)。
4.  署名: `HMAC-SHA256(key=secret, data=serializedContext)`
5.  結果の 16 進ダイジェストをコンテキスト内の `_token` として追加します。

Python の例:

```typescript
import hmac, hashlib, json

secret = hmac.new(
    b"openclaw-mattermost-interactions",
    bot_token.encode(), hashlib.sha256
).hexdigest()

ctx = {"action_id": "mybutton01", "action": "approve"}
payload = json.dumps(ctx, sort_keys=True, separators=(",", ":"))
token = hmac.new(secret.encode(), payload.encode(), hashlib.sha256).hexdigest()

context = {**ctx, "_token": token}
```

一般的な HMAC の落とし穴:

-   Python の `json.dumps` はデフォルトでスペースを追加します (`{"key": "val"}`)。JavaScript のコンパクトな出力 (`{"key":"val"}`) に一致させるには `separators=(",", ":")` を使用します。
-   常に**すべての**コンテキストフィールド (`_token` を除く) に署名します。ゲートウェイは `_token` を削除してから残りのすべてに署名します。サブセットに署名すると、黙って検証が失敗します。
-   `sort_keys=True` を使用します — ゲートウェイは署名前にキーをソートし、Mattermost はペイロードを保存する際にコンテキストフィールドを並べ替える可能性があります。
-   秘密鍵はランダムバイトではなく、ボットトークンから導出します (決定的)。秘密鍵はボタンを作成するプロセスと検証するゲートウェイ間で同じでなければなりません。

## ディレクトリアダプター

Mattermost プラグインには、Mattermost API 経由でチャンネル名とユーザー名を解決するディレクトリアダプターが含まれています。これにより、`openclaw message send` および cron/webhook 配信で `#channel-name` および `@username` ターゲットが可能になります。設定は不要です — アダプターはアカウント設定のボットトークンを使用します。

## マルチアカウント

Mattermost は `channels.mattermost.accounts` の下で複数のアカウントをサポートします:

```json
{
  channels: {
    mattermost: {
      accounts: {
        default: { name: "プライマリ", botToken: "mm-token", baseUrl: "https://chat.example.com" },
        alerts: { name: "アラート", botToken: "mm-token-2", baseUrl: "https://alerts.example.com" },
      },
    },
  },
}
```

## トラブルシューティング

-   チャンネルで応答がない: ボットがチャンネルに参加していること、@メンションしていること (oncall)、トリガープレフィックスを使用していること (onchar)、または `chatmode: "onmessage"` を設定していることを確認してください。
-   認証エラー: ボットトークン、ベース URL、アカウントが有効かどうかを確認してください。
-   マルチアカウントの問題: 環境変数は `default` アカウントにのみ適用されます。
-   ボタンが白いボックスとして表示される: エージェントが不正なボタンデータを送信している可能性があります。各ボタンに `text` と `callback_data` の両方のフィールドがあることを確認してください。
-   ボタンはレンダリングされるがクリックしても何も起こらない: Mattermost サーバー設定の `AllowedUntrustedInternalConnections` に `127.0.0.1 localhost` が含まれていること、および ServiceSettings で `EnablePostActionIntegration` が `true` に設定されていることを確認してください。
-   クリック時にボタンが 404 を返す: ボタンの `id` にハイフンやアンダースコアが含まれている可能性があります。Mattermost のアクションルーターは非英数字 ID で壊れます。`[a-zA-Z0-9]` のみを使用してください。
-   ゲートウェイログに `invalid _token`: HMAC 不一致。すべてのコンテキストフィールド (サブセットではない) に署名していること、ソートされたキーを使用していること、コンパクトな JSON (スペースなし) を使用していることを確認してください。上記の HMAC セクションを参照してください。
-   ゲートウェイログに `missing _token in context`: ボタンのコンテキストに `_token` フィールドがありません。統合ペイロードを構築する際に含まれていることを確認してください。
-   確認メッセージにボタン名ではなく生の ID が表示される: `context.action_id` がボタンの `id` と一致しません。両方を同じサニタイズされた値に設定してください。
-   エージェントがボタンについて知らない: Mattermost チャンネル設定に `capabilities: ["inlineButtons"]` を追加してください。

[Matrix](./matrix.md)[Microsoft Teams](./msteams.md)