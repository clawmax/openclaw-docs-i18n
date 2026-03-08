

  設定と運用

  
# Heartbeat

> **Heartbeat と Cron の違いは？** それぞれの使用タイミングについては、[Cron vs Heartbeat](../automation/cron-vs-heartbeat.md) を参照してください。

Heartbeat は、モデルが注意を要する事項を通知できるように、メインセッションで**定期的なエージェントターン**を実行します。これにより、ユーザーへのスパムを防ぎます。トラブルシューティング: [/automation/troubleshooting](../automation/troubleshooting.md)

## クイックスタート (初心者向け)

1.  Heartbeat を有効のままにします（デフォルトは `30m`、Anthropic OAuth/セットアップトークン使用時は `1h`）。または、独自の間隔を設定します。
2.  エージェントのワークスペースに小さな `HEARTBEAT.md` チェックリストを作成します（任意ですが推奨）。
3.  Heartbeat メッセージの送信先を決定します（デフォルトは `target: "none"`；最後の連絡先に送信するには `target: "last"` を設定します）。
4.  オプション: 透明性のために Heartbeat の推論配信を有効にします。
5.  オプション: Heartbeat の実行に `HEARTBEAT.md` のみが必要な場合は、軽量ブートストラップコンテキストを使用します。
6.  オプション: Heartbeat をアクティブ時間内（現地時間）に制限します。

設定例:

```json
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last", // 最後の連絡先への明示的な配信（デフォルトは "none"）
        directPolicy: "allow", // デフォルト: ダイレクト/DM ターゲットを許可；抑制するには "block" を設定
        lightContext: true, // オプション: ブートストラップファイルから HEARTBEAT.md のみを注入
        // activeHours: { start: "08:00", end: "24:00" },
        // includeReasoning: true, // オプション: 別途 `Reasoning:` メッセージも送信
      },
    },
  },
}
```

## デフォルト設定

-   間隔: `30m` (Anthropic OAuth/セットアップトークンが検出された認証モードの場合は `1h`)。`agents.defaults.heartbeat.every` またはエージェントごとの `agents.list[].heartbeat.every` で設定；`0m` で無効化。
-   プロンプト本文 (`agents.defaults.heartbeat.prompt` で設定可能): `HEARTBEAT.md が存在する場合は読みます（ワークスペースコンテキスト）。それに厳密に従います。以前のチャットからの古いタスクを推測したり繰り返したりしないでください。注意が必要なものがなければ、HEARTBEAT_OK と返信します。`
-   Heartbeat プロンプトは、ユーザーメッセージとして**そのまま**送信されます。システムプロンプトには「Heartbeat」セクションが含まれ、実行は内部的にフラグ付けされます。
-   アクティブ時間 (`heartbeat.activeHours`) は、設定されたタイムゾーンでチェックされます。ウィンドウ外では、ウィンドウ内の次の実行タイミングまで Heartbeat はスキップされます。

## Heartbeat プロンプトの目的

デフォルトのプロンプトは意図的に広範です:

-   **バックグラウンドタスク**: 「未完了のタスクを考慮する」という指示により、エージェントはフォローアップ（受信トレイ、カレンダー、リマインダー、キューに入った作業）を確認し、緊急の事項を通知します。
-   **人間へのチェックイン**: 「日中は時々あなたの人間をチェックする」という指示により、軽量な「何か必要なものはありますか？」というメッセージを時折促しますが、設定された現地タイムゾーンを使用して夜間のスパムを回避します（[/concepts/timezone](../concepts/timezone.md) を参照）。

Heartbeat に特定の動作（例：「Gmail PubSub の統計を確認する」や「ゲートウェイの健全性を確認する」）をさせたい場合は、`agents.defaults.heartbeat.prompt`（または `agents.list[].heartbeat.prompt`）をカスタム本文（そのまま送信）に設定します。

## 応答契約

-   注意が必要なものがなければ、**`HEARTBEAT_OK`** で返信します。
-   Heartbeat 実行中、OpenClaw は返信の**先頭または末尾**に `HEARTBEAT_OK` が現れる場合、それを ACK として扱います。トークンは削除され、残りの内容が **≤ `ackMaxChars`**（デフォルト: 300）の場合、返信は破棄されます。
-   返信の**途中**に `HEARTBEAT_OK` が現れる場合、特別扱いはされません。
-   アラートの場合、`HEARTBEAT_OK` を**含めないでください**；アラートテキストのみを返します。

Heartbeat 以外では、メッセージの先頭/末尾にある `HEARTBEAT_OK` は削除されログに記録されます；`HEARTBEAT_OK` のみのメッセージは破棄されます。

## 設定

```json
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m", // デフォルト: 30m (0m で無効化)
        model: "anthropic/claude-opus-4-6",
        includeReasoning: false, // デフォルト: false (利用可能な場合、別途 Reasoning: メッセージを配信)
        lightContext: false, // デフォルト: false; true の場合、ワークスペースブートストラップファイルから HEARTBEAT.md のみを保持
        target: "last", // デフォルト: none | オプション: last | none | <channel id> (コアまたはプラグイン、例: "bluebubbles")
        to: "+15551234567", // オプション: チャネル固有の上書き
        accountId: "ops-bot", // オプション: マルチアカウントチャネル ID
        prompt: "Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.",
        ackMaxChars: 300, // HEARTBEAT_OK 後に許可される最大文字数
      },
    },
  },
}
```

### スコープと優先順位

-   `agents.defaults.heartbeat` はグローバルな Heartbeat 動作を設定します。
-   `agents.list[].heartbeat` はその上にマージされます；いずれかのエージェントに `heartbeat` ブロックがある場合、**それらのエージェントのみ**が Heartbeat を実行します。
-   `channels.defaults.heartbeat` はすべてのチャネルの可視性デフォルトを設定します。
-   `channels..heartbeat` はチャネルデフォルトを上書きします。
-   `channels..accounts..heartbeat`（マルチアカウントチャネル）はチャネルごとの設定を上書きします。

### エージェントごとの Heartbeat

いずれかの `agents.list[]` エントリに `heartbeat` ブロックが含まれる場合、**それらのエージェントのみ**が Heartbeat を実行します。エージェントごとのブロックは `agents.defaults.heartbeat` の上にマージされます（共有デフォルトを一度設定し、エージェントごとに上書きできます）。例: 2つのエージェントのうち、2番目のエージェントのみが Heartbeat を実行します。

```json
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last", // 最後の連絡先への明示的な配信（デフォルトは "none"）
      },
    },
    list: [
      { id: "main", default: true },
      {
        id: "ops",
        heartbeat: {
          every: "1h",
          target: "whatsapp",
          to: "+15551234567",
          prompt: "Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.",
        },
      },
    ],
  },
}
```

### アクティブ時間の例

特定のタイムゾーンの営業時間内に Heartbeat を制限します:

```json
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last", // 最後の連絡先への明示的な配信（デフォルトは "none"）
        activeHours: {
          start: "09:00",
          end: "22:00",
          timezone: "America/New_York", // オプション；userTimezone が設定されていればそれを使用、それ以外はホストのタイムゾーン
        },
      },
    },
  },
}
```

このウィンドウ外（東部時間の午前9時前または午後10時後）では、Heartbeat はスキップされます。ウィンドウ内の次回のスケジュールされた実行タイミングで通常通り実行されます。

### 24時間365日稼働設定

Heartbeat を終日実行したい場合は、以下のいずれかのパターンを使用します:

-   `activeHours` を完全に省略します（時間ウィンドウ制限なし；これがデフォルトの動作です）。
-   終日ウィンドウを設定します: `activeHours: { start: "00:00", end: "24:00" }`。

同じ `start` と `end` 時間（例: `08:00` から `08:00`）を設定しないでください。これは幅ゼロのウィンドウとして扱われ、Heartbeat は常にスキップされます。

### マルチアカウントの例

Telegram などのマルチアカウントチャネルで特定のアカウントをターゲットにするには `accountId` を使用します:

```json
{
  agents: {
    list: [
      {
        id: "ops",
        heartbeat: {
          every: "1h",
          target: "telegram",
          to: "12345678:topic:42", // オプション: 特定のトピック/スレッドにルーティング
          accountId: "ops-bot",
        },
      },
    ],
  },
  channels: {
    telegram: {
      accounts: {
        "ops-bot": { botToken: "YOUR_TELEGRAM_BOT_TOKEN" },
      },
    },
  },
}
```

### フィールドの注記

-   `every`: Heartbeat 間隔（期間文字列；デフォルト単位 = 分）。
-   `model`: Heartbeat 実行のためのオプションのモデル上書き (`provider/model`)。
-   `includeReasoning`: 有効にすると、利用可能な場合、別途 `Reasoning:` メッセージも配信します（`/reasoning on` と同じ形式）。
-   `lightContext`: true の場合、Heartbeat 実行は軽量ブートストラップコンテキストを使用し、ワークスペースブートストラップファイルから `HEARTBEAT.md` のみを保持します。
-   `session`: Heartbeat 実行のためのオプションのセッションキー。
    -   `main` (デフォルト): エージェントのメインセッション。
    -   明示的なセッションキー (`openclaw sessions --json` または [sessions CLI](../cli/sessions.md) からコピー)。
    -   セッションキーの形式: [Sessions](../concepts/session.md) および [Groups](../channels/groups.md) を参照。
-   `target`:
    -   `last`: 最後に使用された外部チャネルに配信。
    -   明示的なチャネル: `whatsapp` / `telegram` / `discord` / `googlechat` / `slack` / `msteams` / `signal` / `imessage`。
    -   `none` (デフォルト): Heartbeat を実行するが、外部に**配信しない**。
-   `directPolicy`: ダイレクト/DM 配信動作を制御:
    -   `allow` (デフォルト): ダイレクト/DM Heartbeat 配信を許可。
    -   `block`: ダイレクト/DM 配信を抑制 (`reason=dm-blocked`)。
-   `to`: オプションの受信者上書き（チャネル固有の ID、例: WhatsApp の E.164 または Telegram のチャット ID）。Telegram のトピック/スレッドには、`:topic:` を使用します。
-   `accountId`: マルチアカウントチャネルのためのオプションのアカウント ID。`target: "last"` の場合、解決された最後のチャネルがアカウントをサポートしていれば、そのアカウント ID が適用されます；それ以外の場合は無視されます。アカウント ID が解決されたチャネルの設定済みアカウントと一致しない場合、配信はスキップされます。
-   `prompt`: デフォルトのプロンプト本文を上書きします（マージされません）。
-   `ackMaxChars`: 配信前に `HEARTBEAT_OK` の後に許可される最大文字数。
-   `suppressToolErrorWarnings`: true の場合、Heartbeat 実行中のツールエラー警告ペイロードを抑制します。
-   `activeHours`: Heartbeat 実行を時間ウィンドウに制限します。`start` (HH:MM、包括的；一日の開始には `00:00` を使用)、`end` (HH:MM、排他的；一日の終了には `24:00` が許可)、およびオプションの `timezone` を持つオブジェクト。
    -   省略または `"user"`: `agents.defaults.userTimezone` が設定されていればそれを使用、それ以外はホストシステムのタイムゾーンにフォールバック。
    -   `"local"`: 常にホストシステムのタイムゾーンを使用。
    -   任意の IANA 識別子 (例: `America/New_York`): 直接使用；無効な場合は、上記の `"user"` 動作にフォールバック。
    -   アクティブウィンドウの場合、`start` と `end` は等しくしてはいけません；等しい値は幅ゼロ（常にウィンドウ外）として扱われます。
    -   アクティブウィンドウ外では、ウィンドウ内の次の実行タイミングまで Heartbeat はスキップされます。

## 配信動作

-   Heartbeat はデフォルトでエージェントのメインセッション (`agent::`) で実行されます。`session.scope = "global"` の場合は `global` で実行されます。特定のチャネルセッション（Discord/WhatsApp など）に上書きするには `session` を設定します。
-   `session` は実行コンテキストにのみ影響します；配信は `target` と `to` によって制御されます。
-   特定のチャネル/受信者に配信するには、`target` + `to` を設定します。`target: "last"` の場合、配信はそのセッションの最後の外部チャネルを使用します。
-   Heartbeat 配信はデフォルトでダイレクト/DM ターゲットを許可します。Heartbeat ターンを実行しながらダイレクトターゲット送信を抑制するには `directPolicy: "block"` を設定します。
-   メインキューがビジーの場合、Heartbeat はスキップされ、後で再試行されます。
-   `target` が外部宛先に解決されない場合、実行は行われますが、送信メッセージは送信されません。
-   Heartbeat 専用の返信はセッションを**維持しません**；最後の `updatedAt` が復元されるため、アイドル期限切れは通常通り動作します。

## 可視性制御

デフォルトでは、`HEARTBEAT_OK` 確認応答は抑制され、アラート内容は配信されます。これはチャネルごとまたはアカウントごとに調整できます:

```
channels:
  defaults:
    heartbeat:
      showOk: false # HEARTBEAT_OK を非表示（デフォルト）
      showAlerts: true # アラートメッセージを表示（デフォルト）
      useIndicator: true # インジケーターイベントを発行（デフォルト）
  telegram:
    heartbeat:
      showOk: true # Telegram で OK 確認応答を表示
  whatsapp:
    accounts:
      work:
        heartbeat:
          showAlerts: false # このアカウントのアラート配信を抑制
```

優先順位: アカウントごと → チャネルごと → チャネルデフォルト → 組み込みデフォルト。

### 各フラグの動作

-   `showOk`: モデルが OK のみの返信を返した場合、`HEARTBEAT_OK` 確認応答を送信します。
-   `showAlerts`: モデルが非 OK の返信を返した場合、アラート内容を送信します。
-   `useIndicator`: UI ステータス表示のためのインジケーターイベントを発行します。

**3つすべて**が false の場合、OpenClaw は Heartbeat 実行を完全にスキップします（モデル呼び出しなし）。

### チャネルごと vs アカウントごとの例

```yaml
channels:
  defaults:
    heartbeat:
      showOk: false
      showAlerts: true
      useIndicator: true
  slack:
    heartbeat:
      showOk: true # すべての Slack アカウント
    accounts:
      ops:
        heartbeat:
          showAlerts: false # ops アカウントのみアラートを抑制
  telegram:
    heartbeat:
      showOk: true
```

### 一般的なパターン

| 目的 | 設定 |
| --- | --- |
| デフォルト動作 (OK はサイレント、アラートはオン) | *(設定不要)* |
| 完全にサイレント (メッセージなし、インジケーターなし) | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: false }` |
| インジケーターのみ (メッセージなし) | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: true }` |
| 1つのチャネルのみで OK を表示 | `channels.telegram.heartbeat: { showOk: true }` |

## HEARTBEAT.md (オプション)

ワークスペースに `HEARTBEAT.md` ファイルが存在する場合、デフォルトのプロンプトはエージェントにそれを読むように指示します。これを「Heartbeat チェックリスト」と考えてください：小さく、安定しており、30分ごとに含めても安全です。`HEARTBEAT.md` が存在しても実質的に空（空白行と `# 見出し` のようなマークダウン見出しのみ）の場合、OpenClaw は API コールを節約するために Heartbeat 実行をスキップします。ファイルが存在しない場合、Heartbeat は実行され、モデルが何をするかを決定します。プロンプトの肥大化を避けるために、小さく（短いチェックリストやリマインダー）保ってください。`HEARTBEAT.md` の例:

```bash
# Heartbeat チェックリスト

- クイックスキャン: 受信トレイに緊急のものは？
- 日中の場合、他に保留中のものがなければ軽量なチェックインを行う。
- タスクがブロックされている場合、_何が不足しているか_を書き留め、次回 Peter に尋ねる。
```

### エージェントは HEARTBEAT.md を更新できますか？

はい — そう指示すれば可能です。`HEARTBEAT.md` はエージェントワークスペース内の通常のファイルなので、通常のチャットでエージェントに次のように指示できます:

-   「`HEARTBEAT.md` を更新して、毎日のカレンダーチェックを追加してください。」
-   「`HEARTBEAT.md` を書き直して、受信トレイのフォローアップに焦点を当てた短いものにしてください。」

これを積極的に行わせたい場合は、Heartbeat プロンプトに明示的な行を含めることもできます：「チェックリストが古くなった場合は、より良いものに `HEARTBEAT.md` を更新してください。」安全上の注意：シークレット（API キー、電話番号、プライベートトークン）を `HEARTBEAT.md` に入れないでください — プロンプトコンテキストの一部になります。

## 手動ウェイク (オンデマンド)

システムイベントをキューに入れ、即座に Heartbeat をトリガーできます:

```bash
openclaw system event --text "Check for urgent follow-ups" --mode now
```

複数のエージェントに `heartbeat` が設定されている場合、手動ウェイクはそれらの各エージェントの Heartbeat を即座に実行します。次のスケジュールされた実行タイミングを待つには `--mode next-heartbeat` を使用します。

## 推論配信 (オプション)

デフォルトでは、Heartbeat は最終的な「回答」ペイロードのみを配信します。透明性が必要な場合は、以下を有効にします:

-   `agents.defaults.heartbeat.includeReasoning: true`

有効にすると、Heartbeat は `Reasoning:` という接頭辞が付いた別のメッセージも配信します（`/reasoning on` と同じ形式）。これは、エージェントが複数のセッション/コードを管理していて、なぜあなたに通知することを決定したのかを知りたい場合に役立ちます — ただし、望ましい以上の内部詳細が漏れる可能性もあります。グループチャットではオフにしておくことをお勧めします。

## コスト意識

Heartbeat は完全なエージェントターンを実行します。間隔が短いほど、より多くのトークンを消費します。`HEARTBEAT.md` を小さく保ち、より安価な `model` または `target: "none"` を検討してください（内部状態の更新のみが必要な場合）。

[ヘルスチェック](./health.md)[Doctor](./doctor.md)