

  メッセージングプラットフォーム

  
# Synology Chat

ステータス: Synology Chat の Webhook を使用したダイレクトメッセージチャネルとして、プラグイン経由でサポートされています。このプラグインは Synology Chat のアウトバウンド Webhook からのインバウンドメッセージを受け取り、Synology Chat のインバウンド Webhook を通じて返信を送信します。

## プラグインが必要です

Synology Chat はプラグインベースであり、デフォルトのコアチャネルインストールには含まれていません。ローカルのチェックアウトからインストールします:

```bash
openclaw plugins install ./extensions/synology-chat
```

詳細: [プラグイン](../tools/plugin.md)

## クイックセットアップ

1.  Synology Chat プラグインをインストールして有効化します。
2.  Synology Chat の連携機能で:
    -   インバウンド Webhook を作成し、その URL をコピーします。
    -   シークレットトークンを設定したアウトバウンド Webhook を作成します。
3.  アウトバウンド Webhook の URL を OpenClaw ゲートウェイに向けます:
    -   デフォルトでは `https://gateway-host/webhook/synology`。
    -   またはカスタムの `channels.synology-chat.webhookPath`。
4.  OpenClaw で `channels.synology-chat` を設定します。
5.  ゲートウェイを再起動し、Synology Chat ボットに DM を送信します。

最小限の設定:

```json
{
  channels: {
    "synology-chat": {
      enabled: true,
      token: "synology-outgoing-token",
      incomingUrl: "https://nas.example.com/webapi/entry.cgi?api=SYNO.Chat.External&method=incoming&version=2&token=...",
      webhookPath: "/webhook/synology",
      dmPolicy: "allowlist",
      allowedUserIds: ["123456"],
      rateLimitPerMinute: 30,
      allowInsecureSsl: false,
    },
  },
}
```

## 環境変数

デフォルトアカウントについては、環境変数を使用できます:

-   `SYNOLOGY_CHAT_TOKEN`
-   `SYNOLOGY_CHAT_INCOMING_URL`
-   `SYNOLOGY_NAS_HOST`
-   `SYNOLOGY_ALLOWED_USER_IDS` (カンマ区切り)
-   `SYNOLOGY_RATE_LIMIT`
-   `OPENCLAW_BOT_NAME`

設定ファイルの値は環境変数を上書きします。

## DM ポリシーとアクセス制御

-   `dmPolicy: "allowlist"` が推奨されるデフォルトです。
-   `allowedUserIds` は Synology ユーザー ID のリスト（またはカンマ区切りの文字列）を受け付けます。
-   `allowlist` モードでは、空の `allowedUserIds` リストは設定ミスと見なされ、Webhook ルートは起動しません（すべてを許可するには `dmPolicy: "open"` を使用してください）。
-   `dmPolicy: "open"` は任意の送信者を許可します。
-   `dmPolicy: "disabled"` は DM をブロックします。
-   ペアリング承認は以下で動作します:
    -   `openclaw pairing list synology-chat`
    -   `openclaw pairing approve synology-chat `

## アウトバウンド配信

ターゲットとして数値の Synology Chat ユーザー ID を使用します。例:

```bash
openclaw message send --channel synology-chat --target 123456 --text "Hello from OpenClaw"
openclaw message send --channel synology-chat --target synology-chat:123456 --text "Hello again"
```

メディア送信は、URL ベースのファイル配信でサポートされています。

## マルチアカウント

複数の Synology Chat アカウントは `channels.synology-chat.accounts` の下でサポートされています。各アカウントはトークン、インバウンド URL、Webhook パス、DM ポリシー、制限を上書きできます。

```json
{
  channels: {
    "synology-chat": {
      enabled: true,
      accounts: {
        default: {
          token: "token-a",
          incomingUrl: "https://nas-a.example.com/...token=...",
        },
        alerts: {
          token: "token-b",
          incomingUrl: "https://nas-b.example.com/...token=...",
          webhookPath: "/webhook/synology-alerts",
          dmPolicy: "allowlist",
          allowedUserIds: ["987654"],
        },
      },
    },
  },
}
```

## セキュリティに関する注意点

-   `token` は秘密に保管し、漏洩した場合はローテーションしてください。
-   ローカルの自己署名 NAS 証明書を明示的に信頼する場合を除き、`allowInsecureSsl: false` を維持してください。
-   インバウンド Webhook リクエストはトークン検証され、送信者ごとにレート制限されます。
-   本番環境では `dmPolicy: "allowlist"` を推奨します。

[Signal](./signal.md)[Slack](./slack.md)

---