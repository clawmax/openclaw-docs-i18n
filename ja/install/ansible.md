

  その他のインストール方法

  
# Ansible

本番サーバーにOpenClawをデプロイする推奨方法は、**[openclaw-ansible](https://github.com/openclaw/openclaw-ansible)** による自動インストーラーです。これはセキュリティファーストのアーキテクチャを備えています。

## クイックスタート

ワンコマンドインストール:

```bash
curl -fsSL https://raw.githubusercontent.com/openclaw/openclaw-ansible/main/install.sh | bash
```

> **📦 完全ガイド: [github.com/openclaw/openclaw-ansible](https://github.com/openclaw/openclaw-ansible)** openclaw-ansibleリポジトリがAnsibleデプロイメントの信頼できる情報源です。このページは概要です。

## 得られるもの

-   🔒 **ファイアウォールファーストのセキュリティ**: UFW + Docker分離 (SSH + Tailscaleのみアクセス可能)
-   🔐 **Tailscale VPN**: サービスを公開せずに安全なリモートアクセスを実現
-   🐳 **Docker**: 分離されたサンドボックスコンテナ、localhostのみバインド
-   🛡️ **多層防御**: 4層のセキュリティアーキテクチャ
-   🚀 **ワンコマンドセットアップ**: 数分で完全なデプロイメント
-   🔧 **Systemd統合**: セキュリティ強化を施した自動起動

## 要件

-   **OS**: Debian 11+ または Ubuntu 20.04+
-   **アクセス**: rootまたはsudo権限
-   **ネットワーク**: パッケージインストールのためのインターネット接続
-   **Ansible**: 2.14+ (クイックスタートスクリプトにより自動インストール)

## インストールされるもの

Ansibleプレイブックは以下をインストール・設定します:

1.  **Tailscale** (安全なリモートアクセスのためのメッシュVPN)
2.  **UFWファイアウォール** (SSH + Tailscaleポートのみ許可)
3.  **Docker CE + Compose V2** (エージェントサンドボックス用)
4.  **Node.js 22.x + pnpm** (ランタイム依存関係)
5.  **OpenClaw** (ホストベース、コンテナ化されていない)
6.  **Systemdサービス** (セキュリティ強化を施した自動起動)

注: ゲートウェイは **ホスト上で直接実行** されます (Docker内ではありません) が、エージェントサンドボックスは分離のためにDockerを使用します。詳細は [サンドボックス化](../gateway/sandboxing.md) を参照してください。

## インストール後のセットアップ

インストール完了後、openclawユーザーに切り替えます:

```bash
sudo -i -u openclaw
```

インストール後スクリプトが以下を案内します:

1.  **オンボーディングウィザード**: OpenClaw設定の構成
2.  **プロバイダーログイン**: WhatsApp/Telegram/Discord/Signalへの接続
3.  **ゲートウェイテスト**: インストールの検証
4.  **Tailscaleセットアップ**: VPNメッシュへの接続

### クイックコマンド

```bash
# サービスステータスの確認
sudo systemctl status openclaw

# ライブログの表示
sudo journalctl -u openclaw -f

# ゲートウェイの再起動
sudo systemctl restart openclaw

# プロバイダーログイン (openclawユーザーとして実行)
sudo -i -u openclaw
openclaw channels login
```

## セキュリティアーキテクチャ

### 4層防御

1.  **ファイアウォール (UFW)**: SSH (22) + Tailscale (41641/udp) のみ公開
2.  **VPN (Tailscale)**: ゲートウェイはVPNメッシュ経由でのみアクセス可能
3.  **Docker分離**: DOCKER-USER iptablesチェーンにより外部ポート公開を防止
4.  **Systemd強化**: NoNewPrivileges, PrivateTmp, 非特権ユーザー

### 検証

外部からの攻撃対象領域をテスト:

```bash
nmap -p- YOUR_SERVER_IP
```

**ポート22 (SSH) のみ** が開いていることを確認してください。他のすべてのサービス (ゲートウェイ、Docker) はロックダウンされています。

### Dockerの可用性

Dockerは **エージェントサンドボックス** (分離されたツール実行) のためにインストールされ、ゲートウェイ自体の実行には使用されません。ゲートウェイはlocalhostのみにバインドされ、Tailscale VPN経由でアクセス可能です。サンドボックス設定については [マルチエージェントサンドボックス & ツール](../tools/multi-agent-sandbox-tools.md) を参照してください。

## 手動インストール

自動化よりも手動制御を希望する場合:

```bash
# 1. 前提条件のインストール
sudo apt update && sudo apt install -y ansible git

# 2. リポジトリのクローン
git clone https://github.com/openclaw/openclaw-ansible.git
cd openclaw-ansible

# 3. Ansibleコレクションのインストール
ansible-galaxy collection install -r requirements.yml

# 4. プレイブックの実行
./run-playbook.sh

# または直接実行 (その後、手動で /tmp/openclaw-setup.sh を実行)
# ansible-playbook playbook.yml --ask-become-pass
```

## OpenClawの更新

Ansibleインストーラーは、手動更新用にOpenClawをセットアップします。標準的な更新フローについては [更新](./updating.md) を参照してください。Ansibleプレイブックを再実行する場合 (例: 設定変更のため):

```bash
cd openclaw-ansible
./run-playbook.sh
```

注: これは冪等性があり、複数回実行しても安全です。

## トラブルシューティング

### ファイアウォールが接続をブロックする

アクセスできない場合:

-   まずTailscale VPN経由でアクセスできることを確認してください
-   SSHアクセス (ポート22) は常に許可されています
-   ゲートウェイは設計上、**Tailscale経由でのみ** アクセス可能です

### サービスが起動しない

```bash
# ログの確認
sudo journalctl -u openclaw -n 100

# 権限の確認
sudo ls -la /opt/openclaw

# 手動起動のテスト
sudo -i -u openclaw
cd ~/openclaw
pnpm start
```

### Dockerサンドボックスの問題

```bash
# Dockerが実行中か確認
sudo systemctl status docker

# サンドボックスイメージの確認
sudo docker images | grep openclaw-sandbox

# サンドボックスイメージが欠落している場合のビルド
cd /opt/openclaw/openclaw
sudo -u openclaw ./scripts/sandbox-setup.sh
```

### プロバイダーログインが失敗する

`openclaw` ユーザーとして実行していることを確認してください:

```bash
sudo -i -u openclaw
openclaw channels login
```

## 高度な設定

詳細なセキュリティアーキテクチャとトラブルシューティングについては:

-   [セキュリティアーキテクチャ](https://github.com/openclaw/openclaw-ansible/blob/main/docs/security.md)
-   [技術詳細](https://github.com/openclaw/openclaw-ansible/blob/main/docs/architecture.md)
-   [トラブルシューティングガイド](https://github.com/openclaw/openclaw-ansible/blob/main/docs/troubleshooting.md)

## 関連項目

-   [openclaw-ansible](https://github.com/openclaw/openclaw-ansible) — 完全なデプロイメントガイド
-   [Docker](./docker.md) — コンテナ化されたゲートウェイセットアップ
-   [サンドボックス化](../gateway/sandboxing.md) — エージェントサンドボックス設定
-   [マルチエージェントサンドボックス & ツール](../tools/multi-agent-sandbox-tools.md) — エージェントごとの分離

[Nix](./nix.md)[Bun (実験的)](./bun.md)