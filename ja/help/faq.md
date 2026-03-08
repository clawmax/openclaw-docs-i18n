

  ヘルプ

  
# FAQ

実世界のセットアップ（ローカル開発、VPS、マルチエージェント、OAuth/APIキー、モデルフェイルオーバー）に関する迅速な回答と詳細なトラブルシューティング。ランタイム診断については、[トラブルシューティング](../gateway/troubleshooting.md)を参照してください。完全な設定リファレンスについては、[設定](../gateway/configuration.md)を参照してください。

## 目次

-   \[クイックスタートと初回実行セットアップ\]
    -   [行き詰まりました。最も速く解決する方法は？](#im-stuck-whats-the-fastest-way-to-get-unstuck)
    -   [OpenClawのインストールとセットアップのおすすめ方法は？](#whats-the-recommended-way-to-install-and-set-up-openclaw)
    -   [オンボーディング後にダッシュボードを開くには？](#how-do-i-open-the-dashboard-after-onboarding)
    -   [ダッシュボードの認証（トークン）をlocalhostとリモートで行うには？](#how-do-i-authenticate-the-dashboard-token-on-localhost-vs-remote)
    -   [必要なランタイムは？](#what-runtime-do-i-need)
    -   [Raspberry Piで動作しますか？](#does-it-run-on-raspberry-pi)
    -   [Raspberry Piインストールのコツは？](#any-tips-for-raspberry-pi-installs)
    -   ["wake up my friend"で止まる / オンボーディングが孵化しない。どうすれば？](#it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now)
    -   [新しいマシン（Mac mini）にセットアップを移行し、オンボーディングをやり直さずに済ませられますか？](#can-i-migrate-my-setup-to-a-new-machine-mac-mini-without-redoing-onboarding)
    -   [最新バージョンの新機能はどこで確認できますか？](#where-do-i-see-what-is-new-in-the-latest-version)
    -   [docs.openclaw.aiにアクセスできません（SSLエラー）。どうすれば？](#i-cant-access-docsopenclawai-ssl-error-what-now)
    -   [安定版とベータ版の違いは？](#whats-the-difference-between-stable-and-beta)
    -   [ベータ版をインストールするには？また、ベータ版と開発版の違いは？](#how-do-i-install-the-beta-version-and-whats-the-difference-between-beta-and-dev)
    -   [最新のビットを試すには？](#how-do-i-try-the-latest-bits)
    -   [インストールとオンボーディングには通常どのくらい時間がかかりますか？](#how-long-does-install-and-onboarding-usually-take)
    -   [インストーラーが固まった？より詳細なフィードバックを得るには？](#installer-stuck-how-do-i-get-more-feedback)
    -   [Windowsインストールでgitが見つからない、またはopenclawが認識されないと言われる](#windows-install-says-git-not-found-or-openclaw-not-recognized)
    -   [Windowsのexec出力が文字化けした中国語テキストを表示する。どうすれば？](#windows-exec-output-shows-garbled-chinese-text-what-should-i-do)
    -   [ドキュメントに答えがありませんでした - より良い回答を得るには？](#the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer)
    -   [LinuxにOpenClawをインストールするには？](#how-do-i-install-openclaw-on-linux)
    -   [VPSにOpenClawをインストールするには？](#how-do-i-install-openclaw-on-a-vps)
    -   [クラウド/VPSインストールガイドはどこにありますか？](#where-are-the-cloudvps-install-guides)
    -   [OpenClaw自身に更新を依頼できますか？](#can-i-ask-openclaw-to-update-itself)
    -   [オンボーディングウィザードは実際に何をしますか？](#what-does-the-onboarding-wizard-actually-do)
    -   [これを実行するためにClaudeやOpenAIのサブスクリプションが必要ですか？](#do-i-need-a-claude-or-openai-subscription-to-run-this)
    -   [APIキーなしでClaude Maxサブスクリプションを使用できますか？](#can-i-use-claude-max-subscription-without-an-api-key)
    -   [Anthropicの「setup-token」認証はどのように機能しますか？](#how-does-anthropic-setuptoken-auth-work)
    -   [Anthropicのsetup-tokenはどこで見つけられますか？](#where-do-i-find-an-anthropic-setuptoken)
    -   [Claudeサブスクリプション認証（Claude ProまたはMax）をサポートしていますか？](#do-you-support-claude-subscription-auth-claude-pro-or-max)
    -   [Anthropicから`HTTP 429: rate_limit_error`が表示されるのはなぜですか？](#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)
    -   [AWS Bedrockはサポートされていますか？](#is-aws-bedrock-supported)
    -   [Codex認証はどのように機能しますか？](#how-does-codex-auth-work)
    -   [OpenAIサブスクリプション認証（Codex OAuth）をサポートしていますか？](#do-you-support-openai-subscription-auth-codex-oauth)
    -   [Gemini CLI OAuthをセットアップするには？](#how-do-i-set-up-gemini-cli-oauth)
    -   [カジュアルなチャットにはローカルモデルで大丈夫ですか？](#is-a-local-model-ok-for-casual-chats)
    -   [ホストされたモデルのトラフィックを特定のリージョンに保つには？](#how-do-i-keep-hosted-model-traffic-in-a-specific-region)
    -   [これをインストールするためにMac Miniを購入する必要がありますか？](#do-i-have-to-buy-a-mac-mini-to-install-this)
    -   [iMessageサポートのためにMac miniが必要ですか？](#do-i-need-a-mac-mini-for-imessage-support)
    -   [OpenClawを実行するためにMac miniを購入した場合、MacBook Proに接続できますか？](#if-i-buy-a-mac-mini-to-run-openclaw-can-i-connect-it-to-my-macbook-pro)
    -   [Bunを使用できますか？](#can-i-use-bun)
    -   [Telegram: `allowFrom`には何を入れますか？](#telegram-what-goes-in-allowfrom)
    -   [複数の人が1つのWhatsApp番号を異なるOpenClawインスタンスで使用できますか？](#can-multiple-people-use-one-whatsapp-number-with-different-openclaw-instances)
    -   [「高速チャット」エージェントと「コーディング用Opus」エージェントを実行できますか？](#can-i-run-a-fast-chat-agent-and-an-opus-for-coding-agent)
    -   [HomebrewはLinuxで動作しますか？](#does-homebrew-work-on-linux)
    -   [ハッカブル（git）インストールとnpmインストールの違いは？](#whats-the-difference-between-the-hackable-git-install-and-npm-install)
    -   [後でnpmとgitインストールを切り替えられますか？](#can-i-switch-between-npm-and-git-installs-later)
    -   [Gatewayをラップトップで実行すべきか、VPSで実行すべきか？](#should-i-run-the-gateway-on-my-laptop-or-a-vps)
    -   [OpenClawを専用マシンで実行することはどのくらい重要ですか？](#how-important-is-it-to-run-openclaw-on-a-dedicated-machine)
    -   [最小VPS要件と推奨OSは？](#what-are-the-minimum-vps-requirements-and-recommended-os)
    -   [VMでOpenClawを実行できますか？要件は？](#can-i-run-openclaw-in-a-vm-and-what-are-the-requirements)
-   [OpenClawとは？](#what-is-openclaw)
    -   [OpenClawとは、一言で言うと？](#what-is-openclaw-in-one-paragraph)
    -   [価値提案は？](#whats-the-value-proposition)
    -   [セットアップ直後に最初に何をすべきですか？](#i-just-set-it-up-what-should-i-do-first)
    -   [OpenClawの日常的な主な5つのユースケースは？](#what-are-the-top-five-everyday-use-cases-for-openclaw)
    -   [OpenClawはSaaSのリードジェネレーション、アウトリーチ、広告、ブログに役立ちますか？](#can-openclaw-help-with-lead-gen-outreach-ads-and-blogs-for-a-saas)
    -   [Web開発におけるClaude Codeとの比較での利点は？](#what-are-the-advantages-vs-claude-code-for-web-development)
-   [スキルと自動化](#skills-and-automation)
    -   [リポジトリを汚さずにスキルをカスタマイズするには？](#how-do-i-customize-skills-without-keeping-the-repo-dirty)
    -   [カスタムフォルダからスキルを読み込めますか？](#can-i-load-skills-from-a-custom-folder)
    -   [異なるタスクに異なるモデルを使用するには？](#how-can-i-use-different-models-for-different-tasks)
    -   [重い作業中にボットがフリーズします。それをオフロードするには？](#the-bot-freezes-while-doing-heavy-work-how-do-i-offload-that)
    -   [Cronやリマインダーが発火しません。何を確認すべきですか？](#cron-or-reminders-do-not-fire-what-should-i-check)
    -   [Linuxにスキルをインストールするには？](#how-do-i-install-skills-on-linux)
    -   [OpenClawはスケジュールに従って、またはバックグラウンドで継続的にタスクを実行できますか？](#can-openclaw-run-tasks-on-a-schedule-or-continuously-in-the-background)
    -   [LinuxからApple macOS専用スキルを実行できますか？](#can-i-run-apple-macos-only-skills-from-linux)
    -   [NotionやHeyGenの統合はありますか？](#do-you-have-a-notion-or-heygen-integration)
    -   [ブラウザー乗っ取り用のChrome拡張機能をインストールするには？](#how-do-i-install-the-chrome-extension-for-browser-takeover)
-   [サンドボックス化とメモリ](#sandboxing-and-memory)
    -   [専用のサンドボックス化ドキュメントはありますか？](#is-there-a-dedicated-sandboxing-doc)
    -   [ホストフォルダをサンドボックスにバインドするには？](#how-do-i-bind-a-host-folder-into-the-sandbox)
    -   [メモリはどのように機能しますか？](#how-does-memory-work)
    -   [メモリが物事を忘れ続けます。定着させるには？](#memory-keeps-forgetting-things-how-do-i-make-it-stick)
    -   [メモリは永久に保持されますか？制限は？](#does-memory-persist-forever-what-are-the-limits)
    -   [セマンティックメモリ検索にOpenAI APIキーが必要ですか？](#does-semantic-memory-search-require-an-openai-api-key)
-   [ディスク上のファイルの場所](#where-things-live-on-disk)
    -   [OpenClawで使用されるすべてのデータはローカルに保存されますか？](#is-all-data-used-with-openclaw-saved-locally)
    -   [OpenClawはデータをどこに保存しますか？](#where-does-openclaw-store-its-data)
    -   [AGENTS.md / SOUL.md / USER.md / MEMORY.mdはどこに置くべきですか？](#where-should-agentsmd-soulmd-usermd-memorymd-live)
    -   [推奨されるバックアップ戦略は？](#whats-the-recommended-backup-strategy)
    -   [OpenClawを完全にアンインストールするには？](#how-do-i-completely-uninstall-openclaw)
    -   [エージェントはワークスペース外で作業できますか？](#can-agents-work-outside-the-workspace)
    -   [リモートモードです - セッションストアはどこにありますか？](#im-in-remote-mode-where-is-the-session-store)
-   [設定の基本](#config-basics)
    -   [設定の形式は？どこにありますか？](#what-format-is-the-config-where-is-it)
    -   [`gateway.bind: "lan"`（または`"tailnet"`）を設定したら、何もリッスンしない / UIが「unauthorized」と言う](#i-set-gatewaybind-lan-or-tailnet-and-now-nothing-listens-the-ui-says-unauthorized)
    -   [なぜlocalhostでトークンが必要になったのですか？](#why-do-i-need-a-token-on-localhost-now)
    -   [設定変更後に再起動する必要がありますか？](#do-i-have-to-restart-after-changing-config)
    -   [面白いCLIタグラインを無効にするには？](#how-do-i-disable-funny-cli-taglines)
    -   [ウェブ検索（およびウェブフェッチ）を有効にするには？](#how-do-i-enable-web-search-and-web-fetch)
    -   [config.applyが設定を消去しました。回復し、これを避けるには？](#configapply-wiped-my-config-how-do-i-recover-and-avoid-this)
    -   [中央のGatewayとデバイス間の特殊化されたワーカーを実行するには？](#how-do-i-run-a-central-gateway-with-specialized-workers-across-devices)
    -   [OpenClawブラウザはヘッドレスで実行できますか？](#can-the-openclaw-browser-run-headless)
    -   [ブラウザ制御にBraveを使用するには？](#how-do-i-use-brave-for-browser-control)
-   [リモートゲートウェイとノード](#remote-gateways-and-nodes)
    -   [コマンドはTelegram、ゲートウェイ、ノード間でどのように伝播しますか？](#how-do-commands-propagate-between-telegram-the-gateway-and-nodes)
    -   [Gatewayがリモートでホストされている場合、エージェントが私のコンピューターにアクセスするには？](#how-can-my-agent-access-my-computer-if-the-gateway-is-hosted-remotely)
    -   [Tailscaleは接続されていますが、返信がありません。どうすれば？](#tailscale-is-connected-but-i-get-no-replies-what-now)
    -   [2つのOpenClawインスタンスが互いに通信できますか（ローカル + VPS）？](#can-two-openclaw-instances-talk-to-each-other-local-vps)
    -   [複数のエージェントに別々のVPSが必要ですか？](#do-i-need-separate-vpses-for-multiple-agents)
    -   [VPSからのSSHではなく、個人のラップトップ上のノードを使用する利点はありますか？](#is-there-a-benefit-to-using-a-node-on-my-personal-laptop-instead-of-ssh-from-a-vps)
    -   [ノードはゲートウェイサービスを実行しますか？](#do-nodes-run-a-gateway-service)
    -   [設定を適用するAPI / RPCの方法はありますか？](#is-there-an-api-rpc-way-to-apply-config)
    -   [初回インストールのための最小限の「健全な」設定は？](#whats-a-minimal-sane-config-for-a-first-install)
    -   [VPSにTailscaleをセットアップし、Macから接続するには？](#how-do-i-set-up-tailscale-on-a-vps-and-connect-from-my-mac)
    -   [MacノードをリモートGateway（Tailscale Serve）に接続するには？](#how-do-i-connect-a-mac-node-to-a-remote-gateway-tailscale-serve)
    -   [2台目のラップトップにインストールすべきか、それともノードを追加するだけか？](#should-i-install-on-a-second-laptop-or-just-add-a-node)
-   [環境変数と.envの読み込み](#env-vars-and-env-loading)
    -   [OpenClawは環境変数をどのように読み込みますか？](#how-does-openclaw-load-environment-variables)
    -   [「サービス経由でGatewayを起動したら、環境変数が消えました。」どうすれば？](#i-started-the-gateway-via-the-service-and-my-env-vars-disappeared-what-now)
    -   [`COPILOT_GITHUB_TOKEN`を設定しましたが、モデルステータスに「Shell env: off」と表示されます。なぜ？](#i-set-copilotgithubtoken-but-models-status-shows-shell-env-off-why)
-   [セッションと複数チャット](#sessions-and-multiple-chats)
    -   [新しい会話を開始するには？](#how-do-i-start-a-fresh-conversation)
    -   [`/new`を送信しなければ、セッションは自動的にリセットされますか？](#do-sessions-reset-automatically-if-i-never-send-new)
    -   [OpenClawインスタンスのチームを作成する方法はありますか（1つのCEOと多数のエージェント）？](#is-there-a-way-to-make-a-team-of-openclaw-instances-one-ceo-and-many-agents)
    -   [途中でコンテキストが切り詰められました。防ぐには？](#why-did-context-get-truncated-midtask-how-do-i-prevent-it)
    -   [OpenClawを完全にリセットしつつ、インストールは保持するには？](#how-do-i-completely-reset-openclaw-but-keep-it-installed)
    -   [「コンテキストが大きすぎる」エラーが発生します - リセットまたは圧縮するには？](#im-getting-context-too-large-errors-how-do-i-reset-or-compact)
    -   [「LLM request rejected: messages.content.tool\_use.input field required」と表示されるのはなぜですか？](#why-am-i-seeing-llm-request-rejected-messagescontenttool_useinput-field-required)
    -   [なぜ30分ごとにハートビートメッセージが届きますか？](#why-am-i-getting-heartbeat-messages-every-30-minutes)
    -   [WhatsAppグループに「ボットアカウント」を追加する必要がありますか？](#do-i-need-to-add-a-bot-account-to-a-whatsapp-group)
    -   [WhatsAppグループのJIDを取得するには？](#how-do-i-get-the-jid-of-a-whatsapp-group)
    -   [なぜOpenClawはグループで返信しませんか？](#why-doesnt-openclaw-reply-in-a-group)
    -   [グループ/スレッドはDMとコンテキストを共有しますか？](#do-groupsthreads-share-context-with-dms)
    -   [いくつのワークスペースとエージェントを作成できますか？](#how-many-workspaces-and-agents-can-i-create)
    -   [複数のボットやチャットを同時に実行できますか（Slack）、またどのようにセットアップすべきですか？](#can-i-run-multiple-bots-or-chats-at-the-same-time-slack-and-how-should-i-set-that-up)
-   [モデル: デフォルト、選択、エイリアス、切り替え](#models-defaults-selection-aliases-switching)
    -   [「デフォルトモデル」とは？](#what-is-the-default-model)
    -   [どのモデルをお勧めしますか？](#what-model-do-you-recommend)
    -   [設定を消去せずにモデルを切り替えるには？](#how-do-i-switch-models-without-wiping-my-config)
    -   [セルフホストモデル（llama.cpp、vLLM、Ollama）を使用できますか？](#can-i-use-selfhosted-models-llamacpp-vllm-ollama)
    -   [OpenClaw、Flawd、Krillはどのモデルを使用していますか？](#what-do-openclaw-flawd-and-krill-use-for-models)
    -   [（再起動せずに）動的にモデルを切り替えるには？](#how-do-i-switch-models-on-the-fly-without-restarting)
    -   [日常タスクにGPT 5.2、コーディングにCodex 5.3を使用できますか？](#can-i-use-gpt-52-for-daily-tasks-and-codex-53-for-coding)
    -   [「Model … is not allowed」と表示され、その後返信がないのはなぜですか？](#why-do-i-see-model-is-not-allowed-and-then-no-reply)
    -   [「Unknown model: minimax/MiniMax-M2.5」と表示されるのはなぜですか？](#why-do-i-see-unknown-model-minimaxminimaxm25)
    -   [デフォルトにMiniMaxを使用し、複雑なタスクにOpenAIを使用できますか？](#can-i-use-minimax-as-my-default-and-openai-for-complex-tasks)
    -   [opus / sonnet / gptは組み込みのショートカットですか？](#are-opus-sonnet-gpt-builtin-shortcuts)
    -   [モデルショートカット（エイリアス）を定義/上書きするには？](#how-do-i-defineoverride-model-shortcuts-aliases)
    -   [OpenRouterやZ.AIのような他のプロバイダーのモデルを追加するには？](#how-do-i-add-models-from-other-providers-like-openrouter-or-zai)
-   [モデルフェイルオーバーと「すべてのモデルが失敗しました」](#model-failover-and-all-models-failed)
    -   [フェイルオーバーはどのように機能しますか？](#how-does-failover-work)
    -   [このエラーは何を意味しますか？](#what-does-this-error-mean)
    -   [`No credentials found for profile "anthropic:default"`の修正チェックリスト](#fix-checklist-for-no-credentials-found-for-profile-anthropicdefault)
    -   [なぜGoogle Geminiも試して失敗したのですか？](#why-did-it-also-try-google-gemini-and-fail)
-   [認証プロファイル: 概要と管理方法](#auth-profiles-what-they-are-and-how-to-manage-them)
    -   [認証プロファイルとは？](#what-is-an-auth-profile)
    -   [典型的なプロファイルIDは？](#what-are-typical-profile-ids)
    -   [どの認証プロファイルが最初に試行されるかを制御できますか？](#can-i-control-which-auth-profile-is-tried-first)
    -   [OAuthとAPIキーの違いは？](#oauth-vs-api-key-whats-the-difference)
-   [Gateway: ポート、「already running」、リモートモード](#gateway-ports-already-running-and-remote-mode)
    -   [Gatewayはどのポートを使用しますか？](#what-port-does-the-gateway-use)
    -   [なぜ`openclaw gateway status`は`Runtime: running`だが`RPC probe: failed`と言うのですか？](#why-does-openclaw-gateway-status-say-runtime-running-but-rpc-probe-failed)
    -   [なぜ`openclaw gateway status`は`Config (cli)`と`Config (service)`が異なると表示するのですか？](#why-does-openclaw-gateway-status-show-config-cli-and-config-service-different)
    -   [「another gateway instance is already listening」とはどういう意味ですか？](#what-does-another-gateway-instance-is-already-listening-mean)
    -   [OpenClawをリモートモードで実行するには（クライアントが別の場所のGatewayに接続）？](#how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere)
    -   [Control UIが「unauthorized」と言う（または再接続を繰り返す）。どうすれば？](#the-control-ui-says-unauthorized-or-keeps-reconnecting-what-now)
    -   [`gateway.bind: "tailnet"`を設定したが、バインドできない / 何もリッスンしない](#i-set-gatewaybind-tailnet-but-it-cant-bind-nothing-listens)
    -   [同じホストで複数のGatewayを実行できますか？](#can-i-run-multiple-gateways-on-the-same-host)
    -   [「invalid handshake」 / code 1008とはどういう意味ですか？](#what-does-invalid-handshake-code-1008-mean)
-   [ロギングとデバッグ](#logging-and-debugging)
    -   [ログはどこにありますか？](#where-are-logs)
    -   [Gatewayサービスを開始/停止/再起動するには？](#how-do-i-startstoprestart-the-gateway-service)
    -   [Windowsでターミナルを閉じてしまいました - OpenClawを再起動するには？](#i-closed-my-terminal-on-windows-how-do-i-restart-openclaw)
    -   [Gatewayは起動していますが、返信が届きません。何を確認すべきですか？](#the-gateway-is-up-but-replies-never-arrive-what-should-i-check)
    -   [「Disconnected from gateway: no reason」 - どうすれば？](#disconnected-from-gateway-no-reason-what-now)
    -   [TelegramのsetMyCommandsがネットワークエラーで失敗します。何を確認すべきですか？](#telegram-setmycommands-fails-with-network-errors-what-should-i-check)
    -   [TUIに出力が表示されません。何を確認すべきですか？](#tui-shows-no-output-what-should-i-check)
    -   [Gatewayを完全に停止してから開始するには？](#how-do-i-completely-stop-then-start-the-gateway)
    -   [ELI5: `openclaw gateway restart` 対 `openclaw gateway`](#eli5-openclaw-gateway-restart-vs-openclaw-gateway)
    -   [何かが失敗したときに詳細を最も速く得る方法は？](#whats-the-fastest-way-to-get-more-details-when-something-fails)
-   [メディアと添付ファイル](#media-and-attachments)
    -   [スキルが画像/PDFを生成しましたが、何も送信されませんでした](#my-skill-generated-an-imagepdf-but-nothing-was-sent)
-   [セキュリティとアクセス制御](#security-and-access-control)
    -   [OpenClawをインバウンドDMに公開しても安全ですか？](#is-it-safe-to-expose-openclaw-to-inbound-dms)
    -   [プロンプトインジェクションは公開ボットだけの問題ですか？](#is-prompt-injection-only-a-concern-for-public-bots)
    -   [ボットは独自のメール、GitHubアカウント、電話番号を持つべきですか？](#should-my-bot-have-its-own-email-github-account-or-phone-number)
    -   [テキストメッセージに対する自律性を与えても安全ですか？](#can-i-give-it-autonomy-over-my-text-messages-and-is-that-safe)
    -   [個人アシスタントタスクに安価なモデルを使用できますか？](#can-i-use-cheaper-models-for-personal-assistant-tasks)
    -   [Telegramで`/start`を実行しましたが、ペアリングコードが届きませんでした](#i-ran-start-in-telegram-but-didnt-get-a-pairing-code)
    -   [WhatsApp: 連絡先にメッセージを送信しますか？ペアリングはどのように機能しますか？](#whatsapp-will-it-message-my-contacts-how-does-pairing-work)
-   [チャットコマンド、タスク中止、「止まらない」](#chat-commands-aborting-tasks-and-it-wont-stop)
    -   [内部システムメッセージがチャットに表示されないようにするには？](#how-do-i-stop-internal-system-messages-from-showing-in-chat)
    -   [実行中のタスクを停止/キャンセルするには？](#how-do-i-stopcancel-a-running-task)
    -   [TelegramからDiscordメッセージを送信するには？（「Cross-context messaging denied」）](#how-do-i-send-a-discord-message-from-telegram-crosscontext-messaging-denied)
    -   [なぜボットが「無視」しているように感じるのですか（連続メッセージ）？](#why-does-it-feel-like-the-bot-ignores-rapidfire-messages)

## 何かが壊れた場合の最初の60秒

1.  **クイックステータス（最初の確認）**
    
    コピー
    
    ```bash
    openclaw status
    ```
    
    高速なローカルサマリー: OS + 更新、ゲートウェイ/サービスの到達可能性、エージェント/セッション、プロバイダー設定 + ランタイムの問題（ゲートウェイが到達可能な場合）。
2.  **貼り付け可能なレポート（共有安全）**
    
    コピー
    
    ```bash
    openclaw status --all
    ```
    
    ログ末尾付きの読み取り専用診断（トークンは編集済み）。
3.  **デーモン + ポート状態**
    
    コピー
    
    ```bash
    openclaw gateway status
    ```
    
    スーパーバイザーのランタイムとRPC到達可能性、プローブ対象URL、およびサービスが使用した可能性のある設定を表示します。
4.  **詳細プローブ**
    
    コピー
    
    ```bash
    openclaw status --deep
    ```
    
    ゲートウェイヘルスチェック + プロバイダープローブを実行（到達可能なゲートウェイが必要）。[ヘルス](../gateway/health.md)を参照。
5.  **最新のログを追跡**
    
    コピー
    
    ```bash
    openclaw logs --follow
    ```
    
    RPCがダウンしている場合、次にフォールバック:
    
    コピー
    
    ```bash
    tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)"
    ```
    
    ファイルログはサービスログとは別です。[ロギング](../logging.md)と[トラブルシューティング](../gateway/troubleshooting.md)を参照。
6.  **ドクターを実行（修復）**
    
    コピー
    
    ```bash
    openclaw doctor
    ```
    
    設定/状態の修復/移行 + ヘルスチェックを実行。[ドクター](../gateway/doctor.md)を参照。
7.  **ゲートウェイスナップショット**
    
    コピー
    
    ```bash
    openclaw health --json
    openclaw health --verbose   # エラー時にターゲットURL + 設定パスを表示
    ```
    
    実行中のゲートウェイに完全なスナップショットを要求（WSのみ）。[ヘルス](../gateway/health.md)を参照。

## クイックスタートと初回実行セットアップ

### 行き詰まりました。最も速く解決する方法は？

あなたのマシンを**見ることができる**ローカルAIエージェントを使用してください。これはDiscordで質問するよりもはるかに効果的です。なぜなら、ほとんどの「行き詰まり」ケースは、リモートのヘルパーが検査できない**ローカル設定または環境の問題**だからです。

-   **Claude Code**: [https://www.anthropic.com/claude-code/](https://www.anthropic.com/claude-code/)
-   **OpenAI Codex**: [https://openai.com/codex/](https://openai.com/codex/)

これらのツールはリポジトリを読み取り、コマンドを実行し、ログを検査し、マシンレベルのセットアップ（PATH、サービス、権限、認証ファイル）の修正を支援できます。**完全なソースチェックアウト**をハッカブル（git）インストール経由で提供してください:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

これにより、OpenClawが**gitチェックアウトから**インストールされるため、エージェントはコード + ドキュメントを読み、実行中の正確なバージョンについて推論できます。後でいつでも`--install-method git`なしでインストーラーを再実行することで安定版に戻せます。ヒント: エージェントに修正を**計画および監督**（ステップバイステップ）させ、必要なコマンドのみを実行させてください。これにより、変更が小さく、監査が容易になります。実際のバグや修正を発見した場合は、GitHubイシューを提出するかPRを送ってください: [https://github.com/openclaw/openclaw/issues](https://github.com/openclaw/openclaw/issues) [https://github.com/openclaw/openclaw/pulls](https://github.com/open