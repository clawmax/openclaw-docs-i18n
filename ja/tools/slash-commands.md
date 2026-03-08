

  スキル

  
# スラッシュコマンド

コマンドはゲートウェイによって処理されます。ほとんどのコマンドは、`/`で始まる**単独の**メッセージとして送信する必要があります。ホスト専用のbashチャットコマンドは `! ` を使用します (`/bash ` はエイリアスです)。関連するシステムは2つあります:

-   **コマンド**: 単独の `/...` メッセージ。
-   **ディレクティブ**: `/think`, `/verbose`, `/reasoning`, `/elevated`, `/exec`, `/model`, `/queue`。
    -   ディレクティブは、モデルがメッセージを見る前にメッセージから取り除かれます。
    -   通常のチャットメッセージ（ディレクティブのみでない場合）では、これらは「インラインヒント」として扱われ、セッション設定を**永続化しません**。
    -   ディレクティブのみのメッセージ（メッセージがディレクティブのみを含む場合）では、セッションに永続化され、確認応答で返信します。
    -   ディレクティブは**認可された送信者**に対してのみ適用されます。`commands.allowFrom` が設定されている場合、それが唯一の許可リストとして使用されます。そうでない場合、認可はチャネルの許可リスト/ペアリングと `commands.useAccessGroups` から来ます。認可されていない送信者には、ディレクティブはプレーンテキストとして扱われます。

また、いくつかの**インラインショートカット**（許可リスト/認可された送信者のみ）があります: `/help`, `/commands`, `/status`, `/whoami` (`/id`)。これらは即座に実行され、モデルがメッセージを見る前に取り除かれ、残りのテキストは通常のフローを続けます。

## 設定

```json
{
  commands: {
    native: "auto",
    nativeSkills: "auto",
    text: true,
    bash: false,
    bashForegroundMs: 2000,
    config: false,
    debug: false,
    restart: false,
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

-   `commands.text` (デフォルト `true`) は、チャットメッセージ内の `/...` の解析を有効にします。
    -   ネイティブコマンドがないサービス（WhatsApp/WebChat/Signal/iMessage/Google Chat/MS Teams）では、これを `false` に設定してもテキストコマンドは機能します。
-   `commands.native` (デフォルト `"auto"`) はネイティブコマンドを登録します。
    -   Auto: Discord/Telegramではオン、Slackではオフ（スラッシュコマンドを追加するまで）。ネイティブサポートのないプロバイダでは無視されます。
    -   プロバイダごとに `channels.discord.commands.native`、`channels.telegram.commands.native`、または `channels.slack.commands.native` を設定して上書きできます（bool または `"auto"`）。
    -   `false` は、起動時にDiscord/Telegramで以前に登録されたコマンドをクリアします。SlackコマンドはSlackアプリで管理され、自動的には削除されません。
-   `commands.nativeSkills` (デフォルト `"auto"`) は、サポートされている場合に**スキル**コマンドをネイティブに登録します。
    -   Auto: Discord/Telegramではオン、Slackではオフ（Slackはスキルごとにスラッシュコマンドを作成する必要があります）。
    -   プロバイダごとに `channels.discord.commands.nativeSkills`、`channels.telegram.commands.nativeSkills`、または `channels.slack.commands.nativeSkills` を設定して上書きできます（bool または `"auto"`）。
-   `commands.bash` (デフォルト `false`) は、ホストシェルコマンドを実行する `! ` を有効にします (`/bash ` はエイリアス; `tools.elevated` 許可リストが必要)。
-   `commands.bashForegroundMs` (デフォルト `2000`) は、bashがバックグラウンドモードに切り替わる前に待機する時間を制御します (`0` は即座にバックグラウンド化)。
-   `commands.config` (デフォルト `false`) は `/config` を有効にします (`openclaw.json` の読み書き)。
-   `commands.debug` (デフォルト `false`) は `/debug` を有効にします（ランタイムのみの上書き）。
-   `commands.allowFrom` (オプション) は、コマンド認可のためのプロバイダごとの許可リストを設定します。設定されている場合、コマンドとディレクティブの唯一の認可ソースとなります（チャネルの許可リスト/ペアリングと `commands.useAccessGroups` は無視されます）。グローバルなデフォルトには `"*"` を使用します。プロバイダ固有のキーはそれを上書きします。
-   `commands.useAccessGroups` (デフォルト `true`) は、`commands.allowFrom` が設定されていない場合に、コマンドに対して許可リスト/ポリシーを強制します。

## コマンド一覧

テキスト + ネイティブ（有効な場合）:

-   `/help`
-   `/commands`
-   `/skill  [input]` (名前でスキルを実行)
-   `/status` (現在のステータスを表示; 利用可能な場合、現在のモデルプロバイダの使用量/クォータを含む)
-   `/allowlist` (許可リストエントリの一覧表示/追加/削除)
-   `/approve  allow-once|allow-always|deny` (exec承認プロンプトを解決)
-   `/context [list|detail|json]` ("コンテキスト"を説明; `detail` はファイルごと + ツールごと + スキルごと + システムプロンプトサイズを表示)
-   `/export-session [path]` (エイリアス: `/export`) (現在のセッションを完全なシステムプロンプト付きでHTMLにエクスポート)
-   `/whoami` (あなたの送信者IDを表示; エイリアス: `/id`)
-   `/session idle <duration|off>` (フォーカスされたスレッドバインディングの非アクティブ自動アンフォーカスを管理)
-   `/session max-age <duration|off>` (フォーカスされたスレッドバインディングのハード最大経過時間自動アンフォーカスを管理)
-   `/subagents list|kill|log|info|send|steer|spawn` (現在のセッションのサブエージェント実行を検査、制御、またはスポーン)
-   `/acp spawn|cancel|steer|close|status|set-mode|set|cwd|permissions|timeout|model|reset-options|doctor|install|sessions` (ACPランタイムセッションを検査および制御)
-   `/agents` (このセッションのスレッドにバインドされたエージェントを一覧表示)
-   `/focus ` (Discord: このスレッド、または新しいスレッドをセッション/サブエージェントターゲットにバインド)
-   `/unfocus` (Discord: 現在のスレッドバインディングを削除)
-   `/kill <id|#|all>` (このセッションの実行中のサブエージェントを1つまたはすべて直ちに中止; 確認メッセージなし)
-   `/steer <id|#> ` (実行中のサブエージェントを即座に操縦: 可能な場合は実行中に、そうでない場合は現在の作業を中止し、操縦メッセージで再起動)
-   `/tell <id|#> ` (`/steer` のエイリアス)
-   `/config show|get|set|unset` (設定をディスクに永続化、所有者専用; `commands.config: true` が必要)
-   `/debug show|set|unset|reset` (ランタイム上書き、所有者専用; `commands.debug: true` が必要)
-   `/usage off|tokens|full|cost` (応答ごとの使用量フッターまたはローカルコスト概要)
-   `/tts off|always|inbound|tagged|status|provider|limit|summary|audio` (TTSを制御; [/tts](../tts.md) を参照)
    -   Discord: ネイティブコマンドは `/voice` (Discordは `/tts` を予約); テキスト `/tts` は引き続き機能します。
-   `/stop`
-   `/restart`
-   `/dock-telegram` (エイリアス: `/dock_telegram`) (返信をTelegramに切り替え)
-   `/dock-discord` (エイリアス: `/dock_discord`) (返信をDiscordに切り替え)
-   `/dock-slack` (エイリアス: `/dock_slack`) (返信をSlackに切り替え)
-   `/activation mention|always` (グループのみ)
-   `/send on|off|inherit` (所有者専用)
-   `/reset` または `/new [model]` (オプションのモデルヒント; 残りは通過)
-   `/think <off|minimal|low|medium|high|xhigh>` (モデル/プロバイダによる動的選択; エイリアス: `/thinking`, `/t`)
-   `/verbose on|full|off` (エイリアス: `/v`)
-   `/reasoning on|off|stream` (エイリアス: `/reason`; オンの場合、`Reasoning:` で始まる別のメッセージを送信; `stream` = Telegramドラフトのみ)
-   `/elevated on|off|ask|full` (エイリアス: `/elev`; `full` はexec承認をスキップ)
-   `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=` (`/exec` を送信して現在の設定を表示)
-   `/model ` (エイリアス: `/models`; または `agents.defaults.models.*.alias` からの `/エイリアス`)
-   `/queue ` (プラス `debounce:2s cap:25 drop:summarize` のようなオプション; `/queue` を送信して現在の設定を確認)
-   `/bash ` (ホスト専用; `! ` のエイリアス; `commands.bash: true` + `tools.elevated` 許可リストが必要)

テキストのみ:

-   `/compact [instructions]` ([/concepts/compaction](../concepts/compaction.md) を参照)
-   `! ` (ホスト専用; 一度に1つ; 長時間実行ジョブには `!poll` + `!stop` を使用)
-   `!poll` (出力 / ステータスを確認; オプションの `sessionId` を受け入れる; `/bash poll` も機能)
-   `!stop` (実行中のbashジョブを停止; オプションの `sessionId` を受け入れる; `/bash stop` も機能)

注記:

-   コマンドは、コマンドと引数の間にオプションの `:` を受け入れます (例: `/think: high`, `/send: on`, `/help:`)。
-   `/new ` は、モデルエイリアス、`provider/model`、またはプロバイダ名（あいまい一致）を受け入れます。一致しない場合、テキストはメッセージ本文として扱われます。
-   プロバイダごとの完全な使用量内訳については、`openclaw status --usage` を使用してください。
-   `/allowlist add|remove` は `commands.config=true` を必要とし、チャネルの `configWrites` を尊重します。
-   `/usage` は応答ごとの使用量フッターを制御します; `/usage cost` はOpenClawセッションログからのローカルコスト概要を表示します。
-   `/restart` はデフォルトで有効です; `commands.restart: false` を設定して無効にできます。
-   Discord専用ネイティブコマンド: `/vc join|leave|status` はボイスチャネルを制御します (`channels.discord.voice` とネイティブコマンドが必要; テキストでは利用不可)。
-   Discordスレッドバインディングコマンド (`/focus`, `/unfocus`, `/agents`, `/session idle`, `/session max-age`) は、効果的なスレッドバインディングが有効になっている必要があります (`session.threadBindings.enabled` および/または `channels.discord.threadBindings.enabled`)。
-   ACPコマンドリファレンスとランタイム動作: [ACPエージェント](./acp-agents.md)。
-   `/verbose` はデバッグと追加の可視性を目的としています; 通常使用では**オフ**にしておいてください。
-   ツール失敗の概要は関連する場合に表示されますが、詳細な失敗テキストは `/verbose` が `on` または `full` の場合にのみ含まれます。
-   `/reasoning` (および `/verbose`) はグループ設定では危険です: 意図していない内部推論やツール出力を公開する可能性があります。特にグループチャットでは、オフのままにすることをお勧めします。
-   **高速パス:** 許可リスト送信者からのコマンドのみのメッセージは即座に処理されます（キュー + モデルをバイパス）。
-   **グループメンションゲーティング:** 許可リスト送信者からのコマンドのみのメッセージは、メンション要件をバイパスします。
-   **インラインショートカット（許可リスト送信者のみ）:** 特定のコマンドは、通常のメッセージに埋め込まれている場合にも機能し、モデルが残りのテキストを見る前に取り除かれます。
    -   例: `hey /status` はステータス返信をトリガーし、残りのテキストは通常のフローを続けます。
-   現在: `/help`, `/commands`, `/status`, `/whoami` (`/id`)。
-   認可されていないコマンドのみのメッセージは黙って無視され、インライン `/...` トークンはプレーンテキストとして扱われます。
-   **スキルコマンド:** `user-invocable` スキルはスラッシュコマンドとして公開されます。名前は `a-z0-9_` にサニタイズされます（最大32文字）。衝突がある場合は数値サフィックスが付きます（例: `_2`）。
    -   `/skill  [input]` は名前でスキルを実行します（ネイティブコマンドの制限によりスキルごとのコマンドができない場合に便利）。
    -   デフォルトでは、スキルコマンドは通常のリクエストとしてモデルに転送されます。
    -   スキルはオプションで `command-dispatch: tool` を宣言して、コマンドを直接ツールにルーティングできます（決定論的、モデルなし）。
    -   例: `/prose` (OpenProseプラグイン) — [OpenProse](../prose.md) を参照。
-   **ネイティブコマンド引数:** Discordは動的オプションにオートコンプリートを使用します（および必須引数を省略した場合にボタンメニュー）。TelegramとSlackは、コマンドが選択肢をサポートし、引数を省略した場合にボタンメニューを表示します。

## 使用状況表示（どこに何が表示されるか）

-   **プロバイダ使用量/クォータ** (例: "Claude 80% left") は、使用量追跡が有効な場合、現在のモデルプロバイダの `/status` に表示されます。
-   **応答ごとのトークン数/コスト** は `/usage off|tokens|full` で制御されます（通常の返信に追加）。
-   `/model status` は**モデル/認証/エンドポイント**に関するものであり、使用量ではありません。

## モデル選択 (/model)

`/model` はディレクティブとして実装されています。例:

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model opus@anthropic:default
/model status
```

注記:

-   `/model` と `/model list` は、コンパクトな番号付きピッカー（モデルファミリー + 利用可能なプロバイダ）を表示します。
-   Discordでは、`/model` と `/models` は、プロバイダとモデルのドロップダウンと送信ステップを含むインタラクティブなピッカーを開きます。
-   `/model <#>` はそのピッカーから選択します（可能な場合は現在のプロバイダを優先）。
-   `/model status` は、設定されたプロバイダエンドポイント (`baseUrl`) とAPIモード (`api`) を含む詳細ビューを表示します（利用可能な場合）。

## デバッグ上書き

`/debug` を使用すると、**ランタイムのみ**の設定上書き（メモリ、ディスクではない）を設定できます。所有者専用。デフォルトでは無効; `commands.debug: true` で有効化。例:

```bash
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug set channels.whatsapp.allowFrom=["+1555","+4477"]
/debug unset messages.responsePrefix
/debug reset
```

注記:

-   上書きは新しい設定読み取りに即座に適用されますが、`openclaw.json` には書き込まれ**ません**。
-   すべての上書きをクリアしてディスク上の設定に戻すには `/debug reset` を使用します。

## 設定更新

`/config` はディスク上の設定 (`openclaw.json`) に書き込みます。所有者専用。デフォルトでは無効; `commands.config: true` で有効化。例:

```bash
/config show
/config show messages.responsePrefix
/config get messages.responsePrefix
/config set messages.responsePrefix="[openclaw]"
/config unset messages.responsePrefix
```

注記:

-   設定は書き込み前に検証されます; 無効な変更は拒否されます。
-   `/config` の更新は再起動後も永続化されます。

## サービスごとの注意点

-   **テキストコマンド** は通常のチャットセッションで実行されます（DMは `main` を共有、グループは独自のセッションを持ちます）。
-   **ネイティブコマンド** は分離されたセッションを使用します:
    -   Discord: `agent::discord:slash:`
    -   Slack: `agent::slack:slash:` (接頭辞は `channels.slack.slashCommand.sessionPrefix` で設定可能)
    -   Telegram: `telegram:slash:` (`CommandTargetSessionKey` 経由でチャットセッションをターゲット)
-   **`/stop`** はアクティブなチャットセッションをターゲットするため、現在の実行を中止できます。
-   **Slack:** `channels.slack.slashCommand` は、単一の `/openclaw` スタイルのコマンドに対して引き続きサポートされています。`commands.native` を有効にする場合、組み込みコマンドごとに1つのSlackスラッシュコマンドを作成する必要があります（`/help` と同じ名前）。Slackのコマンド引数メニューは、エフェメラルなBlock Kitボタンとして配信されます。
    -   Slackネイティブ例外: Slackは `/status` を予約しているため、`/agentstatus` (not `/status`) を登録します。テキスト `/status` はSlackメッセージ内で引き続き機能します。

[スキルの作成](./creating-skills.md)[スキル](./skills.md)