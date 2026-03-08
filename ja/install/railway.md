

  ホスティングとデプロイ

  
# Railwayにデプロイ

OpenClawをワンクリックテンプレートでRailwayにデプロイし、ブラウザでセットアップを完了します。これは「サーバー上でターミナルを使わない」最も簡単な方法です。Railwayがゲートウェイを実行し、すべてを`/setup`ウェブウィザード経由で設定します。

## クイックチェックリスト（新規ユーザー）

1.  **Deploy on Railway**（下記）をクリック。
2.  `/data`にマウントされた**ボリューム**を追加。
3.  必要な**変数**（少なくとも`SETUP_PASSWORD`）を設定。
4.  ポート`8080`で**HTTP プロキシ**を有効化。
5.  `https://<あなたの-railway-ドメイン>/setup`を開き、ウィザードを完了。

## ワンクリックデプロイ

[Deploy on Railway](https://railway.com/deploy/clawdbot-railway-template) デプロイ後、**Railway → あなたのサービス → Settings → Domains**で公開URLを確認してください。Railwayは以下のいずれかを提供します：

-   生成されたドメイン（多くの場合`https://<何か>.up.railway.app`）、または
-   カスタムドメインを接続している場合はそれを使用。

その後、以下を開きます：

-   `https://<あなたの-railway-ドメイン>/setup` — セットアップウィザード（パスワード保護）
-   `https://<あなたの-railway-ドメイン>/openclaw` — コントロールUI

## 得られるもの

-   ホストされたOpenClawゲートウェイ + コントロールUI
-   `/setup`のウェブセットアップウィザード（ターミナルコマンド不要）
-   Railwayボリューム（`/data`）による永続ストレージ（設定/認証情報/ワークスペースが再デプロイ後も保持）
-   `/setup/export`でのバックアップエクスポート（後でRailwayから移行可能）

## 必要なRailway設定

### パブリックネットワーキング

サービスの**HTTP プロキシ**を有効化します。

-   ポート: `8080`

### ボリューム（必須）

以下のパスにマウントされたボリュームを接続します：

-   `/data`

### 変数

サービスに以下の変数を設定します：

-   `SETUP_PASSWORD`（必須）
-   `PORT=8080`（必須 — パブリックネットワーキングのポートと一致させる必要があります）
-   `OPENCLAW_STATE_DIR=/data/.openclaw`（推奨）
-   `OPENCLAW_WORKSPACE_DIR=/data/workspace`（推奨）
-   `OPENCLAW_GATEWAY_TOKEN`（推奨；管理者シークレットとして扱ってください）

## セットアップの流れ

1.  `https://<あなたの-railway-ドメイン>/setup`にアクセスし、`SETUP_PASSWORD`を入力。
2.  モデル/認証プロバイダーを選択し、キーを貼り付け。
3.  （オプション）Telegram/Discord/Slackトークンを追加。
4.  **Run setup**をクリック。

Telegram DMがペアリング用に設定されている場合、セットアップウィザードがペアリングコードを承認できます。

## チャットトークンの取得方法

### Telegramボットトークン

1.  Telegramで`@BotFather`にメッセージ
2.  `/newbot`を実行
3.  トークンをコピー（`123456789:AA...`のような形式）
4.  `/setup`に貼り付け

### Discordボットトークン

1.  [https://discord.com/developers/applications](https://discord.com/developers/applications)にアクセス
2.  **New Application** → 名前を選択
3.  **Bot** → **Add Bot**
4.  Bot → Privileged Gateway Intents の下で**Enable MESSAGE CONTENT INTENT**を有効化（必須、そうでないと起動時にクラッシュします）
5.  **Bot Token**をコピーし、`/setup`に貼り付け
6.  ボットをサーバーに招待（OAuth2 URL Generator; スコープ: `bot`, `applications.commands`）

## バックアップと移行

以下の場所でバックアップをダウンロード：

-   `https://<あなたの-railway-ドメイン>/setup/export`

これによりOpenClawの状態とワークスペースがエクスポートされ、設定やメモリを失うことなく別のホストに移行できます。

[exe.dev](./exe-dev.md)[Renderにデプロイ](./render.md)