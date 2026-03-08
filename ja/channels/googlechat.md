

  メッセージングプラットフォーム

  
# Google Chat

ステータス: Google Chat API Webhook（HTTPのみ）によるDMおよびスペース対応準備完了。

## クイックセットアップ（初心者向け）

1.  Google Cloudプロジェクトを作成し、**Google Chat API**を有効にします。
    -   アクセス先: [Google Chat API 認証情報](https://console.cloud.google.com/apis/api/chat.googleapis.com/credentials)
    -   まだ有効になっていない場合はAPIを有効にします。
2.  **サービスアカウント**を作成します:
    -   **認証情報を作成** > **サービスアカウント**を押します。
    -   任意の名前を付けます（例: `openclaw-chat`）。
    -   権限は空白のままにします（**続行**を押します）。
    -   アクセス権を持つプリンシパルも空白のままにします（**完了**を押します）。
3.  **JSONキー**を作成してダウンロードします:
    -   サービスアカウントの一覧で、作成したばかりのアカウントをクリックします。
    -   **キー**タブに移動します。
    -   **キーを追加** > **新しいキーを作成**をクリックします。
    -   **JSON**を選択し、**作成**を押します。
4.  ダウンロードしたJSONファイルをゲートウェイホストに保存します（例: `~/.openclaw/googlechat-service-account.json`）。
5.  [Google Cloud Console Chat 設定](https://console.cloud.google.com/apis/api/chat.googleapis.com/hangouts-chat)でGoogle Chatアプリを作成します:
    -   **アプリケーション情報**を入力します:
        -   **アプリ名**: (例: `OpenClaw`)
        -   **アバターURL**: (例: `https://openclaw.ai/logo.png`)
        -   **説明**: (例: `Personal AI Assistant`)
    -   **インタラクティブ機能**を有効にします。
    -   **機能**で、**スペースとグループ会話に参加**にチェックを入れます。
    -   **接続設定**で、**HTTPエンドポイントURL**を選択します。
    -   **トリガー**で、**すべてのトリガーに共通のHTTPエンドポイントURLを使用**を選択し、ゲートウェイの公開URLの後に`/googlechat`を付けたものを設定します。
        -   *ヒント: `openclaw status`を実行してゲートウェイの公開URLを確認してください。*
    -   **公開範囲**で、**このChatアプリを<あなたのドメイン>内の特定のユーザーとグループが利用できるようにする**にチェックを入れます。
    -   テキストボックスにあなたのメールアドレスを入力します（例: `user@example.com`）。
    -   下部の**保存**をクリックします。
6.  **アプリのステータスを有効にします**:
    -   保存後、**ページを更新**します。
    -   **アプリステータス**セクションを探します（通常、保存後に上部または下部近くにあります）。
    -   ステータスを**ライブ - ユーザーが利用可能**に変更します。
    -   再度**保存**をクリックします。
7.  サービスアカウントのパスとWebhookのオーディエンスを指定してOpenClawを設定します:
    -   環境変数: `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE=/path/to/service-account.json`
    -   または設定ファイル: `channels.googlechat.serviceAccountFile: "/path/to/service-account.json"`。
8.  Webhookオーディエンスのタイプと値を設定します（Chatアプリの設定と一致させる）。
9.  ゲートウェイを起動します。Google ChatがあなたのWebhookパスにPOSTリクエストを送信します。

## Google Chatへの追加

ゲートウェイが起動し、あなたのメールアドレスが公開範囲リストに追加されたら:

1.  [Google Chat](https://chat.google.com/)にアクセスします。
2.  **ダイレクトメッセージ**の横にある **+**（プラス）アイコンをクリックします。
3.  検索バー（通常は人を追加する場所）に、Google Cloud Consoleで設定した**アプリ名**を入力します。
    -   **注意**: このボットはプライベートアプリであるため、「マーケットプレイス」の閲覧リストには表示されません。名前で検索する必要があります。
4.  検索結果からあなたのボットを選択します。
5.  **追加**または**チャット**をクリックして1対1の会話を開始します。
6.  「Hello」と送信してアシスタントを起動します！

## 公開URL（Webhook専用）

Google Chat Webhookは公開されたHTTPSエンドポイントを必要とします。セキュリティのため、**`/googlechat`パスのみ**をインターネットに公開してください。OpenClawダッシュボードやその他の機密性の高いエンドポイントはプライベートネットワーク上に保持します。

### オプションA: Tailscale Funnel（推奨）

プライベートダッシュボードにはTailscale Serveを、公開WebhookパスにはFunnelを使用します。これにより、`/`はプライベートに保ちながら、`/googlechat`のみを公開できます。

1.  **ゲートウェイがバインドされているアドレスを確認します:**
    
    Copy
    
    ```bash
    ss -tlnp | grep 18789
    ```
    
    IPアドレスをメモします（例: `127.0.0.1`、`0.0.0.0`、またはTailscale IP `100.x.x.x`など）。
2.  **ダッシュボードをtailnet内のみに公開します（ポート8443）:**
    
    Copy
    
    ```bash
    # localhostにバインドされている場合（127.0.0.1 または 0.0.0.0）:
    tailscale serve --bg --https 8443 http://127.0.0.1:18789
    
    # Tailscale IPのみにバインドされている場合（例: 100.106.161.80）:
    tailscale serve --bg --https 8443 http://100.106.161.80:18789
    ```
    
3.  **Webhookパスのみを公開します:**
    
    Copy
    
    ```bash
    # localhostにバインドされている場合（127.0.0.1 または 0.0.0.0）:
    tailscale funnel --bg --set-path /googlechat http://127.0.0.1:18789/googlechat
    
    # Tailscale IPのみにバインドされている場合（例: 100.106.161.80）:
    tailscale funnel --bg --set-path /googlechat http://100.106.161.80:18789/googlechat
    ```
    
4.  **Funnelアクセスのノードを承認します:** プロンプトが表示されたら、出力に表示される承認URLにアクセスし、tailnetポリシーでこのノードのFunnelを有効にします。
5.  **設定を確認します:**
    
    Copy
    
    ```
    tailscale serve status
    tailscale funnel status
    ```
    

公開Webhook URLは次のようになります: `https://<node-name>..ts.net/googlechat`
プライベートダッシュボードはtailnet内のみでアクセス可能です: `https://<node-name>..ts.net:8443/`
Google Chatアプリの設定では、公開URL（`:8443`なし）を使用してください。

> 注: この設定は再起動後も保持されます。後で削除するには、`tailscale funnel reset` と `tailscale serve reset` を実行します。

### オプションB: リバースプロキシ（Caddy）

Caddyなどのリバースプロキシを使用する場合は、特定のパスのみをプロキシします:

```
your-domain.com {
    reverse_proxy /googlechat* localhost:18789
}
```

この設定では、`your-domain.com/`へのリクエストは無視されるか404が返され、`your-domain.com/googlechat`は安全にOpenClawにルーティングされます。

### オプションC: Cloudflare Tunnel

トンネルのイングレスルールを設定して、Webhookパスのみをルーティングします:

-   **パス**: `/googlechat` -> `http://localhost:18789/googlechat`
-   **デフォルトルール**: HTTP 404 (Not Found)

## 仕組み

1.  Google ChatはWebhook POSTをゲートウェイに送信します。各リクエストには`Authorization: Bearer `ヘッダーが含まれます。
    -   OpenClawは、ヘッダーが存在する場合、完全なWebhookボディを読み取り/解析する前にベアラー認証を検証します。
    -   ボディ内に`authorizationEventObject.systemIdToken`を含むGoogle Workspaceアドオンリクエストは、より厳格な事前認証ボディ予算を介してサポートされます。
2.  OpenClawは、設定された`audienceType` + `audience`に対してトークンを検証します:
    -   `audienceType: "app-url"` → オーディエンスはあなたのHTTPS Webhook URLです。
    -   `audienceType: "project-number"` → オーディエンスはCloudプロジェクト番号です。
3.  メッセージはスペースごとにルーティングされます:
    -   DMはセッションキー`agent::googlechat:dm:`を使用します。
    -   スペースはセッションキー`agent::googlechat:group:`を使用します。
4.  DMアクセスはデフォルトでペアリングです。未知の送信者にはペアリングコードが送信されます。承認は以下で行います:
    -   `openclaw pairing approve googlechat `
5.  グループスペースはデフォルトで@メンションを必要とします。メンション検出にアプリのユーザー名が必要な場合は`botUser`を使用してください。

## ターゲット

配信と許可リストには以下の識別子を使用します:

-   ダイレクトメッセージ: `users/`（推奨）。
-   生のメールアドレス`name@example.com`は可変であり、`channels.googlechat.dangerouslyAllowNameMatching: true`の場合にのみダイレクト許可リストマッチングに使用されます。
-   非推奨: `users/`はメールアドレスの許可リストではなく、ユーザーIDとして扱われます。
-   スペース: `spaces/`。

## 設定のハイライト

```json
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      // または serviceAccountRef: { source: "file", provider: "filemain", id: "/channels/googlechat/serviceAccount" }
      audienceType: "app-url",
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890", // オプション; メンション検出に役立ちます
      dm: {
        policy: "pairing",
        allowFrom: ["users/1234567890"],
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": {
          allow: true,
          requireMention: true,
          users: ["users/1234567890"],
          systemPrompt: "Short answers only.",
        },
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```

注記:

-   サービスアカウントの認証情報は、`serviceAccount`（JSON文字列）でインラインで渡すこともできます。
-   `serviceAccountRef`もサポートされています（環境変数/ファイル SecretRef）、`channels.googlechat.accounts..serviceAccountRef`の下のアカウントごとの参照も含みます。
-   `webhookPath`が設定されていない場合、デフォルトのWebhookパスは`/googlechat`です。
-   `dangerouslyAllowNameMatching`は、許可リストのための可変メールアドレスプリンシパルマッチングを再び有効にします（緊急互換モード）。
-   リアクションは、`actions.reactions`が有効な場合、`reactions`ツールと`channels action`で利用可能です。
-   `typingIndicator`は`none`、`message`（デフォルト）、`reaction`（リアクションにはユーザーOAuthが必要）をサポートします。
-   添付ファイルはChat APIを介してダウンロードされ、メディアパイプラインに保存されます（サイズは`mediaMaxMb`で制限されます）。

シークレット参照の詳細: [シークレット管理](../gateway/secrets.md)。

## トラブルシューティング

### 405 Method Not Allowed

Google Cloud Logs Explorerに以下のようなエラーが表示される場合:

```bash
status code: 405, reason phrase: HTTP error response: HTTP/1.1 405 Method Not Allowed
```

これはWebhookハンドラーが登録されていないことを意味します。一般的な原因:

1.  **チャネルが設定されていない**: 設定に`channels.googlechat`セクションがありません。以下で確認します:
    
    Copy
    
    ```bash
    openclaw config get channels.googlechat
    ```
    
    「Config path not found」が返された場合は、設定を追加してください（[設定のハイライト](#config-highlights)を参照）。
2.  **プラグインが有効になっていない**: プラグインのステータスを確認します:
    
    Copy
    
    ```bash
    openclaw plugins list | grep googlechat
    ```
    
    「disabled」と表示される場合は、設定に`plugins.entries.googlechat.enabled: true`を追加してください。
3.  **ゲートウェイが再起動されていない**: 設定を追加した後、ゲートウェイを再起動します:
    
    Copy
    
    ```bash
    openclaw gateway restart
    ```
    

チャネルが実行されていることを確認します:

```bash
openclaw channels status
# 次のように表示されるはずです: Google Chat default: enabled, configured, ...
```

### その他の問題

-   `openclaw channels status --probe`を実行して、認証エラーやオーディエンス設定の欠落を確認します。
-   メッセージが届かない場合は、ChatアプリのWebhook URLとイベントサブスクリプションを確認してください。
-   メンションゲートにより返信がブロックされる場合は、`botUser`をアプリのユーザーリソース名に設定し、`requireMention`を確認してください。
-   テストメッセージを送信しながら`openclaw logs --follow`を使用して、リクエストがゲートウェイに到達しているか確認します。

関連ドキュメント:

-   [ゲートウェイ設定](../gateway/configuration.md)
-   [セキュリティ](../gateway/security.md)
-   [リアクション](../tools/reactions.md)

[Feishu](./feishu.md)[iMessage](./imessage.md)