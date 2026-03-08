

  最初のステップ

  
# はじめに

目標: 最小限のセットアップで、ゼロから最初の動作するチャットまで進みます。

> **ℹ️** 最速チャット: Control UIを開きます（チャンネル設定は不要）。`openclaw dashboard` を実行してブラウザでチャットするか、ゲートウェイホスト上で `http://127.0.0.1:18789/` を開きます。ドキュメント: [ダッシュボード](../web/dashboard.md) と [Control UI](../web/control-ui.md)。

## 前提条件

-   Node 22 以降

> **💡** Nodeのバージョンがわからない場合は、`node --version` で確認してください。

## クイックセットアップ (CLI)

### ステップ 1: OpenClawをインストール (推奨)

> **ℹ️** その他のインストール方法と要件: [インストール](../install.md)。

### ステップ 2: オンボーディングウィザードを実行

```bash
openclaw onboard --install-daemon
```

ウィザードは、認証、ゲートウェイ設定、オプションのチャンネルを構成します。詳細は [オンボーディングウィザード](./wizard.md) を参照してください。

### ステップ 3: ゲートウェイを確認

サービスをインストールした場合、すでに実行されているはずです:

```bash
openclaw gateway status
```

### ステップ 4: Control UIを開く

```bash
openclaw dashboard
```

 

> **✅** Control UIが読み込まれたら、ゲートウェイは使用準備が整っています。

## オプションの確認と追加機能

クイックテストやトラブルシューティングに便利です。

```bash
openclaw gateway --port 18789
```

設定済みのチャンネルが必要です。

```bash
openclaw message send --target +15555550123 --message "Hello from OpenClaw"
```

## 便利な環境変数

サービスアカウントとしてOpenClawを実行する場合や、カスタムの設定/状態保存場所を使用したい場合:

-   `OPENCLAW_HOME` は、内部パス解決に使用されるホームディレクトリを設定します。
-   `OPENCLAW_STATE_DIR` は状態ディレクトリを上書きします。
-   `OPENCLAW_CONFIG_PATH` は設定ファイルのパスを上書きします。

環境変数の完全なリファレンス: [環境変数](../help/environment.md)。

## さらに深く学ぶ

## これで得られるもの

-   実行中のゲートウェイ
-   設定済みの認証
-   Control UIへのアクセス、または接続済みのチャンネル

## 次のステップ

-   DMの安全性と承認: [ペアリング](../channels/pairing.md)
-   さらにチャンネルを接続: [チャンネル](../channels.md)
-   高度なワークフローとソースからのインストール: [セットアップ](./setup.md)

[機能](../concepts/features.md)[オンボーディング概要](./onboarding-overview.md)

---