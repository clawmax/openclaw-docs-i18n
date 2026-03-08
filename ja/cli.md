

  CLI コマンド

  
# CLI リファレンス

このページでは現在の CLI の動作について説明します。コマンドが変更された場合は、このドキュメントを更新してください。

## コマンドページ

-   [`setup`](./cli/setup.md)
-   [`onboard`](./cli/onboard.md)
-   [`configure`](./cli/configure.md)
-   [`config`](./cli/config.md)
-   [`completion`](./cli/completion.md)
-   [`doctor`](./cli/doctor.md)
-   [`dashboard`](./cli/dashboard.md)
-   [`reset`](./cli/reset.md)
-   [`uninstall`](./cli/uninstall.md)
-   [`update`](./cli/update.md)
-   [`message`](./cli/message.md)
-   [`agent`](./cli/agent.md)
-   [`agents`](./cli/agents.md)
-   [`acp`](./cli/acp.md)
-   [`status`](./cli/status.md)
-   [`health`](./cli/health.md)
-   [`sessions`](./cli/sessions.md)
-   [`gateway`](./cli/gateway.md)
-   [`logs`](./cli/logs.md)
-   [`system`](./cli/system.md)
-   [`models`](./cli/models.md)
-   [`memory`](./cli/memory.md)
-   [`directory`](./cli/directory.md)
-   [`nodes`](./cli/nodes.md)
-   [`devices`](./cli/devices.md)
-   [`node`](./cli/node.md)
-   [`approvals`](./cli/approvals.md)
-   [`sandbox`](./cli/sandbox.md)
-   [`tui`](./cli/tui.md)
-   [`browser`](./cli/browser.md)
-   [`cron`](./cli/cron.md)
-   [`dns`](./cli/dns.md)
-   [`docs`](./cli/docs.md)
-   [`hooks`](./cli/hooks.md)
-   [`webhooks`](./cli/webhooks.md)
-   [`pairing`](./cli/pairing.md)
-   [`qr`](./cli/qr.md)
-   [`plugins`](./cli/plugins.md) (プラグインコマンド)
-   [`channels`](./cli/channels.md)
-   [`security`](./cli/security.md)
-   [`secrets`](./cli/secrets.md)
-   [`skills`](./cli/skills.md)
-   [`daemon`](./cli/daemon.md) (ゲートウェイサービスコマンドのレガシーエイリアス)
-   [`clawbot`](./cli/clawbot.md) (レガシーエイリアス名前空間)
-   [`voicecall`](./cli/voicecall.md) (プラグイン; インストール済みの場合)

## グローバルフラグ

-   `--dev`: 状態を `~/.openclaw-dev` 下に分離し、デフォルトポートをシフトします。
-   `--profile `: 状態を `~/.openclaw-` 下に分離します。
-   `--no-color`: ANSI カラーを無効にします。
-   `--update`: `openclaw update` のショートハンド (ソースインストールのみ)。
-   `-V`, `--version`, `-v`: バージョンを表示して終了します。

## 出力スタイル

-   ANSI カラーとプログレスインジケーターは TTY セッションでのみレンダリングされます。
-   OSC-8 ハイパーリンクは、対応するターミナルではクリック可能なリンクとしてレンダリングされます。それ以外の場合はプレーンな URL にフォールバックします。
-   `--json` (およびサポートされている場合の `--plain`) は、クリーンな出力のためにスタイリングを無効にします。
-   `--no-color` は ANSI スタイリングを無効にします。`NO_COLOR=1` も尊重されます。
-   長時間実行されるコマンドはプログレスインジケーターを表示します (サポートされている場合は OSC 9;4)。

## カラーパレット

OpenClaw は CLI 出力にロブスターパレットを使用します。

-   `accent` (#FF5A2D): 見出し、ラベル、主要なハイライト。
-   `accentBright` (#FF7A3D): コマンド名、強調。
-   `accentDim` (#D14A22): 二次的なハイライトテキスト。
-   `info` (#FF8A5B): 情報的な値。
-   `success` (#2FBF71): 成功状態。
-   `warn` (#FFB020): 警告、フォールバック、注意。
-   `error` (#E23D2D): エラー、失敗。
-   `muted` (#8B7F77): 強調を弱める、メタデータ。

パレットの信頼できる情報源: `src/terminal/palette.ts` (別名「ロブスターシーム」)。

## コマンドツリー

```bash
openclaw [--dev] [--profile <name>] <command>
  setup
  onboard
  configure
  config
    get
    set
    unset
  completion
  doctor
  dashboard
  security
    audit
  secrets
    reload
    migrate
  reset
  uninstall
  update
  channels
    list
    status
    logs
    add
    remove
    login
    logout
  directory
  skills
    list
    info
    check
  plugins
    list
    info
    install
    enable
    disable
    doctor
  memory
    status
    index
    search
  message
  agent
  agents
    list
    add
    delete
  acp
  status
  health
  sessions
  gateway
    call
    health
    status
    probe
    discover
    install
    uninstall
    start
    stop
    restart
    run
  daemon
    status
    install
    uninstall
    start
    stop
    restart
  logs
  system
    event
    heartbeat last|enable|disable
    presence
  models
    list
    status
    set
    set-image
    aliases list|add|remove
    fallbacks list|add|remove|clear
    image-fallbacks list|add|remove|clear
    scan
    auth add|setup-token|paste-token
    auth order get|set|clear
  sandbox
    list
    recreate
    explain
  cron
    status
    list
    add
    edit
    rm
    enable
    disable
    runs
    run
  nodes
  devices
  node
    run
    status
    install
    uninstall
    start
    stop
    restart
  approvals
    get
    set
    allowlist add|remove
  browser
    status
    start
    stop
    reset-profile
    tabs
    open
    focus
    close
    profiles
    create-profile
    delete-profile
    screenshot
    snapshot
    navigate
    resize
    click
    type
    press
    hover
    drag
    select
    upload
    fill
    dialog
    wait
    evaluate
    console
    pdf
  hooks
    list
    info
    check
    enable
    disable
    install
    update
  webhooks
    gmail setup|run
  pairing
    list
    approve
  qr
  clawbot
    qr
  docs
  dns
    setup
  tui
```

注: プラグインは追加のトップレベルコマンドを追加できます (例: `openclaw voicecall`)。

## セキュリティ

-   `openclaw security audit` — 一般的なセキュリティの落とし穴について、設定とローカル状態を監査します。
-   `openclaw security audit --deep` — ベストエフォートでのライブゲートウェイプローブ。
-   `openclaw security audit --fix` — 安全なデフォルトを強化し、状態/設定の chmod を行います。

## シークレット

-   `openclaw secrets reload` — 参照を再解決し、ランタイムスナップショットをアトミックに交換します。
-   `openclaw secrets audit` — 平文の残留物、未解決の参照、優先順位のずれをスキャンします。
-   `openclaw secrets configure` — プロバイダーセットアップ + SecretRef マッピング + プレフライト/適用のためのインタラクティブヘルパー。
-   `openclaw secrets apply --from <plan.json>` — 以前に生成されたプランを適用します (`--dry-run` をサポート)。

## プラグイン

拡張機能とその設定を管理します:

-   `openclaw plugins list` — プラグインを発見します (機械出力には `--json` を使用)。
-   `openclaw plugins info ` — プラグインの詳細を表示します。
-   `openclaw plugins install <path|.tgz|npm-spec>` — プラグインをインストールします (またはプラグインパスを `plugins.load.paths` に追加します)。
-   `openclaw plugins enable ` / `disable ` — `plugins.entries..enabled` を切り替えます。
-   `openclaw plugins doctor` — プラグイン読み込みエラーを報告します。

ほとんどのプラグイン変更にはゲートウェイの再起動が必要です。[/plugin](./tools/plugin.md) を参照してください。

## メモリ

`MEMORY.md` + `memory/*.md` に対するベクター検索:

-   `openclaw memory status` — インデックス統計を表示します。
-   `openclaw memory index` — メモリファイルを再インデックスします。
-   `openclaw memory search ""` (または `--query ""`) — メモリに対するセマンティック検索。

## チャットスラッシュコマンド

チャットメッセージは `/...` コマンドをサポートします (テキストおよびネイティブ)。[/tools/slash-commands](./tools/slash-commands.md) を参照してください。ハイライト:

-   `/status` クイック診断用。
-   `/config` 永続化された設定変更用。
-   `/debug` ランタイムのみの設定オーバーライド用 (メモリ内、ディスクには保存されません; `commands.debug: true` が必要)。

## セットアップ + オンボーディング

### setup

設定とワークスペースを初期化します。オプション:

-   `--workspace `: エージェントワークスペースパス (デフォルト `~/.openclaw/workspace`)。
-   `--wizard`: オンボーディングウィザードを実行します。
-   `--non-interactive`: プロンプトなしでウィザードを実行します。
-   `--mode <local|remote>`: ウィザードモード。
-   `--remote-url `: リモートゲートウェイ URL。
-   `--remote-token `: リモートゲートウェイトークン。

ウィザードフラグが存在する場合 (`--non-interactive`, `--mode`, `--remote-url`, `--remote-token`)、ウィザードは自動的に実行されます。

### onboard

ゲートウェイ、ワークスペース、スキルをセットアップするインタラクティブウィザード。オプション:

-   `--workspace `
-   `--reset` (ウィザード前に設定 + 認証情報 + セッションをリセット)
-   `--reset-scope <config|config+creds+sessions|full>` (デフォルト `config+creds+sessions`; ワークスペースも削除するには `full` を使用)
-   `--non-interactive`
-   `--mode <local|remote>`
-   `--flow <quickstart|advanced|manual>` (manual は advanced のエイリアス)
-   `--auth-choice <setup-token|token|chutes|openai-codex|openai-api-key|openrouter-api-key|ai-gateway-api-key|moonshot-api-key|moonshot-api-key-cn|kimi-code-api-key|synthetic-api-key|venice-api-key|gemini-api-key|zai-api-key|mistral-api-key|apiKey|minimax-api|minimax-api-lightning|opencode-zen|custom-api-key|skip>`
-   `--token-provider ` (非対話型; `--auth-choice token` と共に使用)
-   `--token ` (非対話型; `--auth-choice token` と共に使用)
-   `--token-profile-id ` (非対話型; デフォルト: `:manual`)
-   `--token-expires-in ` (非対話型; 例: `365d`, `12h`)
-   `--secret-input-mode <plaintext|ref>` (デフォルト `plaintext`; 平文キーの代わりにプロバイダーデフォルトの環境参照を保存するには `ref` を使用)
-   `--anthropic-api-key `
-   `--openai-api-key `
-   `--mistral-api-key `
-   `--openrouter-api-key `
-   `--ai-gateway-api-key `
-   `--moonshot-api-key `
-   `--kimi-code-api-key `
-   `--gemini-api-key `
-   `--zai-api-key `
-   `--minimax-api-key `
-   `--opencode-zen-api-key `
-   `--custom-base-url ` (非対話型; `--auth-choice custom-api-key` と共に使用)
-   `--custom-model-id ` (非対話型; `--auth-choice custom-api-key` と共に使用)
-   `--custom-api-key ` (非対話型; オプション; `--auth-choice custom-api-key` と共に使用; 省略時は `CUSTOM_API_KEY` にフォールバック)
-   `--custom-provider-id ` (非対話型; オプションのカスタムプロバイダー ID)
-   `--custom-compatibility <openai|anthropic>` (非対話型; オプション; デフォルト `openai`)
-   `--gateway-port `
-   `--gateway-bind <loopback|lan|tailnet|auto|custom>`
-   `--gateway-auth <token|password>`
-   `--gateway-token `
-   `--gateway-token-ref-env ` (非対話型; `gateway.auth.token` を環境 SecretRef として保存; その環境変数が設定されている必要があります; `--gateway-token` と組み合わせることはできません)
-   `--gateway-password `
-   `--remote-url `
-   `--remote-token `
-   `--tailscale <off|serve|funnel>`
-   `--tailscale-reset-on-exit`
-   `--install-daemon`
-   `--no-install-daemon` (エイリアス: `--skip-daemon`)
-   `--daemon-runtime <node|bun>`
-   `--skip-channels`
-   `--skip-skills`
-   `--skip-health`
-   `--skip-ui`
-   `--node-manager <npm|pnpm|bun>` (pnpm 推奨; bun はゲートウェイランタイムには推奨されません)
-   `--json`

### configure

インタラクティブな設定ウィザード (モデル、チャネル、スキル、ゲートウェイ)。

### config

非対話型の設定ヘルパー (get/set/unset/file/validate)。サブコマンドなしで `openclaw config` を実行するとウィザードが起動します。サブコマンド:

-   `config get `: 設定値を表示します (ドット/ブラケットパス)。
-   `config set  `: 値を設定します (JSON5 または生の文字列)。
-   `config unset `: 値を削除します。
-   `config file`: アクティブな設定ファイルのパスを表示します。
-   `config validate`: ゲートウェイを起動せずに現在の設定をスキーマに対して検証します。
-   `config validate --json`: 機械可読な JSON 出力を生成します。

### doctor

ヘルスチェック + クイックフィックス (設定 + ゲートウェイ + レガシーサービス)。オプション:

-   `--no-workspace-suggestions`: ワークスペースメモリのヒントを無効にします。
-   `--yes`: プロンプトなしでデフォルトを受け入れます (ヘッドレス)。
-   `--non-interactive`: プロンプトをスキップします。安全なマイグレーションのみを適用します。
-   `--deep`: システムサービスをスキャンして追加のゲートウェイインストールを検出します。

## チャネルヘルパー

### channels

チャットチャネルアカウントを管理します (WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (プラグイン)/Signal/iMessage/MS Teams)。サブコマンド:

-   `channels list`: 設定されたチャネルと認証プロファイルを表示します。
-   `channels status`: ゲートウェイの到達可能性とチャネルの健全性をチェックします (`--probe` は追加チェックを実行します; ゲートウェイの健全性プローブには `openclaw health` または `openclaw status --deep` を使用してください)。
-   ヒント: `channels status` は、一般的な設定ミスを検出できる場合、警告と推奨される修正を表示します (その後 `openclaw doctor` を案内します)。
-   `channels logs`: ゲートウェイログファイルから最近のチャネルログを表示します。
-   `channels add`: フラグが渡されない場合はウィザードスタイルのセットアップを実行します。フラグがある場合は非対話型モードに切り替わります。
    -   単一アカウントのトップレベル設定を使用しているチャネルに非デフォルトアカウントを追加する場合、OpenClaw は新しいアカウントを書き込む前に、アカウントスコープの値を `channels..accounts.default` に移動します。
    -   非対話型の `channels add` はバインディングを自動的に作成/アップグレードしません。チャネル専用のバインディングはデフォルトアカウントに引き続きマッチします。
-   `channels remove`: デフォルトでは無効化します。プロンプトなしで設定エントリを削除するには `--delete` を渡します。
-   `channels login`: インタラクティブなチャネルログイン (WhatsApp Web のみ)。
-   `channels logout`: チャネルセッションからログアウトします (サポートされている場合)。

共通オプション:

-   `--channel `: `whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams`
-   `--account `: チャネルアカウント ID (デフォルト `default`)
-   `--name `: アカウントの表示名

`channels login` オプション:

-   `--channel ` (デフォルト `whatsapp`; `whatsapp`/`web` をサポート)
-   `--account `
-   `--verbose`

`channels logout` オプション:

-   `--channel ` (デフォルト `whatsapp`)
-   `--account `

`channels list` オプション:

-   `--no-usage`: モデルプロバイダーの使用量/クォータのスナップショットをスキップします (OAuth/API バックエンドのみ)。
-   `--json`: JSON を出力します (`--no-usage` が設定されていない限り使用量を含みます)。

`channels logs` オプション:

-   `--channel <name|all>` (デフォルト `all`)
-   `--lines ` (デフォルト `200`)
-   `--json`

詳細: [/concepts/oauth](./concepts/oauth.md) 例:

```bash
openclaw channels add --channel telegram --account alerts --name "Alerts Bot" --token $TELEGRAM_BOT_TOKEN
openclaw channels add --channel discord --account work --name "Work Bot" --token $DISCORD_BOT_TOKEN
openclaw channels remove --channel discord --account work --delete
openclaw channels status --probe
openclaw status --deep
```

### skills

利用可能なスキルと準備状況情報をリストおよび検査します。サブコマンド:

-   `skills list`: スキルをリストします (サブコマンドなしのデフォルト)。
-   `skills info `: 1つのスキルの詳細を表示します。
-   `skills check`: 準備完了と不足要件の概要。

オプション:

-   `--eligible`: 準備完了のスキルのみを表示します。
-   `--json`: JSON を出力します (スタイリングなし)。
-   `-v`, `--verbose`: 不足要件の詳細を含めます。

ヒント: スキルの検索、インストール、同期には `npx clawhub` を使用してください。

### pairing

チャネル間での DM ペアリングリクエストを承認します。サブコマンド:

-   `pairing list [channel] [--channel ] [--account ] [--json]`
-   `pairing approve   [--account ] [--notify]`
-   `pairing approve --channel  [--account ]  [--notify]`

### devices

ゲートウェイデバイスペアリングエントリとロールごとのデバイストークンを管理します。サブコマンド:

-   `devices list [--json]`
-   `devices approve [requestId] [--latest]`
-   `devices reject `
-   `devices remove `
-   `devices clear --yes [--pending]`
-   `devices rotate --device  --role  [--scope <scope...>]`
-   `devices revoke --device  --role `

### webhooks gmail

Gmail Pub/Sub フックのセットアップ + ランナー。[/automation/gmail-pubsub](./automation/gmail-pubsub.md) を参照してください。サブコマンド:

-   `webhooks gmail setup` (`--account ` が必要; `--project`, `--topic`, `--subscription`, `--label`, `--hook-url`, `--hook-token`, `--push-token`, `--bind`, `--port`, `--path`, `--include-body`, `--max-bytes`, `--renew-minutes`, `--tailscale`, `--tailscale-path`, `--tailscale-target`, `--push-endpoint`, `--json` をサポート)
-   `webhooks gmail run` (同じフラグのランタイムオーバーライド)

### dns setup

広域ディスカバリー DNS ヘルパー (CoreDNS + Tailscale)。[/gateway/discovery](./gateway/discovery.md) を参照してください。オプション:

-   `--apply`: CoreDNS 設定をインストール/更新します (sudo が必要; macOS のみ)。

## メッセージング + エージェント

### message

統一された送信メッセージング + チャネルアクション。参照: [/cli/message](./cli/message.md) サブコマンド:

-   `message send|poll|react|reactions|read|edit|delete|pin|unpin|pins|permissions|search|timeout|kick|ban`
-   `message thread <create|list|reply>`
-   `message emoji <list|upload>`
-   `message sticker <send|upload>`
-   `message role <info|add|remove>`
-   `message channel <info|list>`
-   `message member info`
-   `message voice status`
-   `message event <list|create>`

例:

-   `openclaw message send --target +15555550123 --message "Hi"`
-   `openclaw message poll --channel discord --target channel:123 --poll-question "Snack?" --poll-option Pizza --poll-option Sushi`

### agent

ゲートウェイ経由で (または `--local` 埋め込みで) 1 エージェントターンを実行します。必須:

-   `--message `

オプション:

-   `--to ` (セッションキーとオプションの配信用)
-   `--session-id `
-   `--thinking <off|minimal|low|medium|high|xhigh>` (GPT-5.2 + Codex モデルのみ)
-   `--verbose <on|full|off>`
-   `--channel <whatsapp|telegram|discord|slack|mattermost|signal|imessage|msteams>`
-   `--local`
-   `--deliver`
-   `--json`
-   `--timeout `

### agents

分離されたエージェントを管理します (ワークスペース + 認証 + ルーティング)。

#### agents list

設定されたエージェントをリストします。オプション:

-   `--json`
-   `--bindings`

#### agents add \[name\]

新しい分離エージェントを追加します。フラグ (または `--non-interactive`) が渡されない限り、ガイド付きウィザードを実行します。非対話型モードでは `--workspace` が必要です。オプション:

-   `--workspace `
-   `--model `
-   `--agent-dir `
-   `--bind <channel[:accountId]>` (繰り返し可能)
-   `--non-interactive`
-   `--json`

バインディング仕様は `channel[:accountId]` を使用します。`accountId` が省略された場合、OpenClaw はチャネルのデフォルト/プラグインフックを介してアカウントスコープを解決する場合があります。それ以外の場合は、明示的なアカウントスコープを持たないチャネルバインディングです。

#### agents bindings

ルーティングバインディングをリストします。オプション:

-   `--agent `
-   `--json`

#### agents bind

エージェントのルーティングバインディングを追加します。オプション:

-   `--agent `
-   `--bind <channel[:accountId]>` (繰り返し可能)
-   `--json`

#### agents unbind

エージェントのルーティングバインディングを削除します。オプション:

-   `--agent `
-   `--bind <channel[:accountId]>` (繰り返し可能)
-   `--all`
-   `--json`

#### agents delete &lt;id&gt;

エージェントを削除し、そのワークスペースと状態を整理します。オプション:

-   `--force`
-   `--json`

### acp

IDE をゲートウェイに接続する ACP ブリッジを実行します。完全なオプションと例については [`acp`](./cli/acp.md) を参照してください。

### status

リンクされたセッションの健全性と最近の受信者を表示します。オプション:

-   `--json`
-   `--all` (完全な診断; 読み取り専用、貼り付け可能)
-   `--deep` (チャネルをプローブ)
-   `--usage` (モデルプロバイダーの使用量/クォータを表示)
-   `--timeout `
-   `--verbose`
-   `--debug` (`--verbose` のエイリアス)

注記:

-   概要には、利用可能な場合はゲートウェイ + ノードホストサービスのステータスが含まれます。

### 使用量追跡

OpenClaw は、OAuth/API 認証情報が利用可能な場合、プロバイダーの使用量/クォータを表示できます。表示:

-   `/status` (利用可能な場合は短いプロバイダー使用量行を追加)
-   `openclaw status --usage` (完全なプロバイダー内訳を表示)
-   macOS メニューバー (コンテキスト下の使用量セクション)

注記:

-   データはプロバイダーの使用量エンドポイントから直接取得されます (推定値ではありません)。
-   プロバイダー: Anthropic、GitHub Copilot、OpenAI Codex OAuth、およびそれらのプロバイダープラグインが有効な場合は Gemini CLI/Antigravity。
-   一致する認証情報が存在しない場合、使用量は非表示になります。
-   詳細: [使用量追跡](./concepts/usage-tracking.md) を参照してください。

### health

実行中のゲートウェイから健全性を取得します。オプション:

-   `--json`
-   `--timeout `
-   `--verbose`

### sessions

保存された会話セッションをリストします。オプション:

-   `--json`
-   `--verbose`
-   `--store `
-   `--active `

## リセット / アンインストール

### reset

ローカル設定/状態をリセットします (CLI はインストールされたまま)。オプション:

-   `--scope <config|config+creds+sessions|full>`
-   `--yes`
-   `--non-interactive`
-   `--dry-run`

注記:

-   `--non-interactive` には `--scope` と `--yes` が必要です。

### uninstall

ゲートウェイサービス + ローカルデータをアンインストールします (CLI は残ります)。オプション:

-   `--service`
-   `--state`
-   `--workspace`
-   `--app`
-   `--all`
-   `--yes`
-   `--non-interactive`
-   `--dry-run`

注記:

-   `--non-interactive` には `--yes` と明示的なスコープ (または `--all`) が必要です。

## ゲートウェイ

### gateway

WebSocket ゲートウェイを実行します。オプション:

-   `--port `
-   `--bind <loopback|tailnet|lan|auto|custom>`
-   `--token `
-   `--auth <token|password>`
-   `--password `
-   `--tailscale <off|serve|funnel>`
-   `--tailscale-reset-on-exit`
-   `--allow-unconfigured`
-   `--dev`
-   `--reset` (開発設定 + 認証情報 + セッション + ワークスペースをリセット)
-   `--force` (ポート上の既存のリスナーを強制終了)
-   `--verbose`
-   `--claude-cli-logs`
-   `--ws-log <auto|full|compact>`
-   `--compact` (`--ws-log compact` のエイリアス)
-   `--raw-stream`
-   `--raw-stream-path `

### gateway service

ゲートウェイサービスを管理します (launchd/systemd/schtasks)。サブコマンド:

-   `gateway status` (デフォルトでゲートウェイ RPC をプローブします)
-   `gateway install` (サービスインストール)
-   `gateway uninstall`
-   `gateway start`
-   `gateway stop`
-   `gateway restart`

注記:

-   `gateway status` は、デフォルトでサービスの解決されたポート/設定を使用してゲートウェイ RPC をプローブします (`--url/--token/--password` でオーバーライ