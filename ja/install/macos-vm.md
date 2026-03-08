

  ホスティングとデプロイメント

  
# macOS VM

## 推奨デフォルト（ほとんどのユーザー）

-   **小型Linux VPS** を常時稼働のゲートウェイと低コストのために使用します。 [VPSホスティング](../vps.md)を参照してください。
-   完全な制御とブラウザ自動化のための**住宅IP**が必要な場合は、**専用ハードウェア**（Mac miniまたはLinuxボックス）を使用します。多くのサイトはデータセンターIPをブロックするため、ローカルブラウジングの方がうまくいくことがよくあります。
-   **ハイブリッド:** ゲートウェイは安価なVPS上で維持し、ブラウザ/UI自動化が必要な時にMacを**ノード**として接続します。 [ノード](../nodes.md)と[ゲートウェイリモート](../gateway/remote.md)を参照してください。

macOS限定の機能（iMessage/BlueBubbles）を特に必要とする場合、または日常使用のMacから厳密に分離したい場合に、macOS VMを使用してください。

## macOS VMのオプション

### Apple Silicon Mac上のローカルVM（Lume）

既存のApple Silicon Mac上で[Lume](https://cua.ai/docs/lume)を使用して、サンドボックス化されたmacOS VM内でOpenClawを実行します。これにより以下が得られます：

-   分離された完全なmacOS環境（ホストはクリーンなまま）
-   BlueBubbles経由のiMessageサポート（Linux/Windowsでは不可能）
-   VMのクローンによる瞬時のリセット
-   追加のハードウェアやクラウドコストなし

### ホステッドMacプロバイダー（クラウド）

クラウドでmacOSが必要な場合は、ホステッドMacプロバイダーも機能します：

-   [MacStadium](https://www.macstadium.com/)（ホステッドMac）
-   他のホステッドMacベンダーも機能します。VM + SSHのドキュメントに従ってください

macOS VMへのSSHアクセスができたら、以下のステップ6から続行してください。

* * *

## クイックパス（Lume、経験豊富なユーザー向け）

1.  Lumeをインストール
2.  `lume create openclaw --os macos --ipsw latest`
3.  セットアップアシスタントを完了し、リモートログイン（SSH）を有効化
4.  `lume run openclaw --no-display`
5.  SSHで接続し、OpenClawをインストール、チャネルを設定
6.  完了

* * *

## 必要なもの（Lume）

-   Apple Silicon Mac（M1/M2/M3/M4）
-   ホスト上のmacOS Sequoia以降
-   VMあたり約60 GBの空きディスク容量
-   約20分

* * *

## 1) Lumeをインストール

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/trycua/cua/main/libs/lume/scripts/install.sh)"
```

`~/.local/bin`がPATHにない場合：

```bash
echo 'export PATH="$PATH:$HOME/.local/bin"' >> ~/.zshrc && source ~/.zshrc
```

確認：

```bash
lume --version
```

ドキュメント：[Lumeインストール](https://cua.ai/docs/lume/guide/getting-started/installation)

* * *

## 2) macOS VMを作成

```bash
lume create openclaw --os macos --ipsw latest
```

これによりmacOSがダウンロードされ、VMが作成されます。VNCウィンドウが自動的に開きます。注：ダウンロードには接続状況によって時間がかかることがあります。

* * *

## 3) セットアップアシスタントを完了

VNCウィンドウ内で：

1.  言語と地域を選択
2.  Apple IDをスキップ（または後でiMessageを使用したい場合はサインイン）
3.  ユーザーアカウントを作成（ユーザー名とパスワードを覚えておく）
4.  すべてのオプション機能をスキップ

セットアップ完了後、SSHを有効化：

1.  システム設定 → 一般 → 共有を開く
2.  「リモートログイン」を有効化

* * *

## 4) VMのIPアドレスを取得

```bash
lume get openclaw
```

IPアドレス（通常は`192.168.64.x`）を探します。

* * *

## 5) VMにSSH接続

```bash
ssh youruser@192.168.64.X
```

`youruser`を作成したアカウントに、IPをVMのIPに置き換えてください。

* * *

## 6) OpenClawをインストール

VM内で：

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

オンボーディングのプロンプトに従って、モデルプロバイダー（Anthropic、OpenAIなど）を設定してください。

* * *

## 7) チャネルを設定

設定ファイルを編集：

```bash
nano ~/.openclaw/openclaw.json
```

チャネルを追加：

```json
{
  "channels": {
    "whatsapp": {
      "dmPolicy": "allowlist",
      "allowFrom": ["+15551234567"]
    },
    "telegram": {
      "botToken": "YOUR_BOT_TOKEN"
    }
  }
}
```

次にWhatsAppにログイン（QRコードをスキャン）：

```bash
openclaw channels login
```

* * *

## 8) VMをヘッドレスで実行

VMを停止し、ディスプレイなしで再起動：

```bash
lume stop openclaw
lume run openclaw --no-display
```

VMはバックグラウンドで実行されます。OpenClawのデーモンがゲートウェイを稼働させ続けます。ステータスを確認：

```bash
ssh youruser@192.168.64.X "openclaw status"
```

* * *

## ボーナス：iMessage統合

これがmacOS上で実行する最大の利点です。[BlueBubbles](https://bluebubbles.app)を使用してOpenClawにiMessageを追加します。VM内で：

1.  bluebubbles.appからBlueBubblesをダウンロード
2.  Apple IDでサインイン
3.  Web APIを有効化し、パスワードを設定
4.  BlueBubblesのウェブフックをゲートウェイに向ける（例：`https://your-gateway-host:3000/bluebubbles-webhook?password=`）

OpenClaw設定に追加：

```json
{
  "channels": {
    "bluebubbles": {
      "serverUrl": "http://localhost:1234",
      "password": "your-api-password",
      "webhookPath": "/bluebubbles-webhook"
    }
  }
}
```

ゲートウェイを再起動。これでエージェントがiMessageを送受信できるようになります。完全なセットアップ詳細：[BlueBubblesチャネル](../channels/bluebubbles.md)

* * *

## ゴールデンイメージを保存

さらにカスタマイズする前に、クリーンな状態をスナップショット：

```bash
lume stop openclaw
lume clone openclaw openclaw-golden
```

いつでもリセット：

```bash
lume stop openclaw && lume delete openclaw
lume clone openclaw-golden openclaw
lume run openclaw --no-display
```

* * *

## 24時間稼働

VMを稼働させ続けるには：

-   Macを電源に接続したままにする
-   システム設定 → 省エネルギーでスリープを無効化
-   必要に応じて`caffeinate`を使用

真の常時稼働には、専用のMac miniまたは小型VPSを検討してください。[VPSホスティング](../vps.md)を参照。

* * *

## トラブルシューティング

| 問題 | 解決策 |
| --- | --- |
| VMにSSH接続できない | VMのシステム設定で「リモートログイン」が有効か確認 |
| VMのIPが表示されない | VMが完全に起動するのを待ち、`lume get openclaw`を再度実行 |
| Lumeコマンドが見つからない | `~/.local/bin`をPATHに追加 |
| WhatsApp QRコードがスキャンできない | `openclaw channels login`を実行する際、ホストではなくVMにログインしていることを確認 |

* * *

## 関連ドキュメント

-   [VPSホスティング](../vps.md)
-   [ノード](../nodes.md)
-   [ゲートウェイリモート](../gateway/remote.md)
-   [BlueBubblesチャネル](../channels/bluebubbles.md)
-   [Lumeクイックスタート](https://cua.ai/docs/lume/guide/getting-started/quickstart)
-   [Lume CLIリファレンス](https://cua.ai/docs/lume/reference/cli-reference)
-   [無人VMセットアップ](https://cua.ai/docs/lume/guide/fundamentals/unattended-setup)（上級者向け）
-   [Dockerサンドボックス化](./docker.md)（代替の分離アプローチ）

[GCP](./gcp.md)[exe.dev](./exe-dev.md)