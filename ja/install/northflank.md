

  ホスティングとデプロイ

  
# Northflankにデプロイ

ワンクリックテンプレートを使用してNorthflankにOpenClawをデプロイし、ブラウザでセットアップを完了します。これは「サーバー上でターミナルを操作しない」最も簡単な方法です。Northflankがゲートウェイを実行し、すべてを`/setup` Webウィザード経由で設定します。

## 始め方

1.  [Deploy OpenClaw](https://northflank.com/stacks/deploy-openclaw)をクリックしてテンプレートを開きます。
2.  まだお持ちでない場合は、[Northflankでアカウントを作成](https://app.northflank.com/signup)します。
3.  **Deploy OpenClaw now**をクリックします。
4.  必要な環境変数`SETUP_PASSWORD`を設定します。
5.  **Deploy stack**をクリックしてOpenClawテンプレートをビルド・実行します。
6.  デプロイが完了するのを待ち、**View resources**をクリックします。
7.  OpenClawサービスを開きます。
8.  OpenClawの公開URLを開き、`/setup`でセットアップを完了します。
9.  コントロールUIを`/openclaw`で開きます。

## 得られるもの

-   ホストされたOpenClawゲートウェイ + コントロールUI
-   `/setup`でのWebセットアップウィザード（ターミナルコマンド不要）
-   Northflankボリューム(`/data`)による永続ストレージ（設定/認証情報/ワークスペースは再デプロイ後も保持）

## セットアップの流れ

1.  `https://<your-northflank-domain>/setup`にアクセスし、`SETUP_PASSWORD`を入力します。
2.  モデル/認証プロバイダーを選択し、キーを貼り付けます。
3.  （オプション）Telegram/Discord/Slackトークンを追加します。
4.  **Run setup**をクリックします。
5.  コントロールUIを`https://<your-northflank-domain>/openclaw`で開きます。

Telegram DMがペアリング用に設定されている場合、セットアップウィザードがペアリングコードを承認できます。

## チャットトークンの取得方法

### Telegramボットトークン

1.  Telegramで`@BotFather`にメッセージを送る
2.  `/newbot`を実行する
3.  トークンをコピーする（例: `123456789:AA...`）
4.  `/setup`に貼り付ける

### Discordボットトークン

1.  [https://discord.com/developers/applications](https://discord.com/developers/applications)にアクセスする
2.  **New Application** → 名前を選択する
3.  **Bot** → **Add Bot**
4.  Bot → Privileged Gateway Intentsの下で**Enable MESSAGE CONTENT INTENT**を有効にする（必須。設定しないと起動時にクラッシュします）
5.  **Bot Token**をコピーして`/setup`に貼り付ける
6.  ボットをサーバーに招待する（OAuth2 URL Generator; スコープ: `bot`, `applications.commands`）

[Renderにデプロイ](./render.md)[開発チャンネル](./development-channels.md)

---