

  ホスティングとデプロイ

  
# Fly.io

**目標:** [Fly.io](https://fly.io) マシン上で動作するOpenClaw Gateway。永続ストレージ、自動HTTPS、Discord/チャンネルアクセスを備えています。

## 必要なもの

-   [flyctl CLI](https://fly.io/docs/hands-on/install-flyctl/) がインストール済み
-   Fly.ioアカウント（無料枠で動作）
-   モデル認証: 選択したモデルプロバイダーのAPIキー
-   チャンネル認証情報: Discordボットトークン、Telegramトークンなど

## 初心者向けクイックパス

1.  リポジトリをクローン → `fly.toml` をカスタマイズ
2.  アプリとボリュームを作成 → シークレットを設定
3.  `fly deploy` でデプロイ
4.  SSHで接続して設定を作成、またはControl UIを使用

## 1) Flyアプリを作成

```bash
# リポジトリをクローン
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# 新しいFlyアプリを作成（名前は任意）
fly apps create my-openclaw

# 永続ボリュームを作成（通常1GBで十分）
fly volumes create openclaw_data --size 1 --region iad
```

**ヒント:** 近くのリージョンを選択してください。一般的なオプション: `lhr` (ロンドン), `iad` (バージニア), `sjc` (サンノゼ)。

## 2) fly.tomlを設定

`fly.toml` を編集して、アプリ名と要件に合わせます。**セキュリティ注意:** デフォルト設定は公開URLを公開します。パブリックIPなしの強化されたデプロイについては、[プライベートデプロイ](#private-deployment-hardened)を参照するか、`fly.private.toml` を使用してください。

```
app = "my-openclaw"  # あなたのアプリ名
primary_region = "iad"

[build]
  dockerfile = "Dockerfile"

[env]
  NODE_ENV = "production"
  OPENCLAW_PREFER_PNPM = "1"
  OPENCLAW_STATE_DIR = "/data"
  NODE_OPTIONS = "--max-old-space-size=1536"

[processes]
  app = "node dist/index.js gateway --allow-unconfigured --port 3000 --bind lan"

[http_service]
  internal_port = 3000
  force_https = true
  auto_stop_machines = false
  auto_start_machines = true
  min_machines_running = 1
  processes = ["app"]

[[vm]]
  size = "shared-cpu-2x"
  memory = "2048mb"

[mounts]
  source = "openclaw_data"
  destination = "/data"
```

**主要設定:**

| 設定 | 理由 |
| --- | --- |
| `--bind lan` | Flyのプロキシがゲートウェイに到達できるよう `0.0.0.0` にバインド |
| `--allow-unconfigured` | 設定ファイルなしで起動（後で作成します） |
| `internal_port = 3000` | Flyのヘルスチェックのために `--port 3000` (または `OPENCLAW_GATEWAY_PORT`) と一致させる必要があります |
| `memory = "2048mb"` | 512MBでは小さすぎます。2GBを推奨 |
| `OPENCLAW_STATE_DIR = "/data"` | ボリューム上で状態を永続化 |

## 3) シークレットを設定

```bash
# 必須: ゲートウェイトークン（非ループバックバインド用）
fly secrets set OPENCLAW_GATEWAY_TOKEN=$(openssl rand -hex 32)

# モデルプロバイダーAPIキー
fly secrets set ANTHROPIC_API_KEY=sk-ant-...

# オプション: 他のプロバイダー
fly secrets set OPENAI_API_KEY=sk-...
fly secrets set GOOGLE_API_KEY=...

# チャンネルトークン
fly secrets set DISCORD_BOT_TOKEN=MTQ...
```

**注意:**

-   非ループバックバインド (`--bind lan`) にはセキュリティのために `OPENCLAW_GATEWAY_TOKEN` が必要です。
-   これらのトークンはパスワードのように扱ってください。
-   すべてのAPIキーとトークンについて、**設定ファイルよりも環境変数を優先**してください。これにより、シークレットが誤って公開またはログに記録される可能性のある `openclaw.json` から除外されます。

## 4) デプロイ

```bash
fly deploy
```

初回デプロイではDockerイメージがビルドされます（約2-3分）。以降のデプロイはより高速です。デプロイ後、確認してください:

```bash
fly status
fly logs
```

以下のような表示がされるはずです:

```ini
[gateway] listening on ws://0.0.0.0:3000 (PID xxx)
[discord] logged in to discord as xxx
```

## 5) 設定ファイルを作成

適切な設定を作成するためにマシンにSSH接続します:

```bash
fly ssh console
```

設定ディレクトリとファイルを作成:

```bash
mkdir -p /data
cat > /data/openclaw.json << 'EOF'
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-6",
        "fallbacks": ["anthropic/claude-sonnet-4-5", "openai/gpt-4o"]
      },
      "maxConcurrent": 4
    },
    "list": [
      {
        "id": "main",
        "default": true
      }
    ]
  },
  "auth": {
    "profiles": {
      "anthropic:default": { "mode": "token", "provider": "anthropic" },
      "openai:default": { "mode": "token", "provider": "openai" }
    }
  },
  "bindings": [
    {
      "agentId": "main",
      "match": { "channel": "discord" }
    }
  ],
  "channels": {
    "discord": {
      "enabled": true,
      "groupPolicy": "allowlist",
      "guilds": {
        "YOUR_GUILD_ID": {
          "channels": { "general": { "allow": true } },
          "requireMention": false
        }
      }
    }
  },
  "gateway": {
    "mode": "local",
    "bind": "auto"
  },
  "meta": {
    "lastTouchedVersion": "2026.1.29"
  }
}
EOF
```

**注意:** `OPENCLAW_STATE_DIR=/data` の場合、設定パスは `/data/openclaw.json` です。 **注意:** Discordトークンは以下のいずれかから取得できます:

-   環境変数: `DISCORD_BOT_TOKEN` (シークレットにはこちらを推奨)
-   設定ファイル: `channels.discord.token`

環境変数を使用する場合、設定にトークンを追加する必要はありません。ゲートウェイは `DISCORD_BOT_TOKEN` を自動的に読み取ります。適用するために再起動:

```
exit
fly machine restart <machine-id>
```

## 6) ゲートウェイにアクセス

### Control UI

ブラウザで開く:

```bash
fly open
```

または `https://my-openclaw.fly.dev/` にアクセスしてください。ゲートウェイトークン (`OPENCLAW_GATEWAY_TOKEN` のもの) を貼り付けて認証します。

### ログ

```bash
fly logs              # ライブログ
fly logs --no-tail    # 最近のログ
```

### SSHコンソール

```bash
fly ssh console
```

## トラブルシューティング

### ”App is not listening on expected address”

ゲートウェイが `0.0.0.0` ではなく `127.0.0.1` にバインドしています。 **修正:** `fly.toml` のプロセスコマンドに `--bind lan` を追加してください。

### ヘルスチェック失敗 / 接続拒否

Flyが設定されたポートでゲートウェイに到達できません。 **修正:** `internal_port` がゲートウェイポート (`--port 3000` または `OPENCLAW_GATEWAY_PORT=3000` を設定) と一致していることを確認してください。

### OOM / メモリ問題

コンテナが再起動を繰り返す、または強制終了されます。兆候: `SIGABRT`, `v8::internal::Runtime_AllocateInYoungGeneration`, またはサイレント再起動。 **修正:** `fly.toml` でメモリを増やしてください:

```
[[vm]]
  memory = "2048mb"
```

または既存のマシンを更新:

```bash
fly machine update <machine-id> --vm-memory 2048 -y
```

**注意:** 512MBでは小さすぎます。1GBは動作する可能性がありますが、負荷がかかった場合や詳細なログ記録時にはOOMになる可能性があります。 **2GBを推奨します。**

### ゲートウェイロック問題

ゲートウェイが「already running」エラーで起動を拒否します。これは、コンテナが再起動してもボリューム上のPIDロックファイルが残存する場合に発生します。 **修正:** ロックファイルを削除:

```bash
fly ssh console --command "rm -f /data/gateway.*.lock"
fly machine restart <machine-id>
```

ロックファイルは `/data/gateway.*.lock` にあります（サブディレクトリ内ではありません）。

### 設定が読み込まれない

`--allow-unconfigured` を使用している場合、ゲートウェイは最小限の設定を作成します。再起動時に `/data/openclaw.json` にあるカスタム設定が読み込まれるはずです。設定が存在することを確認:

```bash
fly ssh console --command "cat /data/openclaw.json"
```

### SSH経由での設定書き込み

`fly ssh console -C` コマンドはシェルリダイレクトをサポートしていません。設定ファイルを書き込むには:

```bash
# echo + tee を使用 (ローカルからリモートへパイプ)
echo '{"your":"config"}' | fly ssh console -C "tee /data/openclaw.json"

# または sftp を使用
fly sftp shell
> put /local/path/config.json /data/openclaw.json
```

**注意:** ファイルが既に存在する場合、`fly sftp` は失敗する可能性があります。先に削除:

```bash
fly ssh console --command "rm /data/openclaw.json"
```

### 状態が永続化しない

再起動後に認証情報やセッションが失われる場合、状態ディレクトリがコンテナのファイルシステムに書き込まれています。 **修正:** `fly.toml` に `OPENCLAW_STATE_DIR=/data` が設定されていることを確認し、再デプロイしてください。

## 更新

```bash
# 最新の変更を取得
git pull

# 再デプロイ
fly deploy

# ヘルスを確認
fly status
fly logs
```

### マシンコマンドの更新

完全な再デプロイなしで起動コマンドを変更する必要がある場合:

```bash
# マシンIDを取得
fly machines list

# コマンドを更新
fly machine update <machine-id> --command "node dist/index.js gateway --port 3000 --bind lan" -y

# またはメモリ増加と共に
fly machine update <machine-id> --vm-memory 2048 --command "node dist/index.js gateway --port 3000 --bind lan" -y
```

**注意:** `fly deploy` の後、マシンコマンドは `fly.toml` の内容にリセットされる可能性があります。手動で変更を加えた場合は、デプロイ後に再適用してください。

## プライベートデプロイ (強化)

デフォルトでは、FlyはパブリックIPを割り当て、ゲートウェイを `https://your-app.fly.dev` でアクセス可能にします。これは便利ですが、デプロイがインターネットスキャナー (Shodan, Censysなど) によって発見可能であることを意味します。**公開されていない**強化されたデプロイの場合は、プライベートテンプレートを使用してください。

### プライベートデプロイを使用する場合

-   **外向き**の呼び出し/メッセージのみを行う場合 (内向きのウェブフックなし)
-   ウェブフックコールバックに **ngrok または Tailscale** トンネルを使用する場合
-   ブラウザではなく **SSH、プロキシ、または WireGuard** 経由でゲートウェイにアクセスする場合
-   デプロイを **インターネットスキャナーから隠したい** 場合

### セットアップ

標準設定の代わりに `fly.private.toml` を使用:

```bash
# プライベート設定でデプロイ
fly deploy -c fly.private.toml
```

または既存のデプロイを変換:

```bash
# 現在のIPをリスト
fly ips list -a my-openclaw

# パブリックIPを解放
fly ips release <public-ipv4> -a my-openclaw
fly ips release <public-ipv6> -a my-openclaw

# 将来のデプロイでパブリックIPが再割り当てされないよう、プライベート設定に切り替え
# ([http_service] を削除するか、プライベートテンプレートでデプロイ)
fly deploy -c fly.private.toml

# プライベート専用IPv6を割り当て
fly ips allocate-v6 --private -a my-openclaw
```

この後、`fly ips list` は `private` タイプのIPのみを表示するはずです:

```bash
VERSION  IP                   TYPE             REGION
v6       fdaa:x:x:x:x::x      private          global
```

### プライベートデプロイへのアクセス

パブリックURLがないため、以下のいずれかの方法を使用します: **オプション 1: ローカルプロキシ (最も簡単)**

```bash
# ローカルポート3000をアプリに転送
fly proxy 3000:3000 -a my-openclaw

# その後、ブラウザで http://localhost:3000 を開く
```

**オプション 2: WireGuard VPN**

```bash
# WireGuard設定を作成 (一度だけ)
fly wireguard create

# WireGuardクライアントにインポートし、内部IPv6経由でアクセス
# 例: http://[fdaa:x:x:x:x::x]:3000
```

**オプション 3: SSHのみ**

```bash
fly ssh console -a my-openclaw
```

### プライベートデプロイでのウェブフック

公開せずにウェブフックコールバック (Twilio, Telnyxなど) が必要な場合:

1.  **ngrokトンネル** - コンテナ内またはサイドカーとしてngrokを実行
2.  **Tailscale Funnel** - Tailscale経由で特定のパスを公開
3.  **外向きのみ** - 一部のプロバイダー (Twilio) はウェブフックなしで外向き呼び出しに問題なく動作します

ngrokを使用した音声通話設定の例:

```json
{
  "plugins": {
    "entries": {
      "voice-call": {
        "enabled": true,
        "config": {
          "provider": "twilio",
          "tunnel": { "provider": "ngrok" },
          "webhookSecurity": {
            "allowedHosts": ["example.ngrok.app"]
          }
        }
      }
    }
  }
}
```

ngrokトンネルはコンテナ内で実行され、Flyアプリ自体を公開せずにパブリックなウェブフックURLを提供します。転送されたホストヘッダーが受け入れられるよう、`webhookSecurity.allowedHosts` をパブリックトンネルのホスト名に設定してください。

### セキュリティ上の利点

| 側面 | パブリック | プライベート |
| --- | --- | --- |
| インターネットスキャナー | 発見可能 | 隠蔽 |
| 直接攻撃 | 可能性あり | ブロック |
| Control UIアクセス | ブラウザ | プロキシ/VPN |
| ウェブフック配信 | 直接 | トンネル経由 |

## 注意点

-   Fly.ioは **x86アーキテクチャ** を使用します (ARMではありません)
-   Dockerfileは両方のアーキテクチャと互換性があります
-   WhatsApp/Telegramのオンボーディングには `fly ssh console` を使用してください
-   永続データはボリューム上の `/data` に保存されます
-   SignalにはJava + signal-cliが必要です。カスタムイメージを使用し、メモリを2GB以上に保ってください。

## コスト

推奨設定 (`shared-cpu-2x`, 2GB RAM) の場合:

-   使用状況に応じて月額約$10-15
-   無料枠には一定の割り当てが含まれます

詳細は [Fly.io料金](https://fly.io/docs/about/pricing/) を参照してください。

[VPSホスティング](../vps.md)[Hetzner](./hetzner.md)