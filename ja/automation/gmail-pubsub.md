

  自動化

  
# Gmail PubSub

目標: Gmail watch -> Pub/Sub プッシュ -> `gog gmail watch serve` -> OpenClaw webhook.

## 前提条件

-   `gcloud` がインストールされ、ログイン済み ([インストールガイド](https://docs.cloud.google.com/sdk/docs/install-sdk)).
-   `gog` (gogcli) がインストールされ、Gmailアカウントで認可済み ([gogcli.sh](https://gogcli.sh/)).
-   OpenClaw フックが有効化済み ([Webhooks](./webhook.md) を参照).
-   `tailscale` がログイン済み ([tailscale.com](https://tailscale.com/)). サポートされているセットアップでは、公開HTTPSエンドポイントにTailscale Funnelを使用します。他のトンネルサービスも動作する可能性がありますが、DIY/非サポートであり、手動での接続が必要です。現時点では、Tailscaleがサポート対象です。

フック設定の例 (Gmailプリセットマッピングを有効化):

```json
{
  hooks: {
    enabled: true,
    token: "OPENCLAW_HOOK_TOKEN",
    path: "/hooks",
    presets: ["gmail"],
  },
}
```

Gmail要約をチャット画面に配信するには、`deliver` とオプションの `channel`/`to` を設定するマッピングでプリセットを上書きします:

```json
{
  hooks: {
    enabled: true,
    token: "OPENCLAW_HOOK_TOKEN",
    presets: ["gmail"],
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate: "New email from {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}\n{{messages[0].body}}",
        model: "openai/gpt-5.2-mini",
        deliver: true,
        channel: "last",
        // to: "+15551234567"
      },
    ],
  },
}
```

固定のチャネルを使用したい場合は、`channel` + `to` を設定します。そうでなければ `channel: "last"` は最後の配信ルートを使用します (フォールバックはWhatsApp)。Gmail実行に安価なモデルを強制するには、マッピング内で `model` を設定します (`provider/model` またはエイリアス)。`agents.defaults.models` を強制する場合は、そこに含めてください。Gmailフック専用のデフォルトモデルと思考レベルを設定するには、設定に `hooks.gmail.model` / `hooks.gmail.thinking` を追加します:

```json
{
  hooks: {
    gmail: {
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      thinking: "off",
    },
  },
}
```

注意点:

-   フックごとのマッピング内の `model`/`thinking` は、これらのデフォルトを上書きします。
-   フォールバック順序: `hooks.gmail.model` → `agents.defaults.model.fallbacks` → プライマリ (認証/レート制限/タイムアウト).
-   `agents.defaults.models` が設定されている場合、Gmailモデルは許可リストに含まれている必要があります。
-   Gmailフックのコンテンツは、デフォルトで外部コンテンツ安全境界でラップされます。無効化するには (危険)、`hooks.gmail.allowUnsafeExternalContent: true` を設定します。

ペイロード処理をさらにカスタマイズするには、`hooks.mappings` を追加するか、`~/.openclaw/hooks/transforms` 配下にJS/TS変換モジュールを追加します ([Webhooks](./webhook.md) を参照)。

## ウィザード (推奨)

OpenClawヘルパーを使用してすべてを接続します (macOSではbrew経由で依存関係をインストール):

```bash
openclaw webhooks gmail setup \
  --account openclaw@gmail.com
```

デフォルト:

-   公開プッシュエンドポイントにTailscale Funnelを使用します。
-   `openclaw webhooks gmail run` 用に `hooks.gmail` 設定を書き込みます。
-   Gmailフックプリセットを有効化します (`hooks.presets: ["gmail"]`).

パスに関する注意: `tailscale.mode` が有効な場合、OpenClawは自動的に `hooks.gmail.serve.path` を `/` に設定し、公開パスを `hooks.gmail.tailscale.path` (デフォルト `/gmail-pubsub`) に保持します。これは、Tailscaleがプロキシする前に設定されたパス接頭辞を取り除くためです。バックエンドが接頭辞付きのパスを受け取る必要がある場合は、`hooks.gmail.tailscale.target` (または `--tailscale-target`) を `http://127.0.0.1:8788/gmail-pubsub` のような完全なURLに設定し、`hooks.gmail.serve.path` と一致させます。カスタムエンドポイントが必要ですか？ `--push-endpoint ` または `--tailscale off` を使用してください。プラットフォームに関する注意: macOSではウィザードが `gcloud`、`gogcli`、`tailscale` をHomebrew経由でインストールします。Linuxでは事前に手動でインストールしてください。ゲートウェイ自動起動 (推奨):

-   `hooks.enabled=true` かつ `hooks.gmail.account` が設定されている場合、ゲートウェイは起動時に `gog gmail watch serve` を開始し、ウォッチを自動更新します。
-   `OPENCLAW_SKIP_GMAIL_WATCHER=1` を設定してオプトアウトします (自分でデーモンを実行する場合に便利)。
-   手動デーモンを同時に実行しないでください。そうしないと `listen tcp 127.0.0.1:8788: bind: address already in use` が発生します。

手動デーモン (`gog gmail watch serve` + 自動更新を開始):

```bash
openclaw webhooks gmail run
```

## ワンタイムセットアップ

1.  `gog` が使用する **OAuthクライアントを所有するGCPプロジェクト** を選択します。

```bash
gcloud auth login
gcloud config set project <project-id>
```

注意: Gmail watchは、Pub/SubトピックがOAuthクライアントと同じプロジェクト内に存在することを要求します。

2.  APIを有効化:

```bash
gcloud services enable gmail.googleapis.com pubsub.googleapis.com
```

3.  トピックを作成:

```bash
gcloud pubsub topics create gog-gmail-watch
```

4.  Gmailプッシュがパブリッシュできるように許可:

```
gcloud pubsub topics add-iam-policy-binding gog-gmail-watch \
  --member=serviceAccount:gmail-api-push@system.gserviceaccount.com \
  --role=roles/pubsub.publisher
```

## ウォッチを開始

```
gog gmail watch start \
  --account openclaw@gmail.com \
  --label INBOX \
  --topic projects/<project-id>/topics/gog-gmail-watch
```

出力から `history_id` を保存します (デバッグ用)。

## プッシュハンドラーを実行

ローカル例 (共有トークン認証):

```
gog gmail watch serve \
  --account openclaw@gmail.com \
  --bind 127.0.0.1 \
  --port 8788 \
  --path /gmail-pubsub \
  --token <shared> \
  --hook-url http://127.0.0.1:18789/hooks/gmail \
  --hook-token OPENCLAW_HOOK_TOKEN \
  --include-body \
  --max-bytes 20000
```

注意点:

-   `--token` はプッシュエンドポイントを保護します (`x-gog-token` または `?token=`).
-   `--hook-url` はOpenClaw `/hooks/gmail` を指します (マッピング済み; 分離実行 + 要約をメインに送信).
-   `--include-body` と `--max-bytes` はOpenClawに送信される本文スニペットを制御します。

推奨: `openclaw webhooks gmail run` は同じフローをラップし、ウォッチを自動更新します。

## ハンドラーを公開 (上級者向け、非サポート)

Tailscale以外のトンネルが必要な場合は、手動で接続し、プッシュサブスクリプションで公開URLを使用します (非サポート、ガードレールなし):

```bash
cloudflared tunnel --url http://127.0.0.1:8788 --no-autoupdate
```

生成されたURLをプッシュエンドポイントとして使用:

```
gcloud pubsub subscriptions create gog-gmail-watch-push \
  --topic gog-gmail-watch \
  --push-endpoint "https://<public-url>/gmail-pubsub?token=<shared>"
```

本番環境: 安定したHTTPSエンドポイントを使用し、Pub/Sub OIDC JWTを設定してから実行:

```bash
gog gmail watch serve --verify-oidc --oidc-email <svc@...>
```

## テスト

監視対象の受信トレイにメッセージを送信:

```
gog gmail send \
  --account openclaw@gmail.com \
  --to openclaw@gmail.com \
  --subject "watch test" \
  --body "ping"
```

ウォッチ状態と履歴を確認:

```bash
gog gmail watch status --account openclaw@gmail.com
gog gmail history --account openclaw@gmail.com --since <historyId>
```

## トラブルシューティング

-   `Invalid topicName`: プロジェクト不一致 (トピックがOAuthクライアントプロジェクト内にない).
-   `User not authorized`: トピックに対する `roles/pubsub.publisher` が不足.
-   空のメッセージ: Gmailプッシュは `historyId` のみを提供します; `gog gmail history` 経由で取得.

## クリーンアップ

```bash
gog gmail watch stop --account openclaw@gmail.com
gcloud pubsub subscriptions delete gog-gmail-watch-push
gcloud pubsub topics delete gog-gmail-watch
```

[Webhooks](./webhook.md)[Polls](./poll.md)