

  インストール概要

  
# インストーラー内部詳細

OpenClaw は、`openclaw.ai` から提供される3つのインストーラースクリプトを同梱しています。

| スクリプト | プラットフォーム | 機能 |
| --- | --- | --- |
| [`install.sh`](#installsh) | macOS / Linux / WSL | 必要に応じて Node をインストールし、npm（デフォルト）または git 経由で OpenClaw をインストールし、オンボーディングを実行できます。 |
| [`install-cli.sh`](#install-clish) | macOS / Linux / WSL | Node + OpenClaw をローカルプレフィックス (`~/.openclaw`) にインストールします。root 権限は不要です。 |
| [`install.ps1`](#installps1) | Windows (PowerShell) | 必要に応じて Node をインストールし、npm（デフォルト）または git 経由で OpenClaw をインストールし、オンボーディングを実行できます。 |

## クイックコマンド

> **ℹ️** インストールが成功しても新しいターミナルで `openclaw` が見つからない場合は、[Node.js トラブルシューティング](./node.md#troubleshooting) を参照してください。

* * *

## install.sh

> **💡** macOS/Linux/WSL でのほとんどの対話型インストールに推奨されます。

### フロー (install.sh)

### ステップ 1: OS の検出

macOS と Linux（WSL を含む）をサポートします。macOS が検出された場合、Homebrew がなければインストールします。

### ステップ 2: Node.js 22+ の確保

Node バージョンをチェックし、必要に応じて Node 22 をインストールします（macOS では Homebrew、Linux apt/dnf/yum では NodeSource セットアップスクリプトを使用）。

### ステップ 3: Git の確保

Git がなければインストールします。

### ステップ 4: OpenClaw のインストール

-   `npm` 方式（デフォルト）: グローバル npm インストール
-   `git` 方式: リポジトリをクローン/更新し、pnpm で依存関係をインストール、ビルドし、`~/.local/bin/openclaw` にラッパーをインストールします。

### ステップ 5: インストール後のタスク

-   アップグレード時および git インストール時に `openclaw doctor --non-interactive` を実行します（ベストエフォート）
-   適切な場合（TTY が利用可能、オンボーディングが無効化されていない、ブートストラップ/設定チェックが成功）にオンボーディングを試みます
-   `SHARP_IGNORE_GLOBAL_LIBVIPS=1` をデフォルトとします

### ソースチェックアウトの検出

スクリプトが OpenClaw のチェックアウト内（`package.json` + `pnpm-workspace.yaml`）で実行された場合、以下の選択肢を提供します:

-   チェックアウトを使用する (`git`)、または
-   グローバルインストールを使用する (`npm`)

TTY が利用できず、インストール方法が設定されていない場合、デフォルトで `npm` を使用し警告を表示します。無効な方法選択または無効な `--install-method` 値の場合、スクリプトはコード `2` で終了します。

### 例 (install.sh)

| フラグ | 説明 |
| --- | --- |
| `--install-method npm\|git` | インストール方法を選択（デフォルト: `npm`）。エイリアス: `--method` |
| `--npm` | npm 方式のショートカット |
| `--git` | git 方式のショートカット。エイリアス: `--github` |
| `--version <version\|dist-tag>` | npm バージョンまたは dist-tag（デフォルト: `latest`） |
| `--beta` | 利用可能であれば beta dist-tag を使用、それ以外は `latest` にフォールバック |
| `--git-dir ` | チェックアウトディレクトリ（デフォルト: `~/openclaw`）。エイリアス: `--dir` |
| `--no-git-update` | 既存のチェックアウトに対する `git pull` をスキップ |
| `--no-prompt` | プロンプトを無効化 |
| `--no-onboard` | オンボーディングをスキップ |
| `--onboard` | オンボーディングを有効化 |
| `--dry-run` | 変更を適用せずにアクションを表示 |
| `--verbose` | デバッグ出力を有効化（`set -x`、npm notice レベルログ） |
| `--help` | 使用方法を表示（`-h`） |

| 変数 | 説明 |
| --- | --- |
| `OPENCLAW_INSTALL_METHOD=git\|npm` | インストール方法 |
| `OPENCLAW_VERSION=latest\|next\|` | npm バージョンまたは dist-tag |
| `OPENCLAW_BETA=0\|1` | 利用可能であれば beta を使用 |
| `OPENCLAW_GIT_DIR=` | チェックアウトディレクトリ |
| `OPENCLAW_GIT_UPDATE=0\|1` | git 更新の切り替え |
| `OPENCLAW_NO_PROMPT=1` | プロンプトを無効化 |
| `OPENCLAW_NO_ONBOARD=1` | オンボーディングをスキップ |
| `OPENCLAW_DRY_RUN=1` | ドライランモード |
| `OPENCLAW_VERBOSE=1` | デバッグモード |
| `OPENCLAW_NPM_LOGLEVEL=error\|warn\|notice` | npm ログレベル |
| `SHARP_IGNORE_GLOBAL_LIBVIPS=0\|1` | sharp/libvips の動作を制御（デフォルト: `1`） |

* * *

## install-cli.sh

> **ℹ️** すべてをローカルプレフィックス（デフォルト `~/.openclaw`）の下に配置し、システム Node への依存をなくしたい環境向けに設計されています。

### フロー (install-cli.sh)

### ステップ 1: ローカル Node ランタイムのインストール

Node の tarball（デフォルト `22.22.0`）を `/tools/node-v` にダウンロードし、SHA-256 を検証します。

### ステップ 2: Git の確保

Git がない場合、Linux では apt/dnf/yum、macOS では Homebrew 経由でのインストールを試みます。

### ステップ 3: プレフィックス配下に OpenClaw をインストール

`--prefix ` を使用して npm でインストールし、ラッパーを `/bin/openclaw` に書き込みます。

### 例 (install-cli.sh)

| フラグ | 説明 |
| --- | --- |
| `--prefix ` | インストールプレフィックス（デフォルト: `~/.openclaw`） |
| `--version ` | OpenClaw バージョンまたは dist-tag（デフォルト: `latest`） |
| `--node-version ` | Node バージョン（デフォルト: `22.22.0`） |
| `--json` | NDJSON イベントを出力 |
| `--onboard` | インストール後に `openclaw onboard` を実行 |
| `--no-onboard` | オンボーディングをスキップ（デフォルト） |
| `--set-npm-prefix` | Linux で、現在のプレフィックスが書き込み可能でない場合、npm プレフィックスを `~/.npm-global` に強制設定 |
| `--help` | 使用方法を表示（`-h`） |

| 変数 | 説明 |
| --- | --- |
| `OPENCLAW_PREFIX=` | インストールプレフィックス |
| `OPENCLAW_VERSION=` | OpenClaw バージョンまたは dist-tag |
| `OPENCLAW_NODE_VERSION=` | Node バージョン |
| `OPENCLAW_NO_ONBOARD=1` | オンボーディングをスキップ |
| `OPENCLAW_NPM_LOGLEVEL=error\|warn\|notice` | npm ログレベル |
| `OPENCLAW_GIT_DIR=` | レガシークリーンアップ用の検索パス（古い `Peekaboo` サブモジュールチェックアウトの削除時に使用） |
| `SHARP_IGNORE_GLOBAL_LIBVIPS=0\|1` | sharp/libvips の動作を制御（デフォルト: `1`） |

* * *

## install.ps1

### フロー (install.ps1)

### ステップ 1: PowerShell + Windows 環境の確保

PowerShell 5+ が必要です。

### ステップ 2: Node.js 22+ の確保

ない場合、winget、次に Chocolatey、次に Scoop 経由でのインストールを試みます。

### ステップ 3: OpenClaw のインストール

-   `npm` 方式（デフォルト）: 選択した `-Tag` を使用してグローバル npm インストール
-   `git` 方式: リポジトリをクローン/更新し、pnpm でインストール/ビルドし、ラッパーを `%USERPROFILE%\.local\bin\openclaw.cmd` にインストールします。

### ステップ 4: インストール後のタスク

可能な場合、必要な bin ディレクトリをユーザー PATH に追加し、アップグレード時および git インストール時に `openclaw doctor --non-interactive` を実行します（ベストエフォート）。

### 例 (install.ps1)

| フラグ | 説明 |
| --- | --- |
| `-InstallMethod npm\|git` | インストール方法（デフォルト: `npm`） |
| `-Tag ` | npm dist-tag（デフォルト: `latest`） |
| `-GitDir ` | チェックアウトディレクトリ（デフォルト: `%USERPROFILE%\openclaw`） |
| `-NoOnboard` | オンボーディングをスキップ |
| `-NoGitUpdate` | `git pull` をスキップ |
| `-DryRun` | アクションのみを表示 |

| 変数 | 説明 |
| --- | --- |
| `OPENCLAW_INSTALL_METHOD=git\|npm` | インストール方法 |
| `OPENCLAW_GIT_DIR=` | チェックアウトディレクトリ |
| `OPENCLAW_NO_ONBOARD=1` | オンボーディングをスキップ |
| `OPENCLAW_GIT_UPDATE=0` | git pull を無効化 |
| `OPENCLAW_DRY_RUN=1` | ドライランモード |

> **ℹ️** `-InstallMethod git` が使用され、Git がない場合、スクリプトは終了し、Git for Windows のリンクを表示します。

* * *

## CI と自動化

予測可能な実行のために、非対話型フラグ/環境変数を使用してください。

* * *

## トラブルシューティング

Git は `git` インストール方式に必要です。`npm` インストールの場合でも、依存関係が git URL を使用する際の `spawn git ENOENT` エラーを避けるために、Git のチェック/インストールが行われます。

一部の Linux 設定では、npm グローバルプレフィックスが root 所有のパスを指しています。`install.sh` は、プレフィックスを `~/.npm-global` に切り替え、シェル rc ファイルが存在する場合に PATH エクスポートを追加できます。

スクリプトは、sharp がシステム libvips に対してビルドされるのを避けるために、デフォルトで `SHARP_IGNORE_GLOBAL_LIBVIPS=1` とします。上書きするには:

```
SHARP_IGNORE_GLOBAL_LIBVIPS=0 curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash
```

Git for Windows をインストールし、PowerShell を再起動して、インストーラーを再実行してください。

`npm config get prefix` を実行し、そのディレクトリをユーザー PATH に追加してください（Windows では `\bin` サフィックスは不要です）。その後、PowerShell を再起動してください。

`install.ps1` は現在 `-Verbose` スイッチを公開していません。スクリプトレベルの診断には PowerShell トレースを使用してください:

```powershell
Set-PSDebug -Trace 1
& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -NoOnboard
Set-PSDebug -Trace 0
```

通常は PATH の問題です。[Node.js トラブルシューティング](./node.md#troubleshooting) を参照してください。

[インストール](../install.md)[Docker](./docker.md)