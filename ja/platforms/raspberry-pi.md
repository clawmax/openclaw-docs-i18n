

  プラットフォーム概要

  
# Raspberry Pi

## 目標

**約35〜80ドル**の初期費用（月額費用なし）で、Raspberry Pi上に永続的で常時稼働するOpenClaw Gatewayを実行します。以下に最適です：

-   24時間365日の個人用AIアシスタント
-   ホームオートメーション ハブ
-   低電力、常時利用可能なTelegram/WhatsAppボット

## ハードウェア要件

| Pi モデル | RAM | 動作可否 | 備考 |
| --- | --- | --- | --- |
| **Pi 5** | 4GB/8GB | ✅ 最適 | 最速、推奨 |
| **Pi 4** | 4GB | ✅ 良好 | ほとんどのユーザーに最適 |
| **Pi 4** | 2GB | ✅ 可 | 動作しますが、スワップを追加 |
| **Pi 4** | 1GB | ⚠️ 厳しい | スワップで可能、最小構成 |
| **Pi 3B+** | 1GB | ⚠️ 低速 | 動作しますが重い |
| **Pi Zero 2 W** | 512MB | ❌ | 非推奨 |

**最小スペック:** 1GB RAM、1コア、500MB ディスク  
**推奨スペック:** 2GB以上 RAM、64-bit OS、16GB以上 SDカード（またはUSB SSD）

## 必要なもの

-   Raspberry Pi 4 または 5 (2GB以上推奨)
-   MicroSDカード (16GB以上) または USB SSD (パフォーマンス向上)
-   電源アダプター (公式Pi PSU推奨)
-   ネットワーク接続 (イーサネットまたはWiFi)
-   ~30分

## 1) OSを書き込む

ヘッドレスサーバーにはデスクトップが不要なので、**Raspberry Pi OS Lite (64-bit)** を使用します。

1.  [Raspberry Pi Imager](https://www.raspberrypi.com/software/) をダウンロード
2.  OSを選択: **Raspberry Pi OS Lite (64-bit)**
3.  歯車アイコン (⚙️) をクリックして事前設定:
    -   ホスト名を設定: `gateway-host`
    -   SSHを有効化
    -   ユーザー名/パスワードを設定
    -   WiFiを設定 (イーサネットを使用しない場合)
4.  SDカード / USBドライブに書き込む
5.  Piに挿入して起動

## 2) SSHで接続

```bash
ssh user@gateway-host
# またはIPアドレスを使用
ssh user@192.168.x.x
```

## 3) システムセットアップ

```bash
# システムを更新
sudo apt update && sudo apt upgrade -y

# 必須パッケージをインストール
sudo apt install -y git curl build-essential

# タイムゾーンを設定 (cron/リマインダーに重要)
sudo timedatectl set-timezone America/Chicago  # お住まいのタイムゾーンに変更
```

## 4) Node.js 22 (ARM64) をインストール

```bash
# NodeSource経由でNode.jsをインストール
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs

# 確認
node --version  # v22.x.x が表示されるはず
npm --version
```

## 5) スワップを追加 (2GB以下の場合重要)

スワップはメモリ不足によるクラッシュを防ぎます：

```bash
# 2GBのスワップファイルを作成
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# 永続化
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# 低RAM用に最適化 (スワップ頻度を減らす)
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## 6) OpenClawをインストール

### オプションA: 標準インストール (推奨)

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

### オプションB: ハッカブルインストール (カスタマイズ用)

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
npm install
npm run build
npm link
```

ハッカブルインストールでは、ログとコードに直接アクセスできます — ARM固有の問題のデバッグに便利です。

## 7) オンボーディングを実行

```bash
openclaw onboard --install-daemon
```

ウィザードに従います：

1.  **ゲートウェイモード:** ローカル
2.  **認証:** APIキー推奨 (ヘッドレスPiではOAuthが不安定な場合あり)
3.  **チャネル:** 最初はTelegramが最も簡単
4.  **デーモン:** はい (systemd)

## 8) インストールを確認

```bash
# ステータスを確認
openclaw status

# サービスを確認
sudo systemctl status openclaw

# ログを表示
journalctl -u openclaw -f
```

## 9) ダッシュボードにアクセス

Piはヘッドレスなので、SSHトンネルを使用します：

```bash
# ラップトップ/デスクトップから
ssh -L 18789:localhost:18789 user@gateway-host

# その後ブラウザで開く
open http://localhost:18789
```

または、常時アクセスのためにTailscaleを使用します：

```bash
# Pi上で
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up

# 設定を更新
openclaw config set gateway.bind tailnet
sudo systemctl restart openclaw
```

* * *

## パフォーマンス最適化

### USB SSDを使用する (大幅な改善)

SDカードは遅く、消耗します。USB SSDはパフォーマンスを劇的に向上させます：

```bash
# USBから起動しているか確認
lsblk
```

セットアップについては [Pi USBブートガイド](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#usb-mass-storage-boot) を参照してください。

### CLI起動を高速化 (モジュールコンパイルキャッシュ)

低電力Piホストでは、Nodeのモジュールコンパイルキャッシュを有効にして、CLIの繰り返し実行を高速化します：

```
grep -q 'NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache' ~/.bashrc || cat >> ~/.bashrc <<'EOF' # pragma: allowlist secret
export NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache
mkdir -p /var/tmp/openclaw-compile-cache
export OPENCLAW_NO_RESPAWN=1
EOF
source ~/.bashrc
```

注意点：

-   `NODE_COMPILE_CACHE` は後続の実行 (`status`, `health`, `--help`) を高速化します。
-   `/var/tmp` は `/tmp` よりも再起動後の生存性が高いです。
-   `OPENCLAW_NO_RESPAWN=1` はCLIの自己再起動による追加の起動コストを回避します。
-   最初の実行でキャッシュがウォームアップされ、後の実行で最も効果を発揮します。

### systemd起動チューニング (オプション)

このPiが主にOpenClawを実行する場合は、サービスドロップインを追加して再起動時のジッターを減らし、起動環境を安定させます：

```bash
sudo systemctl edit openclaw
```

```ini
[Service]
Environment=OPENCLAW_NO_RESPAWN=1
Environment=NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache
Restart=always
RestartSec=2
TimeoutStartSec=90
```

その後、適用します：

```bash
sudo systemctl daemon-reload
sudo systemctl restart openclaw
```

可能であれば、OpenClawの状態/キャッシュをSSDバックアップストレージに保持して、コールドスタート時のSDカードのランダムI/Oボトルネックを回避します。`Restart=` ポリシーが自動復旧にどのように役立つか: [systemd can automate service recovery](https://www.redhat.com/en/blog/systemd-automate-recovery)。

### メモリ使用量を削減

```bash
# GPUメモリ割り当てを無効化 (ヘッドレス)
echo 'gpu_mem=16' | sudo tee -a /boot/config.txt

# 不要な場合はBluetoothを無効化
sudo systemctl disable bluetooth
```

### リソースを監視

```bash
# メモリを確認
free -h

# CPU温度を確認
vcgencmd measure_temp

# ライブ監視
htop
```

* * *

## ARM固有の注意点

### バイナリ互換性

ほとんどのOpenClaw機能はARM64で動作しますが、一部の外部バイナリはARMビルドが必要な場合があります：

| ツール | ARM64 ステータス | 備考 |
| --- | --- | --- |
| Node.js | ✅ | 問題なく動作 |
| WhatsApp (Baileys) | ✅ | 純粋なJS、問題なし |
| Telegram | ✅ | 純粋なJS、問題なし |
| gog (Gmail CLI) | ⚠️ | ARMリリースを確認 |
| Chromium (ブラウザ) | ✅ | `sudo apt install chromium-browser` |

スキルが失敗する場合は、そのバイナリにARM64ビルドがあるか確認してください。多くのGo/Rustツールはありますが、一部はありません。

### 32-bit 対 64-bit

**常に64-bit OSを使用してください。** Node.jsや多くの最新ツールはそれを必要とします。以下で確認します：

```bash
uname -m
# 表示されるはず: aarch64 (64-bit) ではなく armv7l (32-bit)
```

* * *

## 推奨モデル設定

Piは単なるゲートウェイ（モデルはクラウドで実行）なので、APIベースのモデルを使用します：

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-sonnet-4-20250514",
        "fallbacks": ["openai/gpt-4o-mini"]
      }
    }
  }
}
```

**PiでローカルLLMを実行しようとしないでください** — 小さなモデルでも遅すぎます。重い処理はClaude/GPTに任せましょう。

* * *

## 起動時の自動起動

オンボーディングウィザードで設定されますが、確認するには：

```bash
# サービスが有効化されているか確認
sudo systemctl is-enabled openclaw

# 有効化されていない場合は有効化
sudo systemctl enable openclaw

# 起動時に開始
sudo systemctl start openclaw
```

* * *

## トラブルシューティング

### メモリ不足 (OOM)

```bash
# メモリを確認
free -h

# スワップを追加 (ステップ5を参照)
# またはPiで実行中のサービスを減らす
```

### パフォーマンスが遅い

-   SDカードの代わりにUSB SSDを使用
-   未使用のサービスを無効化: `sudo systemctl disable cups bluetooth avahi-daemon`
-   CPUスロットリングを確認: `vcgencmd get_throttled` (`0x0` が返るはず)

### サービスが起動しない

```bash
# ログを確認
journalctl -u openclaw --no-pager -n 100

# 一般的な修正: リビルド
cd ~/openclaw  # ハッカブルインストールを使用している場合
npm run build
sudo systemctl restart openclaw
```

### ARMバイナリの問題

スキルが "exec format error" で失敗する場合：

1.  バイナリにARM64ビルドがあるか確認
2.  ソースからビルドを試みる
3.  またはARMサポート付きのDockerコンテナを使用する

### WiFiが切断する

WiFi上のヘッドレスPiの場合：

```bash
# WiFi省電力管理を無効化
sudo iwconfig wlan0 power off

# 永続化
echo 'wireless-power off' | sudo tee -a /etc/network/interfaces
```

* * *

## コスト比較

| セットアップ | 初期費用 | 月額費用 | 備考 |
| --- | --- | --- | --- |
| **Pi 4 (2GB)** | ~$45 | $0 | \+ 電力 (~$5/年) |
| **Pi 4 (4GB)** | ~$55 | $0 | 推奨 |
| **Pi 5 (4GB)** | ~$60 | $0 | 最高のパフォーマンス |
| **Pi 5 (8GB)** | ~$80 | $0 | 過剰ですが将来性あり |
| DigitalOcean | $0 | $6/月 | $72/年 |
| Hetzner | $0 | €3.79/月 | ~$50/年 |

**損益分岐点:** PiはクラウドVPSと比較して約6〜12ヶ月で元が取れます。

* * *

## 関連項目

-   [Linuxガイド](./linux.md) — 一般的なLinuxセットアップ
-   [DigitalOceanガイド](./digitalocean.md) — クラウド代替案
-   [Hetznerガイド](../install/hetzner.md) — Dockerセットアップ
-   [Tailscale](../gateway/tailscale.md) — リモートアクセス
-   [ノード](../nodes.md) — ラップトップ/電話をPiゲートウェイとペアリング

[Oracle Cloud](./oracle.md)[macOS 開発セットアップ](./mac/dev-setup.md)