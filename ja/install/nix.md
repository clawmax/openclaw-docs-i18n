

  その他のインストール方法

  
# Nix

NixでOpenClawを実行する推奨方法は、**[nix-openclaw](https://github.com/openclaw/nix-openclaw)** を使用することです。これはバッテリー内蔵のHome Managerモジュールです。

## クイックスタート

これをあなたのAIエージェント（Claude、Cursorなど）に貼り付けてください：

```
I want to set up nix-openclaw on my Mac.
Repository: github:openclaw/nix-openclaw

What I need you to do:
1. Check if Determinate Nix is installed (if not, install it)
2. Create a local flake at ~/code/openclaw-local using templates/agent-first/flake.nix
3. Help me create a Telegram bot (@BotFather) and get my chat ID (@userinfobot)
4. Set up secrets (bot token, model provider API key) - plain files at ~/.secrets/ is fine
5. Fill in the template placeholders and run home-manager switch
6. Verify: launchd running, bot responds to messages

Reference the nix-openclaw README for module options.
```

> **📦 完全ガイド: [github.com/openclaw/nix-openclaw](https://github.com/openclaw/nix-openclaw)** nix-openclawリポジトリがNixインストールの信頼できる情報源です。このページは概要のみを提供します。

## 得られるもの

-   Gateway + macOSアプリ + ツール（whisper、spotify、cameras） — すべて固定バージョン
-   再起動後も持続するLaunchdサービス
-   宣言的設定によるプラグインシステム
-   即時ロールバック: `home-manager switch --rollback`

* * *

## Nixモードのランタイム動作

`OPENCLAW_NIX_MODE=1` が設定されている場合（nix-openclawでは自動設定）: OpenClawは**Nixモード**をサポートし、設定を決定論的にし、自動インストールフローを無効にします。以下のようにエクスポートして有効化します：

```
OPENCLAW_NIX_MODE=1
```

macOSでは、GUIアプリはシェルの環境変数を自動的に継承しません。defaultsコマンドでもNixモードを有効化できます：

```bash
defaults write ai.openclaw.mac openclaw.nixMode -bool true
```

### 設定 + 状態パス

OpenClawはJSON5設定を `OPENCLAW_CONFIG_PATH` から読み取り、変更可能なデータを `OPENCLAW_STATE_DIR` に保存します。必要に応じて、内部パス解決に使用されるベースホームディレクトリを制御するために `OPENCLAW_HOME` も設定できます。

-   `OPENCLAW_HOME` (デフォルトの優先順位: `HOME` / `USERPROFILE` / `os.homedir()`)
-   `OPENCLAW_STATE_DIR` (デフォルト: `~/.openclaw`)
-   `OPENCLAW_CONFIG_PATH` (デフォルト: `$OPENCLAW_STATE_DIR/openclaw.json`)

Nixで実行する場合、ランタイムの状態と設定が不変ストアの外に残るように、これらを明示的にNix管理の場所に設定してください。

### Nixモードでのランタイム動作

-   自動インストールと自己変更フローは無効化されます
-   不足している依存関係はNix固有の修正メッセージを表示します
-   UIには、Nixモードが有効な場合に読み取り専用のバナーが表示されます

## パッケージングに関する注意（macOS）

macOSのパッケージングフローは、以下の場所に安定したInfo.plistテンプレートを期待します：

```
apps/macos/Sources/OpenClaw/Resources/Info.plist
```

[`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) はこのテンプレートをアプリバンドルにコピーし、動的フィールド（バンドルID、バージョン/ビルド、Git SHA、Sparkleキー）をパッチ適用します。これにより、SwiftPMパッケージングとNixビルド（完全なXcodeツールチェーンに依存しない）のためにplistを決定論的に保ちます。

## 関連リンク

-   [nix-openclaw](https://github.com/openclaw/nix-openclaw) — 完全なセットアップガイド
-   [ウィザード](../start/wizard.md) — Nix以外のCLIセットアップ
-   [Docker](./docker.md) — コンテナ化されたセットアップ

[Podman](./podman.md)[Ansible](./ansible.md)

---