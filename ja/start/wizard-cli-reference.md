

  ガイド

  
# CLI オンボーディングリファレンス

このページは `openclaw onboard` の完全なリファレンスです。短いガイドについては、[オンボーディングウィザード (CLI)](./wizard.md) を参照してください。

## ウィザードの機能

ローカルモード（デフォルト）では、以下の手順を案内します：

-   モデルと認証の設定（OpenAI Code サブスクリプション OAuth、Anthropic API キーまたはセットアップトークン、さらに MiniMax、GLM、Moonshot、AI Gateway オプション）
-   ワークスペースの場所とブートストラップファイル
-   ゲートウェイ設定（ポート、バインド、認証、Tailscale）
-   チャネルとプロバイダー（Telegram、WhatsApp、Discord、Google Chat、Mattermost プラグイン、Signal）
-   デーモンのインストール（LaunchAgent または systemd ユーザーユニット）
-   ヘルスチェック
-   スキルのセットアップ

リモートモードでは、このマシンを他の場所にあるゲートウェイに接続するように設定します。リモートホスト上には何もインストールまたは変更しません。

## ローカルフローの詳細

### ステップ 1: 既存設定の検出

-   `~/.openclaw/openclaw.json` が存在する場合、保持、変更、またはリセットを選択します。
-   ウィザードを再実行しても、明示的にリセットを選択（または `--reset` を渡す）しない限り、何も消去されません。
-   CLI の `--reset` はデフォルトで `config+creds+sessions` です。ワークスペースも削除するには `--reset-scope full` を使用します。
-   設定が無効またはレガシーキーを含む場合、ウィザードは停止し、続行前に `openclaw doctor` を実行するよう求めます。
-   リセットは `trash` を使用し、以下のスコープを提供します：
    -   設定のみ
    -   設定 + 認証情報 + セッション
    -   完全リセット（ワークスペースも削除）

### ステップ 2: モデルと認証

-   完全なオプション一覧は [認証とモデルオプション](#auth-and-model-options) にあります。

### ステップ 3: ワークスペース

-   デフォルトは `~/.openclaw/workspace`（設定可能）。
-   初回実行のブートストラップ儀式に必要なワークスペースファイルをシードします。
-   ワークスペースレイアウト: [エージェントワークスペース](../concepts/agent-workspace.md)。

### ステップ 4: ゲートウェイ

-   ポート、バインド、認証モード、Tailscale 公開についてプロンプトを表示します。
-   推奨：ローカル WS クライアントが認証する必要があるように、ループバックでもトークン認証を有効にしたままにします。
-   トークンモードでは、インタラクティブオンボーディングで以下を提供します：
    -   **平文トークンを生成/保存**（デフォルト）
    -   **SecretRef を使用**（オプトイン）
-   パスワードモードでは、インタラクティブオンボーディングも平文または SecretRef ストレージをサポートします。
-   非インタラクティブトークン SecretRef パス: `--gateway-token-ref-env <ENV_VAR>`。
    -   オンボーディングプロセス環境内で空でない環境変数が必要です。
    -   `--gateway-token` と組み合わせることはできません。
-   認証を無効にするのは、すべてのローカルプロセスを完全に信頼する場合のみです。
-   非ループバックバインドでも認証が必要です。

### ステップ 5: チャネル

-   [WhatsApp](../channels/whatsapp.md): オプションの QR ログイン
-   [Telegram](../channels/telegram.md): ボットトークン
-   [Discord](../channels/discord.md): ボットトークン
-   [Google Chat](../channels/googlechat.md): サービスアカウント JSON + ウェブフックオーディエンス
-   [Mattermost](../channels/mattermost.md) プラグイン: ボットトークン + ベース URL
-   [Signal](../channels/signal.md): オプションの `signal-cli` インストール + アカウント設定
-   [BlueBubbles](../channels/bluebubbles.md): iMessage に推奨。サーバー URL + パスワード + ウェブフック
-   [iMessage](../channels/imessage.md): レガシー `imsg` CLI パス + DB アクセス
-   DM セキュリティ: デフォルトはペアリングです。最初の DM でコードが送信されます。`openclaw pairing approve  ` で承認するか、許可リストを使用します。

### ステップ 6: デーモンのインストール

-   macOS: LaunchAgent
    -   ログイン済みユーザーセッションが必要です。ヘッドレスの場合は、カスタム LaunchDaemon（同梱されていません）を使用します。
-   Linux および WSL2 経由の Windows: systemd ユーザーユニット
    -   ウィザードは `loginctl enable-linger ` を試行し、ログアウト後もゲートウェイが稼働し続けるようにします。
    -   sudo を要求する場合があります（`/var/lib/systemd/linger` に書き込み）。最初に sudo なしで試行します。
-   ランタイム選択: Node（推奨。WhatsApp と Telegram に必須）。Bun は推奨されません。

### ステップ 7: ヘルスチェック

-   ゲートウェイを起動し（必要に応じて）、`openclaw health` を実行します。
-   `openclaw status --deep` は、ステータス出力にゲートウェイヘルスプローブを追加します。

### ステップ 8: スキル

-   利用可能なスキルを読み取り、要件をチェックします。
-   ノードマネージャーを選択できます: npm または pnpm（bun は推奨されません）。
-   オプションの依存関係をインストールします（一部は macOS で Homebrew を使用します）。

### ステップ 9: 完了

-   要約と次のステップ（iOS、Android、macOS アプリオプションを含む）。

 

> **ℹ️** GUI が検出されない場合、ウィザードはブラウザを開く代わりに、Control UI 用の SSH ポートフォワード手順を表示します。Control UI アセットが欠落している場合、ウィザードはそれらをビルドしようとします。フォールバックは `pnpm ui:build` です（UI の依存関係を自動インストールします）。

## リモートモードの詳細

リモートモードでは、このマシンを他の場所にあるゲートウェイに接続するように設定します。

> **ℹ️** リモートモードは、リモートホスト上に何もインストールまたは変更しません。

 設定内容：

-   リモートゲートウェイ URL (`ws://...`)
-   リモートゲートウェイ認証が必要な場合のトークン（推奨）

> **ℹ️** -   ゲートウェイがループバックのみの場合、SSH トンネリングまたは tailnet を使用します。
> -   ディスカバリのヒント：
>     -   macOS: Bonjour (`dns-sd`)
>     -   Linux: Avahi (`avahi-browse`)

## 認証とモデルオプション

`ANTHROPIC_API_KEY` が存在する場合はそれを使用し、存在しない場合はキーの入力を求め、デーモンで使用するために保存します。

-   macOS: キーチェーン項目「Claude Code-credentials」をチェック
-   Linux および Windows: `~/.claude/.credentials.json` が存在する場合は再利用

macOS では、launchd の起動がブロックされないように「常に許可」を選択します。

任意のマシンで `claude setup-token` を実行し、トークンを貼り付けます。名前を付けることができます。空白の場合はデフォルトを使用します。

`~/.codex/auth.json` が存在する場合、ウィザードはそれを再利用できます。

ブラウザフロー。`code#state` を貼り付けます。モデルが未設定または `openai/*` の場合、`agents.defaults.model` を `openai-codex/gpt-5.4` に設定します。

`OPENAI_API_KEY` が存在する場合はそれを使用し、存在しない場合はキーの入力を求め、認証プロファイルに認証情報を保存します。モデルが未設定、`openai/*`、または `openai-codex/*` の場合、`agents.defaults.model` を `openai/gpt-5.1-codex` に設定します。

`XAI_API_KEY` の入力を求め、xAI をモデルプロバイダーとして設定します。

`OPENCODE_API_KEY`（または `OPENCODE_ZEN_API_KEY`）の入力を求めます。セットアップ URL: [opencode.ai/auth](https://opencode.ai/auth)。

キーを保存します。

`AI_GATEWAY_API_KEY` の入力を求めます。詳細: [Vercel AI Gateway](../providers/vercel-ai-gateway.md)。

アカウント ID、ゲートウェイ ID、`CLOUDFLARE_AI_GATEWAY_API_KEY` の入力を求めます。詳細: [Cloudflare AI Gateway](../providers/cloudflare-ai-gateway.md)。

設定は自動書き込みされます。詳細: [MiniMax](../providers/minimax.md)。

`SYNTHETIC_API_KEY` の入力を求めます。詳細: [Synthetic](../providers/synthetic.md)。

Moonshot (Kimi K2) と Kimi Coding の設定は自動書き込みされます。詳細: [Moonshot AI (Kimi + Kimi Coding)](../providers/moonshot.md)。

OpenAI 互換および Anthropic 互換エンドポイントで動作します。インタラクティブオンボーディングは、他のプロバイダー API キーフローと同じ API キーストレージ選択肢をサポートします：

-   **API キーを今貼り付ける**（平文）
-   **シークレット参照を使用**（環境変数参照または設定済みプロバイダー参照、事前検証付き）

非インタラクティブフラグ：

-   `--auth-choice custom-api-key`
-   `--custom-base-url`
-   `--custom-model-id`
-   `--custom-api-key`（オプション。`CUSTOM_API_KEY` にフォールバック）
-   `--custom-provider-id`（オプション）
-   `--custom-compatibility <openai|anthropic>`（オプション。デフォルト `openai`）

認証を設定せずにスキップします。

 モデルの動作：

-   検出されたオプションからデフォルトモデルを選択するか、プロバイダーとモデルを手動で入力します。
-   ウィザードはモデルチェックを実行し、設定されたモデルが不明または認証が欠落している場合に警告します。

認証情報とプロファイルのパス：

-   OAuth 認証情報: `~/.openclaw/credentials/oauth.json`
-   認証プロファイル（API キー + OAuth）: `~/.openclaw/agents//agent/auth-profiles.json`

認証情報ストレージモード：

-   デフォルトのオンボーディング動作は、API キーを認証プロファイル内の平文値として永続化します。
-   `--secret-input-mode ref` は、平文キーストレージの代わりに参照モードを有効にします。インタラクティブオンボーディングでは、以下を選択できます：
    -   環境変数参照（例：`keyRef: { source: "env", provider: "default", id: "OPENAI_API_KEY" }`）
    -   設定済みプロバイダー参照（`file` または `exec`）、プロバイダーエイリアス + ID 付き
-   インタラクティブ参照モードは、保存前に高速な事前検証を実行します。
    -   環境変数参照: 現在のオンボーディング環境での変数名 + 空でない値の検証。
    -   プロバイダー参照: プロバイダー設定の検証と要求された ID の解決。
    -   事前検証が失敗した場合、オンボーディングはエラーを表示し、再試行を許可します。
-   非インタラクティブモードでは、`--secret-input-mode ref` は環境変数バックのみです。
    -   プロバイダー環境変数をオンボーディングプロセス環境に設定します。
    -   インラインキーフラグ（例：`--openai-api-key`）は、その環境変数が設定されていることを必要とします。そうでない場合、オンボーディングは高速に失敗します。
    -   カスタムプロバイダーの場合、非インタラクティブ `ref` モードは `models.providers..apiKey` を `{ source: "env", provider: "default", id: "CUSTOM_API_KEY" }` として保存します。
    -   そのカスタムプロバイダーの場合、`--custom-api-key` は `CUSTOM_API_KEY` が設定されていることを必要とします。そうでない場合、オンボーディングは高速に失敗します。
-   ゲートウェイ認証情報は、インタラクティブオンボーディングで平文と SecretRef の選択をサポートします：
    -   トークンモード: **平文トークンを生成/保存**（デフォルト）または **SecretRef を使用**。
    -   パスワードモード: 平文または SecretRef。
-   非インタラクティブトークン SecretRef パス: `--gateway-token-ref-env <ENV_VAR>`。
-   既存の平文セットアップは変更なく引き続き動作します。

> **ℹ️** ヘッドレスおよびサーバーのヒント: ブラウザがあるマシンで OAuth を完了し、`~/.openclaw/credentials/oauth.json`（または `$OPENCLAW_STATE_DIR/credentials/oauth.json`）をゲートウェイホストにコピーします。

## 出力と内部構造

`~/.openclaw/openclaw.json` の典型的なフィールド：

-   `agents.defaults.workspace`
-   `agents.defaults.model` / `models.providers`（MiniMax が選択された場合）
-   `tools.profile`（ローカルオンボーディングは、未設定時にデフォルトで `"coding"` に設定。既存の明示的な値は保持されます）
-   `gateway.*`（モード、バインド、認証、Tailscale）
-   `session.dmScope`（ローカルオンボーディングは、未設定時にデフォルトでこれを `per-channel-peer` に設定。既存の明示的な値は保持されます）
-   `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
-   プロンプト中にオプトインした場合のチャネル許可リスト（Slack、Discord、Matrix、Microsoft Teams）（可能な場合は名前が ID に解決されます）
-   `skills.install.nodeManager`
-   `wizard.lastRunAt`
-   `wizard.lastRunVersion`
-   `wizard.lastRunCommit`
-   `wizard.lastRunCommand`
-   `wizard.lastRunMode`

`openclaw agents add` は `agents.list[]` とオプションの `bindings` を書き込みます。WhatsApp 認証情報は `~/.openclaw/credentials/whatsapp//` の下に保存されます。セッションは `~/.openclaw/agents//sessions/` の下に保存されます。

> **ℹ️** 一部のチャネルはプラグインとして提供されます。オンボーディング中に選択されると、ウィザードはチャネル設定前にプラグイン（npm またはローカルパス）のインストールを促します。

 ゲートウェイウィザード RPC：

-   `wizard.start`
-   `wizard.next`
-   `wizard.cancel`
-   `wizard.status`

クライアント（macOS アプリと Control UI）は、オンボーディングロジックを再実装せずにステップをレンダリングできます。Signal セットアップの動作：

-   適切なリリースアセットをダウンロード
-   `~/.openclaw/tools/signal-cli//` の下に保存
-   設定内に `channels.signal.cliPath` を書き込み
-   JVM ビルドには Java 21 が必要
-   ネイティブビルドは利用可能な場合に使用
-   Windows は WSL2 を使用し、WSL 内で Linux signal-cli フローに従う

## 関連ドキュメント

-   オンボーディングハブ: [オンボーディングウィザード (CLI)](./wizard.md)
-   自動化とスクリプト: [CLI 自動化](./wizard-cli-automation.md)
-   コマンドリファレンス: [`openclaw onboard`](../cli/onboard.md)

[個人用アシスタントセットアップ](./openclaw.md)[CLI 自動化](./wizard-cli-automation.md)