

  ホスティングとデプロイメント

  
# GCP

## 目標

Docker を使用して GCP Compute Engine VM 上で永続的な OpenClaw Gateway を実行し、耐久性のある状態、組み込み済みバイナリ、安全な再起動動作を実現します。「月額約 $5-12 で OpenClaw を 24/7 稼働」させたい場合、これは Google Cloud 上での信頼性の高いセットアップです。価格はマシンタイプとリージョンによって異なります。ワークロードに合った最小の VM を選択し、OOM が発生した場合はスケールアップしてください。

## 何をするのか（簡単な説明）

-   GCP プロジェクトを作成し、課金を有効化する
-   Compute Engine VM を作成する
-   Docker（分離されたアプリケーションランタイム）をインストールする
-   Docker 内で OpenClaw Gateway を起動する
-   ホスト上に `~/.openclaw` + `~/.openclaw/workspace` を永続化する（再起動/リビルド後も生存）
-   SSH トンネル経由でラップトップから Control UI にアクセスする

ゲートウェイには以下の方法でアクセスできます：

-   ラップトップからの SSH ポートフォワーディング
-   ファイアウォールとトークンを自身で管理する場合の直接ポート公開

このガイドでは、GCP Compute Engine 上の Debian を使用します。Ubuntu でも動作します。パッケージを適宜マッピングしてください。一般的な Docker フローについては、[Docker](./docker.md) を参照してください。

* * *

## クイックパス（経験豊富なオペレーター向け）

1.  GCP プロジェクト作成 + Compute Engine API 有効化
2.  Compute Engine VM 作成 (e2-small, Debian 12, 20GB)
3.  VM に SSH 接続
4.  Docker インストール
5.  OpenClaw リポジトリをクローン
6.  永続的なホストディレクトリを作成
7.  `.env` と `docker-compose.yml` を設定
8.  必要なバイナリを組み込み、ビルドして起動

* * *

## 必要なもの

-   GCP アカウント (e2-micro で無料枠対象)
-   gcloud CLI がインストール済み（または Cloud Console を使用）
-   ラップトップからの SSH アクセス
-   SSH + コピー/ペーストの基本的な操作に慣れていること
-   ~20-30 分
-   Docker と Docker Compose
-   モデル認証情報
-   オプションのプロバイダー認証情報
    -   WhatsApp QR
    -   Telegram ボットトークン
    -   Gmail OAuth

* * *

## 1) gcloud CLI をインストール（または Console を使用）

**オプション A: gcloud CLI**（自動化におすすめ） [https://cloud.google.com/sdk/docs/install](https://cloud.google.com/sdk/docs/install) からインストールし、初期化と認証を行います：

```bash
gcloud init
gcloud auth login
```

**オプション B: Cloud Console** すべての手順は、[https://console.cloud.google.com](https://console.cloud.google.com) のウェブ UI で実行できます。

* * *

## 2) GCP プロジェクトを作成

**CLI:**

```bash
gcloud projects create my-openclaw-project --name="OpenClaw Gateway"
gcloud config set project my-openclaw-project
```

[https://console.cloud.google.com/billing](https://console.cloud.google.com/billing) で課金を有効化します（Compute Engine に必須）。Compute Engine API を有効化します：

```bash
gcloud services enable compute.googleapis.com
```

**Console:**

1.  IAM & Admin > プロジェクトを作成 に移動
2.  名前を付けて作成
3.  プロジェクトの課金を有効化
4.  API とサービス > API を有効化 > 「Compute Engine API」を検索 > 有効化

* * *

## 3) VM を作成

**マシンタイプ:**

| タイプ | 仕様 | コスト | 備考 |
| --- | --- | --- | --- |
| e2-medium | 2 vCPU, 4GB RAM | ~$25/月 | ローカル Docker ビルドで最も信頼性が高い |
| e2-small | 2 vCPU, 2GB RAM | ~$12/月 | Docker ビルドに推奨される最小構成 |
| e2-micro | 2 vCPU (共有), 1GB RAM | 無料枠対象 | Docker ビルドで OOM (exit 137) が頻発 |

**CLI:**

```
gcloud compute instances create openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-small \
  --boot-disk-size=20GB \
  --image-family=debian-12 \
  --image-project=debian-cloud
```

**Console:**

1.  Compute Engine > VM インスタンス > インスタンスを作成 に移動
2.  名前: `openclaw-gateway`
3.  リージョン: `us-central1`, ゾーン: `us-central1-a`
4.  マシンタイプ: `e2-small`
5.  ブートディスク: Debian 12, 20GB
6.  作成

* * *

## 4) VM に SSH 接続

**CLI:**

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a
```

**Console:** Compute Engine ダッシュボードで VM の横にある「SSH」ボタンをクリックします。注: SSH キーの伝播は VM 作成後 1-2 分かかることがあります。接続が拒否された場合は、待ってから再試行してください。

* * *

## 5) Docker をインストール（VM 上）

```bash
sudo apt-get update
sudo apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
```

グループ変更を反映させるためにログアウトし、再度ログインします：

```
exit
```

その後、再度 SSH 接続します：

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a
```

確認：

```bash
docker --version
docker compose version
```

* * *

## 6) OpenClaw リポジトリをクローン

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

このガイドでは、バイナリの永続性を保証するためにカスタムイメージをビルドすることを前提としています。

* * *

## 7) 永続的なホストディレクトリを作成

Docker コンテナは一時的なものです。すべての長寿命な状態はホスト上に存在する必要があります。

```bash
mkdir -p ~/.openclaw
mkdir -p ~/.openclaw/workspace
```

* * *

## 8) 環境変数を設定

リポジトリルートに `.env` を作成します。

```
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-now
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/home/$USER/.openclaw
OPENCLAW_WORKSPACE_DIR=/home/$USER/.openclaw/workspace

GOG_KEYRING_PASSWORD=change-me-now
XDG_CONFIG_HOME=/home/node/.openclaw
```

強力なシークレットを生成します：

```bash
openssl rand -hex 32
```

**このファイルをコミットしないでください。**

* * *

## 9) Docker Compose 設定

`docker-compose.yml` を作成または更新します。

```
services:
  openclaw-gateway:
    image: ${OPENCLAW_IMAGE}
    build: .
    restart: unless-stopped
    env_file:
      - .env
    environment:
      - HOME=/home/node
      - NODE_ENV=production
      - TERM=xterm-256color
      - OPENCLAW_GATEWAY_BIND=${OPENCLAW_GATEWAY_BIND}
      - OPENCLAW_GATEWAY_PORT=${OPENCLAW_GATEWAY_PORT}
      - OPENCLAW_GATEWAY_TOKEN=${OPENCLAW_GATEWAY_TOKEN}
      - GOG_KEYRING_PASSWORD=${GOG_KEYRING_PASSWORD}
      - XDG_CONFIG_HOME=${XDG_CONFIG_HOME}
      - PATH=/home/linuxbrew/.linuxbrew/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    volumes:
      - ${OPENCLAW_CONFIG_DIR}:/home/node/.openclaw
      - ${OPENCLAW_WORKSPACE_DIR}:/home/node/.openclaw/workspace
    ports:
      # 推奨: ゲートウェイを VM 上でループバック専用に保ち、SSH トンネル経由でアクセスします。
      # 公開する場合は、`127.0.0.1:` プレフィックスを削除し、ファイアウォールを適切に設定してください。
      - "127.0.0.1:${OPENCLAW_GATEWAY_PORT}:18789"
    command:
      [
        "node",
        "dist/index.js",
        "gateway",
        "--bind",
        "${OPENCLAW_GATEWAY_BIND}",
        "--port",
        "${OPENCLAW_GATEWAY_PORT}",
      ]
```

* * *

## 10) 必要なバイナリをイメージに組み込む（重要）

実行中のコンテナ内でバイナリをインストールすることは罠です。ランタイムでインストールされたものは、再起動時に失われます。スキルが必要とするすべての外部バイナリは、イメージビルド時にインストールする必要があります。以下の例は、3つの一般的なバイナリのみを示しています：

-   Gmail アクセスのための `gog`
-   Google Places のための `goplaces`
-   WhatsApp のための `wacli`

これらは例であり、完全なリストではありません。同じパターンを使用して、必要なだけ多くのバイナリをインストールできます。後で追加のバイナリに依存する新しいスキルを追加する場合は、以下を行う必要があります：

1.  Dockerfile を更新
2.  イメージをリビルド
3.  コンテナを再起動

**Dockerfile の例**

```dockerfile
FROM node:22-bookworm

RUN apt-get update && apt-get install -y socat && rm -rf /var/lib/apt/lists/*

# 例 1: Gmail CLI
RUN curl -L https://github.com/steipete/gog/releases/latest/download/gog_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/gog

# 例 2: Google Places CLI
RUN curl -L https://github.com/steipete/goplaces/releases/latest/download/goplaces_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/goplaces

# 例 3: WhatsApp CLI
RUN curl -L https://github.com/steipete/wacli/releases/latest/download/wacli_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/wacli

# 同じパターンを使用して、以下にさらにバイナリを追加

WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN corepack enable
RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

* * *

## 11) ビルドと起動

```bash
docker compose build
docker compose up -d openclaw-gateway
```

ビルドが `pnpm install --frozen-lockfile` 中に `Killed` / `exit code 137` で失敗する場合、VM のメモリ不足です。最初のビルドをより確実に行うには、最小でも `e2-small`、または `e2-medium` を使用してください。LAN にバインドする場合 (`OPENCLAW_GATEWAY_BIND=lan`)、続行する前に信頼できるブラウザオリジンを設定します：

```bash
docker compose run --rm openclaw-cli config set gateway.controlUi.allowedOrigins '["http://127.0.0.1:18789"]' --strict-json
```

ゲートウェイポートを変更した場合は、`18789` を設定したポートに置き換えてください。バイナリを確認します：

```bash
docker compose exec openclaw-gateway which gog
docker compose exec openclaw-gateway which goplaces
docker compose exec openclaw-gateway which wacli
```

期待される出力：

```
/usr/local/bin/gog
/usr/local/bin/goplaces
/usr/local/bin/wacli
```

* * *

## 12) ゲートウェイを確認

```bash
docker compose logs -f openclaw-gateway
```

成功：

```ini
[gateway] listening on ws://0.0.0.0:18789
```

* * *

## 13) ラップトップからアクセス

ゲートウェイポートを転送する SSH トンネルを作成します：

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a -- -L 18789:127.0.0.1:18789
```

ブラウザで開きます：`http://127.0.0.1:18789/` 新しいトークン化されたダッシュボードリンクを取得します：

```bash
docker compose run --rm openclaw-cli dashboard --no-open
```

その URL からトークンを貼り付けます。Control UI が `unauthorized` または `disconnected (1008): pairing required` と表示する場合は、ブラウザデバイスを承認します：

```bash
docker compose run --rm openclaw-cli devices list
docker compose run --rm openclaw-cli devices approve <requestId>
```

* * *

## 何がどこに永続化されるか（信頼できる情報源）

OpenClaw は Docker 内で実行されますが、Docker は信頼できる情報源ではありません。すべての長寿命な状態は、再起動、リビルド、リブート後も生存する必要があります。

| コンポーネント | 場所 | 永続化メカニズム | 備考 |
| --- | --- | --- | --- |
| ゲートウェイ設定 | `/home/node/.openclaw/` | ホストボリュームマウント | `openclaw.json`、トークンを含む |
| モデル認証プロファイル | `/home/node/.openclaw/` | ホストボリュームマウント | OAuth トークン、API キー |
| スキル設定 | `/home/node/.openclaw/skills/` | ホストボリュームマウント | スキルレベルの状態 |
| エージェントワークスペース | `/home/node/.openclaw/workspace/` | ホストボリュームマウント | コードとエージェント成果物 |
| WhatsApp セッション | `/home/node/.openclaw/` | ホストボリュームマウント | QR ログインを保持 |
| Gmail キーリング | `/home/node/.openclaw/` | ホストボリューム + パスワード | `GOG_KEYRING_PASSWORD` が必要 |
| 外部バイナリ | `/usr/local/bin/` | Docker イメージ | ビルド時に組み込む必要あり |
| Node ランタイム | コンテナファイルシステム | Docker イメージ | イメージビルドごとにリビルド |
| OS パッケージ | コンテナファイルシステム | Docker イメージ | ランタイムでインストールしない |
| Docker コンテナ | 一時的 | 再起動可能 | 破棄しても安全 |

* * *

## 更新

VM 上の OpenClaw を更新するには：

```bash
cd ~/openclaw
git pull
docker compose build
docker compose up -d
```

* * *

## トラブルシューティング

**SSH 接続拒否** SSH キーの伝播は VM 作成後 1-2 分かかることがあります。待ってから再試行してください。**OS Login の問題** OS Login プロファイルを確認します：

```bash
gcloud compute os-login describe-profile
```

アカウントに必要な IAM 権限（Compute OS Login または Compute OS Admin Login）があることを確認してください。**メモリ不足 (OOM)** Docker ビルドが `Killed` と `exit code 137` で失敗する場合、VM が OOM キルされました。e2-small（最小）または e2-medium（信頼性の高いローカルビルドに推奨）にアップグレードしてください：

```bash
# まず VM を停止
gcloud compute instances stop openclaw-gateway --zone=us-central1-a

# マシンタイプを変更
gcloud compute instances set-machine-type openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-small

# VM を起動
gcloud compute instances start openclaw-gateway --zone=us-central1-a
```

* * *

## サービスアカウント（セキュリティのベストプラクティス）

個人使用の場合、デフォルトのユーザーアカウントで十分です。自動化や CI/CD パイプラインの場合は、最小限の権限を持つ専用のサービスアカウントを作成します：

1.  サービスアカウントを作成：
    
    Copy
    
    ```
    gcloud iam service-accounts create openclaw-deploy \
      --display-name="OpenClaw Deployment"
    ```
    
2.  Compute Instance Admin ロール（またはより限定されたカスタムロール）を付与：
    
    Copy
    
    ```
    gcloud projects add-iam-policy-binding my-openclaw-project \
      --member="serviceAccount:openclaw-deploy@my-openclaw-project.iam.gserviceaccount.com" \
      --role="roles/compute.instanceAdmin.v1"
    ```
    

自動化に Owner ロールを使用しないでください。最小権限の原則を使用します。IAM ロールの詳細については、[https://cloud.google.com/iam/docs/understanding-roles](https://cloud.google.com/iam/docs/understanding-roles) を参照してください。

* * *

## 次のステップ

-   メッセージングチャネルを設定：[チャネル](../channels.md)
-   ローカルデバイスをノードとしてペアリング：[ノード](../nodes.md)
-   ゲートウェイを設定：[ゲートウェイ設定](../gateway/configuration.md)

[Hetzner](./hetzner.md)[macOS VMs](./macos-vm.md)