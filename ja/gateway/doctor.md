

  設定と運用

  
# Doctor

`openclaw doctor` は、OpenClaw の修復および移行ツールです。古くなった設定や状態を修正し、ヘルスチェックを行い、実行可能な修復手順を提供します。

## クイックスタート

```bash
openclaw doctor
```

### ヘッドレス / 自動化

```bash
openclaw doctor --yes
```

プロンプトなしでデフォルトを受け入れます（再起動、サービス、サンドボックスの修復手順が適用可能な場合を含む）。

```bash
openclaw doctor --repair
```

プロンプトなしで推奨される修復を適用します（安全な場合の修復と再起動）。

```bash
openclaw doctor --repair --force
```

積極的な修復も適用します（カスタムスーパーバイザー設定を上書きします）。

```bash
openclaw doctor --non-interactive
```

プロンプトなしで実行し、安全な移行のみを適用します（設定の正規化とディスク上の状態の移動）。人間の確認を必要とする再起動、サービス、サンドボックスのアクションはスキップします。検出されたレガシー状態の移行は自動的に実行されます。

```bash
openclaw doctor --deep
```

システムサービスをスキャンして追加のゲートウェイインストールを検出します（launchd/systemd/schtasks）。書き込み前に変更を確認したい場合は、まず設定ファイルを開いてください：

```bash
cat ~/.openclaw/openclaw.json
```

## 機能概要

-   オプションの事前アップデート（git インストールの場合、対話モードのみ）。
-   UI プロトコルの鮮度チェック（プロトコルスキーマが新しい場合に Control UI をリビルド）。
-   ヘルスチェック + 再起動プロンプト。
-   スキルステータスの概要（利用可能/不足/ブロック）。
-   レガシー値の設定正規化。
-   OpenCode Zen プロバイダーオーバーライドの警告 (`models.providers.opencode`)。
-   レガシーディスク状態の移行（セッション/エージェントディレクトリ/WhatsApp 認証）。
-   状態の整合性とパーミッションチェック（セッション、トランスクリプト、状態ディレクトリ）。
-   ローカル実行時の設定ファイルパーミッションチェック (chmod 600)。
-   モデル認証ヘルス：OAuth の有効期限をチェックし、期限切れ間近のトークンを更新でき、認証プロファイルのクールダウン/無効状態を報告します。
-   追加のワークスペースディレクトリ検出 (`~/openclaw`)。
-   サンドボックスが有効な場合のサンドボックスイメージ修復。
-   レガシーサービスの移行と追加ゲートウェイの検出。
-   ゲートウェイランタイムチェック（サービスはインストールされているが実行されていない、キャッシュされた launchd ラベル）。
-   チャネルステータスの警告（実行中のゲートウェイからプローブ）。
-   スーパーバイザー設定の監査（launchd/systemd/schtasks）とオプションの修復。
-   ゲートウェイランタイムのベストプラクティスチェック（Node 対 Bun、バージョンマネージャーのパス）。
-   ゲートウェイポート競合診断（デフォルト `18789`）。
-   オープンな DM ポリシーに対するセキュリティ警告。
-   ローカルトークンモードのゲートウェイ認証チェック（トークンソースが存在しない場合にトークン生成を提案。トークン SecretRef 設定は上書きしません）。
-   Linux での systemd linger チェック。
-   ソースインストールのチェック（pnpm ワークスペースの不一致、不足している UI アセット、不足している tsx バイナリ）。
-   更新された設定とウィザードメタデータを書き込みます。

## 詳細な動作と理由

### 0) オプションのアップデート（git インストール）

これが git チェックアウトで、doctor が対話的に実行されている場合、doctor を実行する前にアップデート（フェッチ/リベース/ビルド）を提案します。

### 1) 設定の正規化

設定にレガシーな値の形式（例えば、チャネル固有のオーバーライドがない `messages.ackReaction`）が含まれている場合、doctor はそれらを現在のスキーマに正規化します。

### 2) レガシー設定キーの移行

設定に非推奨のキーが含まれている場合、他のコマンドは実行を拒否し、`openclaw doctor` の実行を求めます。Doctor は以下を行います：

-   どのレガシーキーが見つかったかを説明します。
-   適用した移行を表示します。
-   更新されたスキーマで `~/.openclaw/openclaw.json` を書き換えます。

ゲートウェイもまた、起動時にレガシー設定フォーマットを検出すると doctor 移行を自動実行するため、手動介入なしで古い設定が修復されます。現在の移行：

-   `routing.allowFrom` → `channels.whatsapp.allowFrom`
-   `routing.groupChat.requireMention` → `channels.whatsapp/telegram/imessage.groups."*".requireMention`
-   `routing.groupChat.historyLimit` → `messages.groupChat.historyLimit`
-   `routing.groupChat.mentionPatterns` → `messages.groupChat.mentionPatterns`
-   `routing.queue` → `messages.queue`
-   `routing.bindings` → トップレベルの `bindings`
-   `routing.agents`/`routing.defaultAgentId` → `agents.list` + `agents.list[].default`
-   `routing.agentToAgent` → `tools.agentToAgent`
-   `routing.transcribeAudio` → `tools.media.audio.models`
-   `bindings[].match.accountID` → `bindings[].match.accountId`
-   名前付き `accounts` を持つが `accounts.default` が欠けているチャネルの場合、アカウントスコープのトップレベル単一アカウントチャネル値を `channels..accounts.default` に移動します（存在する場合）
-   `identity` → `agents.list[].identity`
-   `agent.*` → `agents.defaults` + `tools.*` (tools/elevated/exec/sandbox/subagents)
-   `agent.model`/`allowedModels`/`modelAliases`/`modelFallbacks`/`imageModelFallbacks` → `agents.defaults.models` + `agents.defaults.model.primary/fallbacks` + `agents.defaults.imageModel.primary/fallbacks`
-   `browser.ssrfPolicy.allowPrivateNetwork` → `browser.ssrfPolicy.dangerouslyAllowPrivateNetwork`

Doctor の警告には、マルチアカウントチャネルに対するアカウントデフォルトのガイダンスも含まれます：

-   2つ以上の `channels..accounts` エントリが `channels..defaultAccount` または `accounts.default` なしで設定されている場合、doctor はフォールバックルーティングが予期しないアカウントを選択する可能性があると警告します。
-   `channels..defaultAccount` が未知のアカウント ID に設定されている場合、doctor は警告し、設定済みのアカウント ID をリストします。

### 2b) OpenCode Zen プロバイダーオーバーライド

`models.providers.opencode`（または `opencode-zen`）を手動で追加した場合、`@mariozechner/pi-ai` からの組み込み OpenCode Zen カタログをオーバーライドします。これにより、すべてのモデルが単一の API に強制されたり、コストがゼロになったりする可能性があります。Doctor は警告を出し、オーバーライドを削除してモデルごとの API ルーティングとコストを復元できるようにします。

### 3) レガシー状態の移行（ディスクレイアウト）

Doctor は、古いディスク上のレイアウトを現在の構造に移行できます：

-   セッションストア + トランスクリプト：
    -   `~/.openclaw/sessions/` から `~/.openclaw/agents//sessions/` へ
-   エージェントディレクトリ：
    -   `~/.openclaw/agent/` から `~/.openclaw/agents//agent/` へ
-   WhatsApp 認証状態 (Baileys)：
    -   レガシーな `~/.openclaw/credentials/*.json` (`oauth.json` を除く) から
    -   `~/.openclaw/credentials/whatsapp//...` へ（デフォルトアカウント ID: `default`）

これらの移行はベストエフォートで冪等性があります。doctor はバックアップとしてレガシーフォルダを残す場合に警告を発します。ゲートウェイ/CLI も起動時にレガシーセッションとエージェントディレクトリを自動移行するため、手動で doctor を実行しなくても履歴/認証/モデルがエージェントごとのパスに配置されます。WhatsApp 認証は意図的に `openclaw doctor` 経由でのみ移行されます。

### 4) 状態の整合性チェック（セッション永続性、ルーティング、安全性）

状態ディレクトリは運用の脳幹です。これが消えると、セッション、認証情報、ログ、設定が失われます（他の場所にバックアップがない限り）。Doctor は以下をチェックします：

-   **状態ディレクトリの欠落**: 壊滅的な状態損失について警告し、ディレクトリの再作成を促し、不足しているデータを回復できないことを思い出させます。
-   **状態ディレクトリのパーミッション**: 書き込み可能か確認します。パーミッションの修復を提案し（所有者/グループの不一致が検出された場合は `chown` のヒントを出力）、必要に応じて修復を提案します。
-   **macOS クラウド同期された状態ディレクトリ**: 状態が iCloud Drive (`~/Library/Mobile Documents/com~apple~CloudDocs/...`) または `~/Library/CloudStorage/...` 配下にある場合に警告します。同期されたパスは I/O が遅くなり、ロック/同期競合を引き起こす可能性があるためです。
-   **Linux SD または eMMC 状態ディレクトリ**: 状態が `mmcblk*` マウントソースに解決される場合に警告します。SD または eMMC バックのランダム I/O は、セッションや認証情報の書き込み下で遅くなり、消耗が早くなる可能性があるためです。
-   **セッションディレクトリの欠落**: `sessions/` とセッションストアディレクトリは履歴を永続化し、`ENOENT` クラッシュを回避するために必要です。
-   **トランスクリプトの不一致**: 最近のセッションエントリにトランスクリプトファイルが欠けている場合に警告します。
-   **メインセッションの「1行 JSONL」**: メイントランスクリプトが1行しかない場合（履歴が蓄積されていない）にフラグを立てます。
-   **複数の状態ディレクトリ**: ホームディレクトリ間で複数の `~/.openclaw` フォルダが存在する場合、または `OPENCLAW_STATE_DIR` が別の場所を指している場合に警告します（履歴がインストール間で分割される可能性があります）。
-   **リモートモードのリマインダー**: `gateway.mode=remote` の場合、doctor はリモートホストで実行するようリマインダーを表示します（状態はそこに存在します）。
-   **設定ファイルのパーミッション**: `~/.openclaw/openclaw.json` がグループまたはワールド読み取り可能な場合に警告し、`600` に厳格化するよう提案します。

### 5) モデル認証ヘルス（OAuth 有効期限）

Doctor は認証ストア内の OAuth プロファイルを検査し、トークンの期限切れ間近/期限切れ時に警告し、安全な場合にそれらを更新できます。Anthropic Claude Code プロファイルが古い場合、`claude setup-token` の実行（またはセットアップトークンの貼り付け）を提案します。更新プロンプトは対話的に（TTYで）実行されている場合にのみ表示されます。`--non-interactive` は更新の試行をスキップします。Doctor はまた、以下の理由で一時的に使用できない認証プロファイルを報告します：

-   短いクールダウン（レート制限/タイムアウト/認証失敗）
-   長い無効化（課金/クレジット失敗）

### 6) フックモデルの検証

`hooks.gmail.model` が設定されている場合、doctor はモデル参照をカタログと許可リストに対して検証し、解決できない場合や許可されていない場合に警告します。

### 7) サンドボックスイメージの修復

サンドボックスが有効な場合、doctor は Docker イメージをチェックし、現在のイメージが欠落している場合にビルドまたはレガシー名への切り替えを提案します。

### 8) ゲートウェイサービスの移行とクリーンアップのヒント

Doctor はレガシーゲートウェイサービス（launchd/systemd/schtasks）を検出し、それらを削除して現在のゲートウェイポートを使用して OpenClaw サービスをインストールするよう提案します。また、追加のゲートウェイのようなサービスをスキャンしてクリーンアップのヒントを表示することもできます。プロファイル名付きの OpenClaw ゲートウェイサービスはファーストクラスと見なされ、「追加」としてフラグは立てられません。

### 9) セキュリティ警告

Doctor は、プロバイダーが許可リストなしで DM に開放されている場合、またはポリシーが危険な方法で設定されている場合に警告を発します。

### 10) systemd linger (Linux)

systemd ユーザーサービスとして実行されている場合、doctor は linger が有効になっていることを確認し、ログアウト後もゲートウェイが生き続けるようにします。

### 11) スキルステータス

Doctor は、現在のワークスペースに対する利用可能/不足/ブロックされたスキルの簡単な概要を表示します。

### 12) ゲートウェイ認証チェック（ローカルトークン）

Doctor はローカルゲートウェイトークン認証の準備状況をチェックします。

-   トークンモードでトークンが必要だがトークンソースが存在しない場合、doctor はトークンの生成を提案します。
-   `gateway.auth.token` が SecretRef で管理されているが利用できない場合、doctor は警告し、プレーンテキストで上書きしません。
-   `openclaw doctor --generate-gateway-token` は、トークン SecretRef が設定されていない場合にのみ生成を強制します。

### 12b) 読み取り専用 SecretRef 対応の修復

一部の修復フローでは、ランタイムのフェイルファスト動作を弱めることなく、設定された認証情報を検査する必要があります。

-   `openclaw doctor --fix` は、ターゲット設定修復のために status 系コマンドと同じ読み取り専用 SecretRef サマリーモデルを使用するようになりました。
-   例：Telegram `allowFrom` / `groupAllowFrom` `@username` 修復は、利用可能な場合に設定されたボット認証情報を使用しようとします。
-   Telegram ボットトークンが SecretRef 経由で設定されているが現在のコマンドパスで利用できない場合、doctor は認証情報が設定されているが利用できないと報告し、クラッシュしたりトークンが欠落していると誤って報告する代わりに自動解決をスキップします。

### 13) ゲートウェイヘルスチェック + 再起動

Doctor はヘルスチェックを実行し、ゲートウェイが不健全に見える場合に再起動を提案します。

### 14) チャネルステータスの警告

ゲートウェイが健全な場合、doctor はチャネルステータスプローブを実行し、推奨される修正を含む警告を報告します。

### 15) スーパーバイザー設定の監査 + 修復

Doctor はインストールされたスーパーバイザー設定（launchd/systemd/schtasks）をチェックし、欠落しているまたは古いデフォルト（例：systemd の network-online 依存関係と再起動遅延）がないか確認します。不一致が見つかった場合、更新を推奨し、サービスファイル/タスクを現在のデフォルトに書き換えることができます。注意：

-   `openclaw doctor` はスーパーバイザー設定を書き換える前にプロンプトを表示します。
-   `openclaw doctor --yes` はデフォルトの修復プロンプトを受け入れます。
-   `openclaw doctor --repair` はプロンプトなしで推奨される修正を適用します。
-   `openclaw doctor --repair --force` はカスタムスーパーバイザー設定を上書きします。
-   トークン認証にトークンが必要で `gateway.auth.token` が SecretRef で管理されている場合、doctor サービスインストール/修復は SecretRef を検証しますが、解決されたプレーンテキストトークン値をスーパーバイザーサービスの環境メタデータに永続化しません。
-   トークン認証にトークンが必要で、設定されたトークン SecretRef が解決されていない場合、doctor はインストール/修復パスを実行可能なガイダンスとともにブロックします。
-   `gateway.auth.token` と `gateway.auth.password` の両方が設定されていて `gateway.auth.mode` が設定されていない場合、doctor はモードが明示的に設定されるまでインストール/修復をブロックします。
-   常に `openclaw gateway install --force` で完全な書き換えを強制できます。

### 16) ゲートウェイランタイム + ポート診断

Doctor はサービスのランタイム（PID、最終終了ステータス）を検査し、サービスがインストールされているが実際には実行されていない場合に警告します。また、ゲートウェイポート（デフォルト `18789`）でのポート競合をチェックし、考えられる原因（ゲートウェイが既に実行中、SSH トンネル）を報告します。

### 17) ゲートウェイランタイムのベストプラクティス

Doctor は、ゲートウェイサービスが Bun またはバージョン管理された Node パス（`nvm`、`fnm`、`volta`、`asdf` など）で実行されている場合に警告します。WhatsApp + Telegram チャネルは Node を必要とし、バージョンマネージャーのパスはアップグレード後に壊れる可能性があります（サービスがシェルの初期化をロードしないため）。Doctor は、利用可能な場合（Homebrew/apt/choco）にシステム Node インストールへの移行を提案します。

### 18) 設定書き込み + ウィザードメタデータ

Doctor は設定の変更を永続化し、doctor 実行を記録するためにウィザードメタデータをスタンプします。

### 19) ワークスペースのヒント（バックアップ + メモリシステム）

Doctor は、ワークスペースメモリシステムが欠落している場合にそれを提案し、ワークスペースがまだ git 管理下にない場合にバックアップのヒントを表示します。ワークスペース構造と git バックアップ（推奨：プライベート GitHub または GitLab）の完全なガイドについては、[/concepts/agent-workspace](../concepts/agent-workspace.md) を参照してください。

[ハートビート](./heartbeat.md)[ロギング](./logging.md)