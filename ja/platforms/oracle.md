

  プラットフォーム概要

  
# Oracle Cloud

## 目標

Oracle Cloud の **Always Free** ARM 階層で永続的な OpenClaw Gateway を実行します。Oracle の無料枠は OpenClaw に非常に適している場合があります（特に OCI アカウントを既にお持ちの場合）が、トレードオフがあります:

-   ARM アーキテクチャ（ほとんどのものは動作しますが、一部のバイナリは x86 のみの場合があります）
-   キャパシティとサインアップが扱いにくい場合がある

## コスト比較 (2026)

| プロバイダー | プラン | スペック | 月額料金 | 備考 |
| --- | --- | --- | --- | --- |
| Oracle Cloud | Always Free ARM | 最大 4 OCPU, 24GB RAM | $0 | ARM、キャパシティ制限あり |
| Hetzner | CX22 | 2 vCPU, 4GB RAM | ~ $4 | 最安値の有料オプション |
| DigitalOcean | Basic | 1 vCPU, 1GB RAM | $6 | 使いやすい UI、優れたドキュメント |
| Vultr | Cloud Compute | 1 vCPU, 1GB RAM | $6 | 多くのロケーション |
| Linode | Nanode | 1 vCPU, 1GB RAM | $5 | 現在は Akamai の一部 |

* * *

## 前提条件

-   Oracle Cloud アカウント ([サインアップ](https://www.oracle.com/cloud/free/)) — 問題が発生した場合は [コミュニティサインアップガイド](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd) を参照
-   Tailscale アカウント ([tailscale.com](https://tailscale.com) で無料)
-   ~30 分

## 1) OCI インスタンスの作成

1.  [Oracle Cloud Console](https://cloud.oracle.com/) にログイン
2.  **コンピュート → インスタンス → インスタンスの作成** に移動
3.  設定:
    -   **名前:** `openclaw`
    -   **イメージ:** Ubuntu 24.04 (aarch64)
    -   **シェイプ:** `VM.Standard.A1.Flex` (Ampere ARM)
    -   **OCPU:** 2 (または最大 4)
    -   **メモリ:** 12 GB (または最大 24 GB)
    -   **ブートボリューム:** 50 GB (最大 200 GB 無料)
    -   **SSH キー:** 公開鍵を追加
4.  **作成** をクリック
5.  パブリック IP アドレスをメモ

**ヒント:** インスタンス作成が「キャパシティ不足」で失敗する場合は、別の可用性ドメインを試すか、後で再試行してください。無料枠のキャパシティは限られています。

## 2) 接続と更新

```bash
# パブリック IP 経由で接続
ssh ubuntu@YOUR_PUBLIC_IP

# システム更新
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential
```

**注:** `build-essential` は、一部の依存関係の ARM コンパイルに必要です。

## 3) ユーザーとホスト名の設定

```bash
# ホスト名を設定
sudo hostnamectl set-hostname openclaw

# ubuntu ユーザーのパスワードを設定
sudo passwd ubuntu

# 残留を有効化 (ログアウト後もユーザーサービスを実行し続ける)
sudo loginctl enable-linger ubuntu
```

## 4) Tailscale のインストール

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up --ssh --hostname=openclaw
```

これにより Tailscale SSH が有効になり、Tailnet 上の任意のデバイスから `ssh openclaw` で接続できるようになります — パブリック IP は不要です。確認:

```bash
tailscale status
```

**今後は Tailscale 経由で接続:** `ssh ubuntu@openclaw` (または Tailscale IP を使用)。

## 5) OpenClaw のインストール

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
source ~/.bashrc
```

「ボットをどのように孵化させますか？」とプロンプトが表示されたら、**「後で行う」** を選択してください。

> 注: ARM ネイティブビルドの問題が発生した場合は、Homebrew に頼る前に、まずシステムパッケージ (例: `sudo apt install -y build-essential`) から始めてください。

## 6) ゲートウェイの設定 (ループバック + トークン認証) と Tailscale Serve の有効化

デフォルトとしてトークン認証を使用します。これは予測可能で、「安全でない認証」コントロール UI フラグを必要としません。

```bash
# ゲートウェイを VM 上でプライベートに保つ
openclaw config set gateway.bind loopback

# ゲートウェイ + コントロール UI に認証を要求
openclaw config set gateway.auth.mode token
openclaw doctor --generate-gateway-token

# Tailscale Serve 経由で公開 (HTTPS + tailnet アクセス)
openclaw config set gateway.tailscale.mode serve
openclaw config set gateway.trustedProxies '["127.0.0.1"]'

systemctl --user restart openclaw-gateway
```

## 7) 検証

```bash
# バージョンを確認
openclaw --version

# デーモンのステータスを確認
systemctl --user status openclaw-gateway

# Tailscale Serve を確認
tailscale serve status

# ローカル応答をテスト
curl http://localhost:18789
```

## 8) VCN セキュリティのロックダウン

すべてが動作するようになったので、Tailscale を除くすべてのトラフィックをブロックするように VCN をロックダウンします。OCI の Virtual Cloud Network はネットワークエッジでファイアウォールとして機能します — トラフィックはインスタンスに到達する前にブロックされます。

1.  OCI コンソールで **ネットワーキング → 仮想クラウド・ネットワーク** に移動
2.  使用中の VCN をクリック → **セキュリティ・リスト** → デフォルト・セキュリティ・リスト
3.  以下のものを除くすべてのインバウンドルールを**削除**:
    -   `0.0.0.0/0 UDP 41641` (Tailscale)
4.  デフォルトのアウトバウンドルールはそのまま保持 (すべてのアウトバウンドを許可)

これにより、ポート 22 での SSH、HTTP、HTTPS、およびその他すべてがネットワークエッジでブロックされます。今後は Tailscale 経由でのみ接続できます。

* * *

## コントロール UI へのアクセス

Tailscale ネットワーク上の任意のデバイスから:

```
https://openclaw.<tailnet-name>.ts.net/
```

`<tailnet-name>` を Tailnet 名 (`tailscale status` で表示) に置き換えてください。SSH トンネルは不要です。Tailscale は以下を提供します:

-   HTTPS 暗号化 (自動証明書)
-   Tailscale ID による認証
-   Tailnet 上の任意のデバイス (ラップトップ、スマートフォンなど) からのアクセス

* * *

## セキュリティ: VCN + Tailscale (推奨ベースライン)

VCN をロックダウン (UDP 41641 のみ開放) し、ゲートウェイをループバックにバインドすることで、強力な多層防御が得られます: パブリックトラフィックはネットワークエッジでブロックされ、管理アクセスは Tailnet 経由で行われます。この設定により、インターネット全体からの SSH ブルートフォース攻撃を止めるためだけの追加のホストベースファイアウォールルールの*必要性*がしばしばなくなります — ただし、OS を最新の状態に保ち、`openclaw security audit` を実行し、誤ってパブリックインターフェースでリッスンしていないことを確認する必要があります。

### 既に保護されているもの

| 従来の手順 | 必要か？ | 理由 |
| --- | --- | --- |
| UFW ファイアウォール | いいえ | VCN がトラフィックがインスタンスに到達する前にブロック |
| fail2ban | いいえ | ポート 22 が VCN でブロックされていればブルートフォース攻撃なし |
| sshd の強化 | いいえ | Tailscale SSH は sshd を使用しない |
| root ログインの無効化 | いいえ | Tailscale はシステムユーザーではなく Tailscale ID を使用 |
| SSH キーのみの認証 | いいえ | Tailscale は Tailnet 経由で認証 |
| IPv6 の強化 | 通常は不要 | VCN/サブネットの設定に依存; 実際に割り当て/公開されているものを確認 |

### 依然として推奨されること

-   **資格情報の権限:** `chmod 700 ~/.openclaw`
-   **セキュリティ監査:** `openclaw security audit`
-   **システム更新:** `sudo apt update && sudo apt upgrade` を定期的に実行
-   **Tailscale の監視:** [Tailscale 管理コンソール](https://login.tailscale.com/admin) でデバイスを確認

### セキュリティ体制の確認

```bash
# パブリックポートがリッスンしていないことを確認
sudo ss -tlnp | grep -v '127.0.0.1\|::1'

# Tailscale SSH がアクティブであることを確認
tailscale status | grep -q 'offers: ssh' && echo "Tailscale SSH active"

# オプション: sshd を完全に無効化
sudo systemctl disable --now ssh
```

* * *

## フォールバック: SSH トンネル

Tailscale Serve が動作しない場合は、SSH トンネルを使用します:

```bash
# ローカルマシンから (Tailscale 経由)
ssh -L 18789:127.0.0.1:18789 ubuntu@openclaw
```

その後、`http://localhost:18789` を開きます。

* * *

## トラブルシューティング

### インスタンス作成が失敗する（「キャパシティ不足」）

無料枠 ARM インスタンスは人気があります。以下を試してください:

-   別の可用性ドメイン
-   オフピーク時間 (早朝) に再試行
-   シェイプ選択時に「Always Free」フィルターを使用

### Tailscale が接続できない

```bash
# ステータスを確認
sudo tailscale status

# 再認証
sudo tailscale up --ssh --hostname=openclaw --reset
```

### ゲートウェイが起動しない

```bash
openclaw gateway status
openclaw doctor --non-interactive
journalctl --user -u openclaw-gateway -n 50
```

### コントロール UI に到達できない

```bash
# Tailscale Serve が実行中であることを確認
tailscale serve status

# ゲートウェイがリッスンしていることを確認
curl http://localhost:18789

# 必要に応じて再起動
systemctl --user restart openclaw-gateway
```

### ARM バイナリの問題

一部のツールには ARM ビルドがない場合があります。確認:

```bash
uname -m  # aarch64 と表示されるはず
```

ほとんどの npm パッケージは問題なく動作します。バイナリの場合は、`linux-arm64` または `aarch64` リリースを探してください。

* * *

## 永続性

すべての状態は以下に保存されます:

-   `~/.openclaw/` — 設定、資格情報、セッションデータ
-   `~/.openclaw/workspace/` — ワークスペース (SOUL.md、メモリ、アーティファクト)

定期的にバックアップ:

```bash
tar -czvf openclaw-backup.tar.gz ~/.openclaw ~/.openclaw/workspace
```

* * *

## 関連項目

-   [ゲートウェイ リモートアクセス](../gateway/remote.md) — その他のリモートアクセスパターン
-   [Tailscale 統合](../gateway/tailscale.md) — 完全な Tailscale ドキュメント
-   [ゲートウェイ設定](../gateway/configuration.md) — すべての設定オプション
-   [DigitalOcean ガイド](./digitalocean.md) — 有料 + 簡単なサインアップが必要な場合
-   [Hetzner ガイド](../install/hetzner.md) — Docker ベースの代替案

[DigitalOcean](./digitalocean.md)[Raspberry Pi](./raspberry-pi.md)