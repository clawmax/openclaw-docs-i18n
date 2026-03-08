

  インストール概要

  
# インストール

[はじめに](./start/getting-started.md)はもうお読みですか？ それなら準備は完了です — このページは、代替インストール方法、プラットフォーム固有の手順、およびメンテナンスに関するものです。

## システム要件

-   **[Node 22+](./install/node.md)** ([インストーラースクリプト](#install-methods)は、Nodeが存在しない場合にインストールします)
-   macOS、Linux、または Windows
-   ソースからビルドする場合のみ `pnpm`

> **ℹ️** Windowsでは、[WSL2](https://learn.microsoft.com/en-us/windows/wsl/install) の下でOpenClawを実行することを強くお勧めします。

## インストール方法

> **💡** **インストーラースクリプト** はOpenClawをインストールする推奨方法です。Nodeの検出、インストール、およびオンボーディングを1ステップで処理します。

 

> **⚠️** VPS/クラウドホストの場合、可能であればサードパーティ製の「1クリック」マーケットプレイスイメージは避けてください。クリーンなベースOSイメージ（例：Ubuntu LTS）を選択し、インストーラースクリプトを使用して自分でOpenClawをインストールすることをお勧めします。

 

CLIをダウンロードし、npm経由でグローバルにインストールし、オンボーディングウィザードを起動します。

以上です — スクリプトがNodeの検出、インストール、オンボーディングを処理します。オンボーディングをスキップしてバイナリのみをインストールするには:

すべてのフラグ、環境変数、およびCI/自動化オプションについては、[インストーラーの内部](./install/installer.md)を参照してください。

すでにNode 22+があり、インストールを自分で管理したい場合:

コントリビューター、またはローカルのチェックアウトから実行したい方へ。

1

[

](#)

リポジトリのクローンとビルド

[OpenClawリポジトリ](https://github.com/openclaw/openclaw)をクローンしてビルドします:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build
pnpm build
```

2

[

](#)

CLIのリンク

`openclaw` コマンドをグローバルに利用可能にします:

```bash
pnpm link --global
```

または、リンクをスキップし、リポジトリ内から `pnpm openclaw ...` でコマンドを実行することもできます。

3

[

](#)

オンボーディングの実行

```bash
openclaw onboard --install-daemon
```

より深い開発ワークフローについては、[セットアップ](./start/setup.md)を参照してください。

## その他のインストール方法

## インストール後

すべてが正常に動作していることを確認します:

```bash
openclaw doctor         # 設定の問題をチェック
openclaw status         # ゲートウェイのステータス
openclaw dashboard      # ブラウザUIを開く
```

カスタムランタイムパスが必要な場合は、以下を使用します:

-   `OPENCLAW_HOME` - ホームディレクトリベースの内部パス用
-   `OPENCLAW_STATE_DIR` - 変更可能な状態の保存場所用
-   `OPENCLAW_CONFIG_PATH` - 設定ファイルの場所用

優先順位と詳細については、[環境変数](./help/environment.md)を参照してください。

## トラブルシューティング: openclaw が見つからない

簡単な診断:

```bash
node -v
npm -v
npm prefix -g
echo "$PATH"
```

`$(npm prefix -g)/bin` (macOS/Linux) または `$(npm prefix -g)` (Windows) が `$PATH` に**含まれていない**場合、シェルはグローバルnpmバイナリ（`openclaw`を含む）を見つけることができません。修正 — シェルの起動ファイル (`~/.zshrc` または `~/.bashrc`) に追加します:

```bash
export PATH="$(npm prefix -g)/bin:$PATH"
```

Windowsでは、`npm prefix -g` の出力をPATHに追加します。その後、新しいターミナルを開きます（またはzshでは `rehash`、bashでは `hash -r` を実行します）。

## 更新 / アンインストール

[インストーラーの内部](./install/installer.md)