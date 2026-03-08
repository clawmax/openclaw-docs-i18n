

  macOS コンパニオンアプリ

  
# macOS 開発環境セットアップ

このガイドでは、ソースから OpenClaw macOS アプリケーションをビルドして実行するために必要な手順を説明します。

## 前提条件

アプリをビルドする前に、以下がインストールされていることを確認してください:

1.  **Xcode 26.2+**: Swift 開発に必要です。
2.  **Node.js 22+ & pnpm**: ゲートウェイ、CLI、およびパッケージングスクリプトに必要です。

## 1\. 依存関係のインストール

プロジェクト全体の依存関係をインストールします:

```bash
pnpm install
```

## 2\. アプリのビルドとパッケージ化

macOS アプリをビルドし、`dist/OpenClaw.app` にパッケージ化するには、以下を実行します:

```
./scripts/package-mac-app.sh
```

Apple Developer ID 証明書をお持ちでない場合、スクリプトは自動的に **アドホック署名** (`-`) を使用します。開発実行モード、署名フラグ、Team ID のトラブルシューティングについては、macOS アプリの README を参照してください: [https://github.com/openclaw/openclaw/blob/main/apps/macos/README.md](https://github.com/openclaw/openclaw/blob/main/apps/macos/README.md)

> **注意**: アドホック署名されたアプリはセキュリティプロンプトを引き起こす可能性があります。アプリが「Abort trap 6」で即座にクラッシュする場合は、[トラブルシューティング](#トラブルシューティング) セクションを参照してください。

## 3\. CLI のインストール

macOS アプリは、バックグラウンドタスクを管理するためにグローバルな `openclaw` CLI のインストールを想定しています。**インストールするには（推奨）:**

1.  OpenClaw アプリを開きます。
2.  **一般** 設定タブに移動します。
3.  **「CLI をインストール」** をクリックします。

または、手動でインストールします:

```bash
npm install -g openclaw@<version>
```

## トラブルシューティング

### ビルド失敗: ツールチェーンまたは SDK の不一致

macOS アプリのビルドは、最新の macOS SDK と Swift 6.2 ツールチェーンを想定しています。**システム依存関係（必須）:**

-   **ソフトウェアアップデートで利用可能な最新の macOS バージョン** (Xcode 26.2 SDK に必要)
-   **Xcode 26.2** (Swift 6.2 ツールチェーン)

**確認方法:**

```bash
xcodebuild -version
xcrun swift --version
```

バージョンが一致しない場合は、macOS/Xcode を更新してビルドを再実行してください。

### 権限付与時のアプリクラッシュ

**音声認識** または **マイク** アクセスを許可しようとしたときにアプリがクラッシュする場合、破損した TCC キャッシュまたは署名の不一致が原因である可能性があります。**修正方法:**

1.  TCC 権限をリセットします:
    
    Copy
    
    ```bash
    tccutil reset All ai.openclaw.mac.debug
    ```
    
2.  それでも失敗する場合は、[`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) 内の `BUNDLE_ID` を一時的に変更して、macOS から「クリーンスレート」を強制します。

### ゲートウェイが「起動中…」のまま

ゲートウェイのステータスが「起動中…」のままの場合は、ゾンビプロセスがポートを保持していないか確認してください:

```bash
openclaw gateway status
openclaw gateway stop

# LaunchAgent を使用していない場合（開発モード / 手動実行）、リスナーを探します:
lsof -nP -iTCP:18789 -sTCP:LISTEN
```

手動実行がポートを保持している場合は、そのプロセスを停止します (Ctrl+C)。最終手段として、上記で見つけた PID を強制終了します。

[Raspberry Pi](../raspberry-pi.md)[メニューバー](./menu-bar.md)