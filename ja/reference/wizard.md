

  技術リファレンス

  
# オンボーディングウィザード リファレンス

これは `openclaw onboard` CLI ウィザードの完全なリファレンスです。概要については、[オンボーディングウィザード](../start/wizard.md)を参照してください。

## フローの詳細 (ローカルモード)

### ステップ 1: 既存設定の検出

-   `~/.openclaw/openclaw.json` が存在する場合、**保持 / 変更 / リセット** を選択します。
-   ウィザードを再実行しても、明示的に **リセット** を選択（または `--reset` を渡す）しない限り、何も削除されません。
-   CLI の `--reset` はデフォルトで `config+creds+sessions` です。ワークスペースも削除するには `--reset-scope full` を使用します。
-   設定が無効またはレガシーキーを含む場合、ウィザードは停止し、続行前に `openclaw doctor` を実行するよう求めます。
-   リセットは `trash` を使用し（決して `rm` ではありません）、スコープを提供します：
    -   設定のみ
    -   設定 + 認証情報 + セッション
    -   完全リセット（ワークスペースも削除）

### ステップ 2: モデル/認証

-   **Anthropic API キー**: `ANTHROPIC_API_KEY` が存在する場合はそれを使用し、なければキーの入力を求め、デーモン用に保存します。
-   **Anthropic OAuth (Claude Code CLI)**: macOSでは、ウィザードはKeychainアイテム「Claude Code-credentials」をチェックします（「常に許可」を選択してlaunchd起動がブロックされないようにします）。Linux/Windowsでは、`~/.claude/.credentials.json` が存在する場合はそれを再利用します。
-   **Anthropic トークン (setup-tokenを貼り付け)**: 任意のマシンで `claude setup-token` を実行し、トークンを貼り付けます（名前を付けられます。空白 = デフォルト）。
-   **OpenAI Code (Codex) サブスクリプション (Codex CLI)**: `~/.codex/auth.json` が存在する場合、ウィザードはそれを再利用できます。
-   **OpenAI Code (Codex) サブスクリプション (OAuth)**: ブラウザーフロー。`code#state` を貼り付けます。
    -   モデルが未設定または `openai/*` の場合、`agents.defaults.model` を `openai-codex/gpt-5.2` に設定します。
-   **OpenAI API キー**: `OPENAI_API_KEY` が存在する場合はそれを使用し、なければキーの入力を求め、認証プロファイルに保存します。
-   **xAI (Grok) API キー**: `XAI_API_KEY` の入力を求め、モデルプロバイダーとしてxAIを設定します。
-   **OpenCode Zen (マルチモデルプロキシ)**: `OPENCODE_API_KEY`（または `OPENCODE_ZEN_API_KEY`、[https://opencode.ai/auth](https://opencode.ai/auth) で取得）の入力を求めます。
-   **API キー**: キーを保存します。
-   **Vercel AI Gateway (マルチモデルプロキシ)**: `AI_GATEWAY_API_KEY` の入力を求めます。
-   詳細: [Vercel AI Gateway](../providers/vercel-ai-gateway.md)
-   **Cloudflare AI Gateway**: アカウントID、ゲートウェイID、`CLOUDFLARE_AI_GATEWAY_API_KEY` の入力を求めます。
-   詳細: [Cloudflare AI Gateway](../providers/cloudflare-ai-gateway.md)
-   **MiniMax M2.5**: 設定は自動書き込みされます。
-   詳細: [MiniMax](../providers/minimax.md)
-   **Synthetic (Anthropic互換)**: `SYNTHETIC_API_KEY` の入力を求めます。
-   詳細: [Synthetic](../providers/synthetic.md)
-   **Moonshot (Kimi K2)**: 設定は自動書き込みされます。
-   **Kimi Coding**: 設定は自動書き込みされます。
-   詳細: [Moonshot AI (Kimi + Kimi Coding)](../providers/moonshot.md)
-   **スキップ**: 認証はまだ設定されません。
-   検出されたオプションからデフォルトモデルを選択します（またはプロバイダー/モデルを手動で入力します）。最高の品質と低いプロンプトインジェクションリスクのため、プロバイダースタックで利用可能な最新世代の最強モデルを選択してください。
-   ウィザードはモデルチェックを実行し、設定されたモデルが不明または認証が不足している場合に警告します。
-   APIキー保存モードはデフォルトでプレーンテキストの認証プロファイル値です。代わりに環境変数参照を保存するには `--secret-input-mode ref` を使用します（例: `keyRef: { source: "env", provider: "default", id: "OPENAI_API_KEY" }`）。
-   OAuth認証情報は `~/.openclaw/credentials/oauth.json` に保存されます。認証プロファイルは `~/.openclaw/agents//agent/auth-profiles.json` に保存されます（APIキー + OAuth）。
-   詳細: [/concepts/oauth](../concepts/oauth.md)

> **ℹ️** ヘッドレス/サーバーのヒント: ブラウザーがあるマシンでOAuthを完了し、`~/.openclaw/credentials/oauth.json`（または `$OPENCLAW_STATE_DIR/credentials/oauth.json`）をゲートウェイホストにコピーします。

### ステップ 3: ワークスペース

-   デフォルト `~/.openclaw/workspace`（設定可能）。
-   エージェントのブートストラップ儀式に必要なワークスペースファイルを準備します。
-   完全なワークスペースレイアウト + バックアップガイド: [エージェントワークスペース](../concepts/agent-workspace.md)

### ステップ 4: ゲートウェイ

-   ポート、バインド、認証モード、Tailscale公開。
-   認証推奨: ローカルWSクライアントも認証が必要になるように、ループバックでも **トークン** を維持してください。
-   トークンモードでは、インタラクティブオンボーディングで以下を提供します：
    -   **プレーンテキストトークンを生成/保存**（デフォルト）
    -   **SecretRefを使用**（オプトイン）
    -   クイックスタートは、オンボーディングプローブ/ダッシュボードブートストラップのために、`env`、`file`、`exec` プロバイダー間で既存の `gateway.auth.token` SecretRefを再利用します。
    -   そのSecretRefが設定されているが解決できない場合、オンボーディングはランタイム認証が暗黙的に低下するのではなく、明確な修正メッセージで早期に失敗します。
-   パスワードモードでは、インタラクティブオンボーディングもプレーンテキストまたはSecretRef保存をサポートします。
-   非インタラクティブトークンSecretRefパス: `--gateway-token-ref-env <ENV_VAR>`。
    -   オンボーディングプロセス環境に空でない環境変数が必要です。
    -   `--gateway-token` と組み合わせることはできません。
-   認証を無効にするのは、すべてのローカルプロセスを完全に信頼する場合のみです。
-   非ループバックバインドでも認証が必要です。

### ステップ 5: チャネル

-   [WhatsApp](../channels/whatsapp.md): オプションのQRログイン。
-   [Telegram](../channels/telegram.md): ボットトークン。
-   [Discord](../channels/discord.md): ボットトークン。
-   [Google Chat](../channels/googlechat.md): サービスアカウントJSON + ウェブフックオーディエンス。
-   [Mattermost](../channels/mattermost.md) (プラグイン): ボットトークン + ベースURL。
-   [Signal](../channels/signal.md): オプションの `signal-cli` インストール + アカウント設定。
-   [BlueBubbles](../channels/bluebubbles.md): **iMessageに推奨**。サーバーURL + パスワード + ウェブフック。
-   [iMessage](../channels/imessage.md): レガシー `imsg` CLIパス + DBアクセス。
-   DMセキュリティ: デフォルトはペアリングです。最初のDMはコードを送信します。`openclaw pairing approve  ` で承認するか、許可リストを使用します。

### ステップ 6: Web検索

-   プロバイダーを選択: Perplexity、Brave、Gemini、Grok、またはKimi（またはスキップ）。
-   APIキーを貼り付けます（QuickStartは環境変数または既存設定からキーを自動検出します）。
-   `--skip-search` でスキップします。
-   後で設定: `openclaw configure --section web`。

### ステップ 7: デーモンインストール

-   macOS: LaunchAgent
    -   ログイン済みユーザーセッションが必要です。ヘッドレスの場合は、カスタムLaunchDaemonを使用します（同梱されていません）。
-   Linux（およびWSL2経由のWindows）: systemdユーザーユニット
    -   ウィザードは `loginctl enable-linger ` 経由でリンガリングを有効にしようとし、ログアウト後もゲートウェイが稼働し続けるようにします。
    -   sudoを要求する場合があります（`/var/lib/systemd/linger` に書き込み）。最初にsudoなしで試行します。
-   **ランタイム選択:** Node（推奨。WhatsApp/Telegramに必須）。Bunは**推奨されません**。
-   トークン認証にトークンが必要で、`gateway.auth.token` がSecretRef管理されている場合、デーモンインストールはそれを検証しますが、解決されたプレーンテキストトークン値をスーパーバイザーサービスの環境メタデータに永続化しません。
-   トークン認証にトークンが必要で、設定されたトークンSecretRefが未解決の場合、デーモンインストールは実行可能なガイダンスとともにブロックされます。
-   `gateway.auth.token` と `gateway.auth.password` の両方が設定され、`gateway.auth.mode` が未設定の場合、モードが明示的に設定されるまでデーモンインストールはブロックされます。

### ステップ 8: ヘルスチェック

-   ゲートウェイを起動し（必要なら）、`openclaw health` を実行します。
-   ヒント: `openclaw status --deep` はステータス出力にゲートウェイヘルスプローブを追加します（到達可能なゲートウェイが必要）。

### ステップ 9: スキル（推奨）

-   利用可能なスキルを読み取り、要件をチェックします。
-   ノードマネージャーを選択できます: **npm / pnpm**（bunは推奨されません）。
-   オプションの依存関係をインストールします（一部はmacOSでHomebrewを使用）。

### ステップ 10: 完了

-   概要 + 次のステップ。追加機能のためのiOS/Android/macOSアプリを含みます。

 

> **ℹ️** GUIが検出されない場合、ウィザードはブラウザーを開く代わりに、Control UI用のSSHポートフォワード手順を表示します。Control UIアセットが欠落している場合、ウィザードはそれらをビルドしようとします。フォールバックは `pnpm ui:build` です（UI依存関係を自動インストールします）。

## 非インタラクティブモード

`--non-interactive` を使用してオンボーディングを自動化またはスクリプト化します：

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

機械可読な概要には `--json` を追加します。非インタラクティブモードでのゲートウェイトークンSecretRef：

```bash
export OPENCLAW_GATEWAY_TOKEN="your-token"
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice skip \
  --gateway-auth token \
  --gateway-token-ref-env OPENCLAW_GATEWAY_TOKEN
```

`--gateway-token` と `--gateway-token-ref-env` は相互排他です。

> **ℹ️** `--json` は**非インタラクティブモードを意味しません**。スクリプトには `--non-interactive`（および `--workspace`）を使用してください。

 

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice gemini-api-key \
  --gemini-api-key "$GEMINI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice zai-api-key \
  --zai-api-key "$ZAI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice cloudflare-ai-gateway-api-key \
  --cloudflare-ai-gateway-account-id "your-account-id" \
  --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
  --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice moonshot-api-key \
  --moonshot-api-key "$MOONSHOT_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice synthetic-api-key \
  --synthetic-api-key "$SYNTHETIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice opencode-zen \
  --opencode-zen-api-key "$OPENCODE_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

### エージェントの追加（非インタラクティブ）

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

## ゲートウェイウィザード RPC

ゲートウェイはウィザードフローをRPC（`wizard.start`、`wizard.next`、`wizard.cancel`、`wizard.status`）で公開します。クライアント（macOSアプリ、Control UI）は、オンボーディングロジックを再実装することなくステップをレンダリングできます。

## Signal セットアップ (signal-cli)

ウィザードはGitHubリリースから `signal-cli` をインストールできます：

-   適切なリリースアセットをダウンロードします。
-   `~/.openclaw/tools/signal-cli//` 以下に保存します。
-   設定に `channels.signal.cliPath` を書き込みます。

注意：

-   JVMビルドには **Java 21** が必要です。
-   ネイティブビルドは利用可能な場合に使用されます。
-   WindowsはWSL2を使用します。signal-cliインストールはWSL内でLinuxフローに従います。

## ウィザードが書き込む内容

`~/.openclaw/openclaw.json` の典型的なフィールド：

-   `agents.defaults.workspace`
-   `agents.defaults.model` / `models.providers`（MiniMax選択時）
-   `tools.profile`（ローカルオンボーディングは未設定時にデフォルトで `"coding"`。既存の明示的な値は保持されます）
-   `gateway.*`（モード、バインド、認証、tailscale）
-   `session.dmScope`（動作詳細: [CLIオンボーディングリファレンス](../start/wizard-cli-reference.md#outputs-and-internals)）
-   `channels.telegram.botToken`、`channels.discord.token`、`channels.signal.*`、`channels.imessage.*`
-   プロンプト中にオプトインした場合のチャネル許可リスト（Slack/Discord/Matrix/Microsoft Teams）（名前は可能な限りIDに解決されます）。
-   `skills.install.nodeManager`
-   `wizard.lastRunAt`
-   `wizard.lastRunVersion`
-   `wizard.lastRunCommit`
-   `wizard.lastRunCommand`
-   `wizard.lastRunMode`

`openclaw agents add` は `agents.list[]` とオプションの `bindings` を書き込みます。WhatsApp認証情報は `~/.openclaw/credentials/whatsapp//` 以下に保存されます。セッションは `~/.openclaw/agents//sessions/` 以下に保存されます。一部のチャネルはプラグインとして提供されます。オンボーディング中に選択すると、ウィザードは設定前にそれをインストールするよう促します（npmまたはローカルパス）。

## 関連ドキュメント

-   ウィザード概要: [オンボーディングウィザード](../start/wizard.md)
-   macOSアプリオンボーディング: [オンボーディング](../start/onboarding.md)
-   設定リファレンス: [ゲートウェイ設定](../gateway/configuration.md)
-   プロバイダー: [WhatsApp](../channels/whatsapp.md)、[Telegram](../channels/telegram.md)、[Discord](../channels/discord.md)、[Google Chat](../channels/googlechat.md)、[Signal](../channels/signal.md)、[BlueBubbles](../channels/bluebubbles.md) (iMessage)、[iMessage](../channels/imessage.md) (レガシー)
-   スキル: [スキル](../tools/skills.md)、[スキル設定](../tools/skills-config.md)

[USER](./templates/USER.md)[トークン使用とコスト](./token-use.md)