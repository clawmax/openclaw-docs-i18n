

  プラットフォーム概要

  
# DigitalOcean

## 目標

DigitalOceanで永続的なOpenClaw Gatewayを **月額6ドル**（または予約価格で月額4ドル）で実行します。月額0ドルのオプションが必要で、ARM + プロバイダ固有のセットアップを気にしない場合は、[Oracle Cloudガイド](./oracle.md)を参照してください。

## コスト比較 (2026)

| プロバイダー | プラン | 仕様 | 月額料金 | 備考 |
| --- | --- | --- | --- | --- |
| Oracle Cloud | Always Free ARM | 最大4 OCPU, 24GB RAM | $0 | ARM、容量制限/サインアップの癖あり |
| Hetzner | CX22 | 2 vCPU, 4GB RAM | €3.79 (~$4) | 最安値の有料オプション |
| DigitalOcean | Basic | 1 vCPU, 1GB RAM | $6 | シンプルなUI、良いドキュメント |
| Vultr | Cloud Compute | 1 vCPU, 1GB RAM | $6 | 多くのロケーション |
| Linode | Nanode | 1 vCPU, 1GB RAM | $5 | 現在Akamaiの一部 |

**プロバイダーの選択:**

-   DigitalOcean: 最もシンプルなUX + 予測可能なセットアップ (このガイド)
-   Hetzner: 良い価格/性能比 ( [Hetznerガイド](../install/hetzner.md)を参照)
-   Oracle Cloud: 月額0ドルの可能性があるが、より扱いが難しくARMのみ ( [Oracleガイド](./oracle.md)を参照)

* * *

## 前提条件

-   DigitalOceanアカウント ([200ドルの無料クレジットでサインアップ](https://m.do.co/c/signup))
-   SSHキーペア (またはパスワード認証の使用意思)
-   ~20分

## 1) Dropletを作成する

> **⚠️** クリーンなベースイメージ (Ubuntu 24.04 LTS) を使用してください。スタートアップスクリプトとファイアウォールのデフォルトを確認していない限り、サードパーティのマーケットプレイス1クリックイメージは避けてください。

1.  [DigitalOcean](https://cloud.digitalocean.com/)にログイン
2.  **Create → Droplets** をクリック
3.  選択:
    -   **Region:** あなた (またはユーザー) に最も近い地域
    -   **Image:** Ubuntu 24.04 LTS
    -   **Size:** Basic → Regular → **$6/mo** (1 vCPU, 1GB RAM, 25GB SSD)
    -   **Authentication:** SSHキー (推奨) またはパスワード
4.  **Create Droplet** をクリック
5.  IPアドレスをメモ

## 2) SSHで接続する

```bash
ssh root@YOUR_DROPLET_IP
```

## 3) OpenClawをインストールする

```bash
# Update system
apt update && apt upgrade -y

# Install Node.js 22
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs

# Install OpenClaw
curl -fsSL https://openclaw.ai/install.sh | bash

# Verify
openclaw --version
```

## 4) オンボーディングを実行する

```bash
openclaw onboard --install-daemon
```

ウィザードが以下を案内します:

-   モデル認証 (APIキーまたはOAuth)
-   チャネル設定 (Telegram, WhatsApp, Discordなど)
-   ゲートウェイトークン (自動生成)
-   デーモンインストール (systemd)

## 5) ゲートウェイを確認する

```bash
# Check status
openclaw status

# Check service
systemctl --user status openclaw-gateway.service

# View logs
journalctl --user -u openclaw-gateway.service -f
```

## 6) ダッシュボードにアクセスする

ゲートウェイはデフォルトでループバックにバインドされます。コントロールUIにアクセスするには: **オプション A: SSHトンネル (推奨)**

```bash
# From your local machine
ssh -L 18789:localhost:18789 root@YOUR_DROPLET_IP

# Then open: http://localhost:18789
```

**オプション B: Tailscale Serve (HTTPS, ループバック限定)**

```bash
# On the droplet
curl -fsSL https://tailscale.com/install.sh | sh
tailscale up

# Configure Gateway to use Tailscale Serve
openclaw config set gateway.tailscale.mode serve
openclaw gateway restart
```

開く: `https:///` 注記:

-   Serveはゲートウェイをループバック限定に保ち、Tailscale IDヘッダーを介してコントロールUI/WebSocketトラフィックを認証します (トークンレス認証は信頼できるゲートウェイホストを前提とします; HTTP APIは依然としてトークン/パスワードを必要とします)。
-   代わりにトークン/パスワードを要求するには、`gateway.auth.allowTailscale: false`を設定するか、`gateway.auth.mode: "password"`を使用してください。

**オプション C: Tailnetバインド (Serveなし)**

```bash
openclaw config set gateway.bind tailnet
openclaw gateway restart
```

開く: `http://<tailscale-ip>:18789` (トークンが必要)。

## 7) チャネルを接続する

### Telegram

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

### WhatsApp

```bash
openclaw channels login whatsapp
# Scan QR code
```

他のプロバイダーについては[チャネル](../channels.md)を参照してください。

* * *

## 1GB RAMのための最適化

6ドルのDropletは1GB RAMしかありません。スムーズに実行し続けるには:

### スワップを追加する (推奨)

```bash
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

### より軽量なモデルを使用する

OOMが発生する場合は、以下を検討してください:

-   ローカルモデルの代わりにAPIベースのモデル (Claude, GPT) を使用する
-   `agents.defaults.model.primary`をより小さなモデルに設定する

### メモリを監視する

```
free -h
htop
```

* * *

## 永続性

すべての状態は以下に保存されます:

-   `~/.openclaw/` — 設定、認証情報、セッションデータ
-   `~/.openclaw/workspace/` — ワークスペース (SOUL.md、メモリなど)

これらは再起動後も保持されます。定期的にバックアップしてください:

```bash
tar -czvf openclaw-backup.tar.gz ~/.openclaw ~/.openclaw/workspace
```

* * *

## Oracle Cloud無料代替案

Oracle Cloudは、ここにあるどの有料オプションよりも大幅に強力な **Always Free** ARMインスタンスを提供します — 月額0ドルで。

| 得られるもの | 仕様 |
| --- | --- |
| **4 OCPU** | ARM Ampere A1 |
| **24GB RAM** | 十分以上 |
| **200GB ストレージ** | ブロックボリューム |
| **永久無料** | クレジットカード請求なし |

**注意点:**

-   サインアップが扱いにくい場合がある (失敗した場合は再試行)
-   ARMアーキテクチャ — ほとんどのものは動作するが、一部のバイナリはARMビルドが必要

完全なセットアップガイドについては、[Oracle Cloud](./oracle.md)を参照してください。サインアップのヒントと登録プロセスのトラブルシューティングについては、この[コミュニティガイド](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd)を参照してください。

* * *

## トラブルシューティング

### ゲートウェイが起動しない

```bash
openclaw gateway status
openclaw doctor --non-interactive
journalctl -u openclaw --no-pager -n 50
```

### ポートが既に使用中

```bash
lsof -i :18789
kill <PID>
```

### メモリ不足

```bash
# Check memory
free -h

# Add more swap
# Or upgrade to $12/mo droplet (2GB RAM)
```

* * *

## 関連項目

-   [Hetznerガイド](../install/hetzner.md) — より安価で強力
-   [Dockerインストール](../install/docker.md) — コンテナ化セットアップ
-   [Tailscale](../gateway/tailscale.md) — 安全なリモートアクセス
-   [設定](../gateway/configuration.md) — 完全な設定リファレンス

[iOSアプリ](./ios.md)[Oracle Cloud](./oracle.md)

---