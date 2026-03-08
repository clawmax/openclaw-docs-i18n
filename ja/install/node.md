title: "OpenClaw AI ランタイム要件のための Node.js インストール"
description: "OpenClaw AI 用に Node.js 22+ をインストールおよび設定する方法を学びます。バージョンの確認、バージョンマネージャーの使用、一般的なインストールエラーのトラブルシューティングを行います。"
keywords: ["node.js インストール", "openclaw node ランタイム", "node バージョンマネージャー", "npm グローバルパス", "fnm", "nvm", "openclaw トラブルシューティング", "node 22"]
---

  Node ランタイム

  
# Node.js

OpenClaw には **Node 22 以降**が必要です。[インストーラースクリプト](../install.md#install-methods)は Node を自動的に検出してインストールします — このページは、自分で Node をセットアップし、すべてが正しく設定されていることを確認したい場合（バージョン、PATH、グローバルインストール）のためのものです。

## バージョンの確認

```bash
node -v
```

これが `v22.x.x` 以上を表示すれば問題ありません。Node がインストールされていないか、バージョンが古すぎる場合は、以下のインストール方法を選択してください。

## Node のインストール

 

バージョンマネージャーを使用すると、Node バージョンを簡単に切り替えることができます。一般的な選択肢:

-   [**fnm**](https://github.com/Schniz/fnm) — 高速、クロスプラットフォーム
-   [**nvm**](https://github.com/nvm-sh/nvm) — macOS/Linux で広く使用
-   [**mise**](https://mise.jdx.dev/) — 多言語対応 (Node, Python, Ruby など)

fnm の例:

```bash
fnm install 22
fnm use 22
```

バージョンマネージャーがシェルのスタートアップファイル (`~/.zshrc` または `~/.bashrc`) で初期化されていることを確認してください。そうでない場合、PATH に Node の bin ディレクトリが含まれないため、新しいターミナルセッションで `openclaw` が見つからない可能性があります。

## トラブルシューティング

### openclaw: command not found

これはほぼ常に、npm のグローバル bin ディレクトリが PATH にないことを意味します。

### ステップ 1: グローバル npm プレフィックスを確認する

```bash
npm prefix -g
```

### ステップ 2: PATH に含まれているか確認する

```bash
echo "$PATH"
```

出力に `<npm-prefix>/bin` (macOS/Linux) または `<npm-prefix>` (Windows) が含まれているか探します。

### ステップ 3: シェルのスタートアップファイルに追加する

```bash
export PATH="$(npm prefix -g)/bin:$PATH"
```

`npm prefix -g` の出力を、設定 → システム → 環境変数からシステム PATH に追加します。

### npm install -g でのパーミッションエラー (Linux)

`EACCES` エラーが表示される場合は、npm のグローバルプレフィックスをユーザーが書き込み可能なディレクトリに切り替えます:

```bash
mkdir -p "$HOME/.npm-global"
npm config set prefix "$HOME/.npm-global"
export PATH="$HOME/.npm-global/bin:$PATH"
```

`export PATH=...` の行を `~/.bashrc` または `~/.zshrc` に追加して永続化します。

[診断フラグ](../diagnostics/flags.md)[セッション管理詳細解説](../reference/session-management-compaction.md)