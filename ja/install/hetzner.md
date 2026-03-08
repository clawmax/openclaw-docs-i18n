

  ホスティングとデプロイ

  
# Hetzner

## 目標

Dockerを使用してHetzner VPS上で永続的なOpenClaw Gatewayを実行し、耐久性のある状態、組み込み済みバイナリ、安全な再起動動作を実現します。「約5ドルでOpenClawを24/7稼働」させたい場合、これが最もシンプルで信頼性の高いセットアップです。Hetznerの価格は変動します。最小のDebian/Ubuntu VPSを選択し、OOMが発生した場合はスケールアップしてください。セキュリティモデルの注意点:

-   会社で共有するエージェントは、全員が同じ信頼境界内にあり、ランタイムが業務専用である場合に問題ありません。
-   厳格な分離を維持する: 専用のVPS/ランタイム + 専用アカウント。そのホスト上に個人のApple/Google/ブラウザ/パスワードマネージャーのプロファイルを置かない。
-   ユーザー同士が敵対的な関係にある場合は、ゲートウェイ/ホスト/OSユーザーごとに分割してください。

[セキュリティ](../gateway/security.md)と[VPSホスティング](../vps.md)を参照してください。

## 何をするのか（簡単な説明）？

-   小さなLinuxサーバーをレンタルする（Hetzner VPS）
-   Dockerをインストールする（分離されたアプリケーションランタイム）
-   Docker内でOpenClaw Gatewayを起動する
-   ホスト上に`~/.openclaw` + `~/.openclaw/workspace`を永続化する（再起動/リビルド後も生存）
-   ラップトップからSSHトンネル経由でControl UIにアクセスする

Gatewayには以下の方法でアクセスできます:

-   ラップトップからのSSHポートフォワーディング
-   ファイアウォールとトークンを自身で管理する場合は、ポートを直接公開

このガイドはHetzner上のUbuntuまたはDebianを想定しています。  
他のLinux VPSを使用している場合は、パッケージを適宜読み替えてください。一般的なDockerの流れについては、[Docker](./docker.md)を参照してください。

* * *

## クイックパス（経験豊富なオペレーター向け）

1.  Hetzner VPSをプロビジョニング
2.  Dockerをインストール
3.  OpenClawリポジトリをクローン
4.  永続的なホストディレクトリを作成
5.  `.env`と`docker-compose.yml`を設定
6.  必要なバイナリをイメージに組み込む
7.  `docker compose up -d`
8.  永続性とGatewayアクセスを検証

* * *

## 必要なもの

-   ルートアクセス権のあるHetzner VPS
-   ラップトップからのSSHアクセス
-   SSHとコピー/ペーストの基本的な操作に慣れていること
-   ~20分
-   DockerとDocker Compose
-   モデル認証情報
-   オプションのプロバイダー認証情報
    -   WhatsApp QRコード
    -   Telegramボットトークン
    -   Gmail OAuth

* * *

## 1) VPSをプロビジョニングする

HetznerでUbuntuまたはDebianのVPSを作成します。rootとして接続:

```bash
ssh root@YOUR_VPS_IP
```

このガイドはVPSがステートフルであることを想定しています。使い捨てのインフラとして扱わないでください。

* * *

## 2) Dockerをインストールする（VPS上で）

```bash
apt-get update
apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sh
```

確認:

```bash
docker --version
docker compose version
```

* * *

## 3) OpenClawリポジトリをクローンする

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

このガイドでは、バイナリの永続性を保証するためにカスタムイメージをビルドすることを想定しています。

* * *

## 4) 永続的なホストディレクトリを作成する

Dockerコンテナはエフェメラルです。長期間存続する状態はすべてホスト上に存在する必要があります。

```bash
mkdir -p /root/.openclaw/workspace

# コンテナユーザー (uid 1000) に所有権を設定:
chown -R 1000:1000 /root/.openclaw
```

* * *

## 5) 環境変数を設定する

リポジトリルートに`.env`を作成します。

```
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-now
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/root/.openclaw
OPENCLAW_WORKSPACE_DIR=/root/.openclaw/workspace

GOG_KEYRING_PASSWORD=change-me-now
XDG_CONFIG_HOME=/home/node/.openclaw
```

強力なシークレットを生成:

```bash
openssl rand -hex 32
```

**このファイルをコミットしないでください。**

* * *

## 6) Docker Compose設定

`docker-compose.yml`を作成または更新します。

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
      # 推奨: VPS上ではGatewayをループバック専用に保ち、SSHトンネル経由でアクセス。
      # 公開する場合は、`127.0.0.1:`プレフィックスを削除し、適切にファイアウォールを設定。
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
        "--allow-unconfigured",
      ]
```

`--allow-unconfigured`はブートストラップの便宜のためだけのものであり、適切なゲートウェイ設定の代わりにはなりません。それでも認証（`gateway.auth.token`またはパスワード）を設定し、デプロイメントに安全なバインド設定を使用してください。

* * *

## 7) 必要なバイナリをイメージに組み込む（重要）

実行中のコンテナ内でバイナリをインストールすることは罠です。ランタイムでインストールされたものは、再起動時に失われます。スキルが必要とするすべての外部バイナリは、イメージビルド時にインストールする必要があります。以下の例は、3つの一般的なバイナリのみを示しています:

-   Gmailアクセスのための`gog`
-   Google Placesのための`goplaces`
-   WhatsAppのための`wacli`

これらは例であり、完全なリストではありません。同じパターンを使用して、必要に応じていくつでもバイナリをインストールできます。後で追加のバイナリに依存する新しいスキルを追加する場合は、以下を行う必要があります:

1.  Dockerfileを更新
2.  イメージをリビルド
3.  コンテナを再起動

**Dockerfileの例**

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

# 同じパターンを使用して、以下により多くのバイナリを追加

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

## 8) ビルドと起動

```bash
docker compose build
docker compose up -d openclaw-gateway
```

バイナリを確認:

```bash
docker compose exec openclaw-gateway which gog
docker compose exec openclaw-gateway which goplaces
docker compose exec openclaw-gateway which wacli
```

期待される出力:

```
/usr/local/bin/gog
/usr/local/bin/goplaces
/usr/local/bin/wacli
```

* * *

## 9) Gatewayを検証する

```bash
docker compose logs -f openclaw-gateway
```

成功:

```ini
[gateway] listening on ws://0.0.0.0:18789
```

ラップトップから:

```bash
ssh -N -L 18789:127.0.0.1:18789 root@YOUR_VPS_IP
```

開く: `http://127.0.0.1:18789/` ゲートウェイトークンを貼り付けます。

* * *

## 何がどこに永続化されるか（信頼できる情報源）

OpenClawはDocker内で実行されますが、Dockerは信頼できる情報源ではありません。すべての長期間存続する状態は、再起動、リビルド、リブート後も生存する必要があります。

| コンポーネント | 場所 | 永続化メカニズム | 備考 |
| --- | --- | --- | --- |
| Gateway設定 | `/home/node/.openclaw/` | ホストボリュームマウント | `openclaw.json`、トークンを含む |
| モデル認証プロファイル | `/home/node/.openclaw/` | ホストボリュームマウント | OAuthトークン、APIキー |
| スキル設定 | `/home/node/.openclaw/skills/` | ホストボリュームマウント | スキルレベルの状態 |
| エージェントワークスペース | `/home/node/.openclaw/workspace/` | ホストボリュームマウント | コードとエージェント成果物 |
| WhatsAppセッション | `/home/node/.openclaw/` | ホストボリュームマウント | QRログインを保持 |
| Gmailキーリング | `/home/node/.openclaw/` | ホストボリューム + パスワード | `GOG_KEYRING_PASSWORD`が必要 |
| 外部バイナリ | `/usr/local/bin/` | Dockerイメージ | ビルド時に組み込む必要あり |
| Nodeランタイム | コンテナファイルシステム | Dockerイメージ | イメージビルドごとにリビルド |
| OSパッケージ | コンテナファイルシステム | Dockerイメージ | ランタイムでインストールしない |
| Dockerコンテナ | エフェメラル | 再起動可能 | 破棄しても安全 |

* * *

## インフラストラクチャーとしてのコード（Terraform）

インフラストラクチャーとしてのコードワークフローを好むチーム向けに、コミュニティ管理のTerraformセットアップが提供されています:

-   リモートステート管理を備えたモジュール式Terraform設定
-   cloud-initによる自動プロビジョニング
-   デプロイスクリプト（ブートストラップ、デプロイ、バックアップ/リストア）
-   セキュリティ強化（ファイアウォール、UFW、SSH専用アクセス）
-   GatewayアクセスのためのSSHトンネル設定

**リポジトリ:**

-   インフラストラクチャー: [openclaw-terraform-hetzner](https://github.com/andreesg/openclaw-terraform-hetzner)
-   Docker設定: [openclaw-docker-config](https://github.com/andreesg/openclaw-docker-config)

このアプローチは、上記のDockerセットアップを、再現可能なデプロイメント、バージョン管理されたインフラストラクチャー、自動化された障害復旧で補完します。

> **注意:** コミュニティ管理。問題や貢献については、上記のリポジトリリンクを参照してください。

[Fly.io](./fly.md)[GCP](./gcp.md)