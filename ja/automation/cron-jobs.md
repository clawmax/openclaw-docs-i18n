title: "OpenClaw Cronジョブガイド: スケジューリングと自動化"
description: "OpenClaw cronジョブで自動化タスクをスケジュールする方法を学びます。ワンショットリマインダー、繰り返しタスクの設定、メイン実行モードと分離実行モードの選択について説明します。"
keywords: ["openclaw cron", "自動化スケジューリング", "cronジョブ", "ゲートウェイスケジューラ", "分離エージェントターン", "システムイベント", "繰り返しタスク", "ジョブ配信"]
---

  自動化

  
# Cronジョブ

> **Cron vs Heartbeat?** それぞれの使用タイミングについては、[Cron vs Heartbeat](./cron-vs-heartbeat.md) を参照してください。

CronはGatewayの組み込みスケジューラです。ジョブを永続化し、適切な時間にエージェントを起動し、オプションで出力をチャットに配信できます。「毎朝これを実行する」や「20分後にエージェントに通知する」といった場合、cronがその仕組みです。トラブルシューティング: [/automation/troubleshooting](./troubleshooting.md)

## 概要

-   Cronは **Gateway内部** で実行されます（モデル内部ではありません）。
-   ジョブは `~/.openclaw/cron/` 以下に永続化されるため、再起動してもスケジュールは失われません。
-   2つの実行スタイル:
    -   **メインセッション**: システムイベントをキューに入れ、次のハートビートで実行します。
    -   **分離**: `cron:` で専用のエージェントターンを実行し、配信を行います（デフォルトはアナウンス、またはなし）。
-   ウェイクアップは第一級です: ジョブは「今すぐ起動」と「次のハートビート」を要求できます。
-   Webhook投稿はジョブごとに `delivery.mode = "webhook"` + `delivery.to = ""` で設定します。
-   レガシーフォールバック: `notify: true` が設定された保存済みジョブは、`cron.webhook` が設定されている場合、そのジョブをwebhook配信モードに移行する警告とともに、引き続き使用されます。

## クイックスタート (実用的)

ワンショットリマインダーを作成し、存在を確認し、即時実行します:

```bash
openclaw cron add \
  --name "Reminder" \
  --at "2026-02-01T16:00:00Z" \
  --session main \
  --system-event "Reminder: check the cron docs draft" \
  --wake now \
  --delete-after-run

openclaw cron list
openclaw cron run <job-id>
openclaw cron runs --id <job-id>
```

配信付きの繰り返し分離ジョブをスケジュールします:

```bash
openclaw cron add \
  --name "Morning brief" \
  --cron "0 7 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize overnight updates." \
  --announce \
  --channel slack \
  --to "channel:C1234567890"
```

## ツールコール相当 (Gateway cronツール)

正規のJSON形式と例については、[ツールコール用JSONスキーマ](./cron-jobs.md#json-schema-for-tool-calls) を参照してください。

## Cronジョブの保存場所

CronジョブはデフォルトでGatewayホストの `~/.openclaw/cron/jobs.json` に永続化されます。Gatewayはファイルをメモリに読み込み、変更時に書き戻すため、手動編集はGatewayが停止している場合にのみ安全です。変更には `openclaw cron add/edit` またはcronツールコールAPIを優先してください。

## 初心者向け概要

cronジョブは **いつ** 実行するか + **何を** するか、と考えてください。

1.  **スケジュールを選択**
    -   ワンショットリマインダー → `schedule.kind = "at"` (CLI: `--at`)
    -   繰り返しジョブ → `schedule.kind = "every"` または `schedule.kind = "cron"`
    -   ISOタイムスタンプにタイムゾーンが含まれていない場合、**UTC** として扱われます。
2.  **実行場所を選択**
    -   `sessionTarget: "main"` → 次のハートビート時にメインコンテキストで実行。
    -   `sessionTarget: "isolated"` → `cron:` で専用のエージェントターンを実行。
3.  **ペイロードを選択**
    -   メインセッション → `payload.kind = "systemEvent"`
    -   分離セッション → `payload.kind = "agentTurn"`

オプション: ワンショットジョブ (`schedule.kind = "at"`) はデフォルトで成功後に削除されます。保持するには `deleteAfterRun: false` を設定します（成功後は無効化されます）。

## 概念

### ジョブ

Cronジョブは以下の情報を持つ保存されたレコードです:

-   **スケジュール** (いつ実行するか),
-   **ペイロード** (何を実行するか),
-   オプションの **配信モード** (`announce`, `webhook`, `none`)。
-   オプションの **エージェントバインディング** (`agentId`): 特定のエージェントの下でジョブを実行します。欠落または不明な場合、ゲートウェイはデフォルトエージェントにフォールバックします。

ジョブは安定した `jobId` で識別されます（CLI/Gateway APIで使用）。エージェントツールコールでは、`jobId` が正規です。互換性のためにレガシーな `id` も受け入れられます。ワンショットジョブはデフォルトで成功後に自動削除されます。保持するには `deleteAfterRun: false` を設定します。

### スケジュール

Cronは3種類のスケジュールをサポートします:

-   `at`: `schedule.at` によるワンショットのタイムスタンプ (ISO 8601)。
-   `every`: 固定間隔 (ミリ秒)。
-   `cron`: 5フィールドのcron式（または秒を含む6フィールド）とオプションのIANAタイムゾーン。

Cron式は `croner` を使用します。タイムゾーンが省略された場合、Gatewayホストのローカルタイムゾーンが使用されます。多数のゲートウェイ間での時間先頭の負荷スパイクを軽減するため、OpenClawは繰り返しの時間先頭式（例: `0 * * * *`, `0 */2 * * *`）に対して、最大5分の決定論的なジョブごとのスタッガーウィンドウを適用します。`0 7 * * *` のような固定時間式は正確なままです。任意のcronスケジュールに対して、明示的なスタッガーウィンドウを `schedule.staggerMs` で設定できます（`0` は正確なタイミングを維持）。CLIショートカット:

-   `--stagger 30s` (または `1m`, `5m`) で明示的なスタッガーウィンドウを設定。
-   `--exact` で `staggerMs = 0` を強制。

### メイン vs 分離実行

#### メインセッションジョブ (システムイベント)

メインジョブはシステムイベントをキューに入れ、オプションでハートビートランナーを起動します。`payload.kind = "systemEvent"` を使用する必要があります。

-   `wakeMode: "now"` (デフォルト): イベントが即時ハートビート実行をトリガー。
-   `wakeMode: "next-heartbeat"`: イベントは次のスケジュールされたハートビートを待機。

これは通常のハートビートプロンプト + メインセッションコンテキストが必要な場合に最適です。[Heartbeat](../gateway/heartbeat.md) を参照してください。

#### 分離ジョブ (専用cronセッション)

分離ジョブはセッション `cron:` で専用のエージェントターンを実行します。主な動作:

-   トレーサビリティのため、プロンプトには `[cron: ]` がプレフィックスとして付きます。
-   各実行は **新しいセッションID** で開始されます（以前の会話の引き継ぎはありません）。
-   デフォルト動作: `delivery` が省略された場合、分離ジョブは要約をアナウンスします (`delivery.mode = "announce"`)。
-   `delivery.mode` で動作を選択:
    -   `announce`: 要約をターゲットチャンネルに配信し、簡単な要約をメインセッションに投稿します。
    -   `webhook`: 完了イベントに要約が含まれる場合、完了イベントペイロードを `delivery.to` にPOSTします。
    -   `none`: 内部のみ（配信なし、メインセッション要約なし）。
-   `wakeMode` はメインセッション要約の投稿タイミングを制御:
    -   `now`: 即時ハートビート。
    -   `next-heartbeat`: 次のスケジュールされたハートビートを待機。

メインのチャット履歴をスパムしないようにすべき、ノイジーで頻繁な、または「バックグラウンド雑務」には分離ジョブを使用します。

### ペイロード形式 (実行内容)

2種類のペイロードがサポートされています:

-   `systemEvent`: メインセッションのみ、ハートビートプロンプトを経由します。
-   `agentTurn`: 分離セッションのみ、専用のエージェントターンを実行します。

一般的な `agentTurn` フィールド:

-   `message`: 必須のテキストプロンプト。
-   `model` / `thinking`: オプションのオーバーライド（下記参照）。
-   `timeoutSeconds`: オプションのタイムアウトオーバーライド。
-   `lightContext`: ワークスペースブートストラップファイルの注入を必要としないジョブのためのオプションの軽量ブートストラップモード。

配信設定:

-   `delivery.mode`: `none` | `announce` | `webhook`。
-   `delivery.channel`: `last` または特定のチャンネル。
-   `delivery.to`: チャンネル固有のターゲット（アナウンス）またはwebhook URL（webhookモード）。
-   `delivery.bestEffort`: アナウンス配信が失敗してもジョブを失敗させない。

アナウンス配信は実行中のメッセージングツール送信を抑制します。代わりにチャットをターゲットするには `delivery.channel`/`delivery.to` を使用します。`delivery.mode = "none"` の場合、メインセッションには要約が投稿されません。分離ジョブで `delivery` が省略された場合、OpenClawはデフォルトで `announce` になります。

#### アナウンス配信フロー

`delivery.mode = "announce"` の場合、cronはアウトバウンドチャンネルアダプターを介して直接配信します。メインエージェントはメッセージを作成または転送するために起動されません。動作の詳細:

-   内容: 配信は分離実行のアウトバウンドペイロード（テキスト/メディア）を通常のチャンキングとチャンネルフォーマットで使用します。
-   ハートビートのみの応答（実際の内容がない `HEARTBEAT_OK`）は配信されません。
-   分離実行がすでにメッセージツールを介して同じターゲットにメッセージを送信している場合、重複を避けるため配信はスキップされます。
-   欠落または無効な配信ターゲットは、`delivery.bestEffort = true` でない限りジョブを失敗させます。
-   短い要約は `delivery.mode = "announce"` の場合にのみメインセッションに投稿されます。
-   メインセッション要約は `wakeMode` を尊重します: `now` は即時ハートビートをトリガーし、`next-heartbeat` は次のスケジュールされたハートビートを待機します。

#### Webhook配信フロー

`delivery.mode = "webhook"` の場合、cronは完了イベントに要約が含まれる場合、完了イベントペイロードを `delivery.to` にPOSTします。動作の詳細:

-   エンドポイントは有効なHTTP(S) URLである必要があります。
-   webhookモードではチャンネル配信は試みられません。
-   webhookモードではメインセッション要約は投稿されません。
-   `cron.webhookToken` が設定されている場合、認証ヘッダーは `Authorization: Bearer <cron.webhookToken>` です。
-   非推奨フォールバック: `notify: true` が設定された保存済みレガシージョブは、`cron.webhook` に（設定されている場合）引き続き投稿され、`delivery.mode = "webhook"` に移行する警告が表示されます。

### モデルと思考オーバーライド

分離ジョブ (`agentTurn`) はモデルと思考レベルをオーバーライドできます:

-   `model`: プロバイダー/モデル文字列 (例: `anthropic/claude-sonnet-4-20250514`) またはエイリアス (例: `opus`)
-   `thinking`: 思考レベル (`off`, `minimal`, `low`, `medium`, `high`, `xhigh`; GPT-5.2 + Codexモデルのみ)

注: メインセッションジョブにも `model` を設定できますが、共有メインセッションモデルが変更されます。予期しないコンテキストシフトを避けるため、モデルオーバーライドは分離ジョブのみに使用することをお勧めします。解決優先順位:

1.  ジョブペイロードオーバーライド (最高)
2.  フック固有のデフォルト (例: `hooks.gmail.model`)
3.  エージェント設定デフォルト

### 軽量ブートストラップコンテキスト

分離ジョブ (`agentTurn`) は `lightContext: true` を設定して軽量ブートストラップコンテキストで実行できます。

-   ワークスペースブートストラップファイルの注入を必要としないスケジュールされた雑務に使用します。
-   実際には、組み込みランタイムは `bootstrapContextMode: "lightweight"` で実行され、意図的にcronブートストラップコンテキストを空に保ちます。
-   CLI相当: `openclaw cron add --light-context ...` および `openclaw cron edit --light-context`。

### 配信 (チャンネル + ターゲット)

分離ジョブはトップレベルの `delivery` 設定を介して出力をチャンネルに配信できます:

-   `delivery.mode`: `announce` (チャンネル配信), `webhook` (HTTP POST), `none`。
-   `delivery.channel`: `whatsapp` / `telegram` / `discord` / `slack` / `mattermost` (プラグイン) / `signal` / `imessage` / `last`。
-   `delivery.to`: チャンネル固有の受信者ターゲット。

`announce` 配信は分離ジョブ (`sessionTarget: "isolated"`) のみ有効です。`webhook` 配信はメインジョブと分離ジョブの両方で有効です。`delivery.channel` または `delivery.to` が省略された場合、cronはメインセッションの「最後のルート」（エージェントが最後に返信した場所）にフォールバックできます。ターゲット形式の注意点:

-   Slack/Discord/Mattermost (プラグイン) ターゲットは、曖昧さを避けるために明示的なプレフィックス (例: `channel:`, `user:`) を使用する必要があります。
-   Telegramトピックは `:topic:` 形式を使用する必要があります（下記参照）。

#### Telegram配信ターゲット (トピック / フォーラムスレッド)

Telegramは `message_thread_id` を介してフォーラムトピックをサポートします。cron配信では、トピック/スレッドを `to` フィールドにエンコードできます:

-   `-1001234567890` (チャットIDのみ)
-   `-1001234567890:topic:123` (推奨: 明示的なトピックマーカー)
-   `-1001234567890:123` (省略形: 数値サフィックス)

`telegram:...` / `telegram:group:...` のようなプレフィックス付きターゲットも受け入れられます:

-   `telegram:group:-1001234567890:topic:123`

## ツールコール用JSONスキーマ

Gateway `cron.*` ツールを直接呼び出す場合（エージェントツールコールまたはRPC）、これらの形式を使用してください。CLIフラグは `20m` のような人間が読める期間を受け入れますが、ツールコールでは `schedule.at` にはISO 8601文字列、`schedule.everyMs` にはミリ秒を使用する必要があります。

### cron.add パラメータ

ワンショット、メインセッションジョブ (システムイベント):

```json
{
  "name": "Reminder",
  "schedule": { "kind": "at", "at": "2026-02-01T16:00:00Z" },
  "sessionTarget": "main",
  "wakeMode": "now",
  "payload": { "kind": "systemEvent", "text": "Reminder text" },
  "deleteAfterRun": true
}
```

繰り返し、配信付き分離ジョブ:

```json
{
  "name": "Morning brief",
  "schedule": { "kind": "cron", "expr": "0 7 * * *", "tz": "America/Los_Angeles" },
  "sessionTarget": "isolated",
  "wakeMode": "next-heartbeat",
  "payload": {
    "kind": "agentTurn",
    "message": "Summarize overnight updates.",
    "lightContext": true
  },
  "delivery": {
    "mode": "announce",
    "channel": "slack",
    "to": "channel:C1234567890",
    "bestEffort": true
  }
}
```

注意:

-   `schedule.kind`: `at` (`at`), `every` (`everyMs`), `cron` (`expr`, オプション `tz`)。
-   `schedule.at` はISO 8601を受け入れます（タイムゾーンはオプション。省略時はUTCとして扱われます）。
-   `everyMs` はミリ秒です。
-   `sessionTarget` は `"main"` または `"isolated"` である必要があり、`payload.kind` と一致する必要があります。
-   オプションフィールド: `agentId`, `description`, `enabled`, `deleteAfterRun` (`at` のデフォルトはtrue), `delivery`。
-   `wakeMode` は省略時、デフォルトで `"now"` になります。

### cron.update パラメータ

```json
{
  "jobId": "job-123",
  "patch": {
    "enabled": false,
    "schedule": { "kind": "every", "everyMs": 3600000 }
  }
}
```

注意:

-   `jobId` が正規です。互換性のために `id` も受け入れられます。
-   エージェントバインディングをクリアするには、パッチで `agentId: null` を使用します。

### cron.run および cron.remove パラメータ

```json
{ "jobId": "job-123", "mode": "force" }
```

```json
{ "jobId": "job-123" }
```

## ストレージ & 履歴

-   ジョブストア: `~/.openclaw/cron/jobs.json` (Gateway管理JSON)。
-   実行履歴: `~/.openclaw/cron/runs/.jsonl` (JSONL、サイズと行数で自動プルーニング)。
-   `sessions.json` 内の分離cron実行セッションは `cron.sessionRetention` でプルーニングされます（デフォルト `24h`。無効化するには `false` を設定）。
-   ストアパスのオーバーライド: 設定内の `cron.store`。

## リトライポリシー

ジョブが失敗した場合、OpenClawはエラーを **一時的** (再試行可能) または **永続的** (即時無効化) に分類します。

### 一時的エラー (再試行)

-   レート制限 (429, リクエスト過多, リソース枯渇)
-   プロバイダー過負荷 (例: Anthropic `529 overloaded_error`, 過負荷フォールバック要約)
-   ネットワークエラー (タイムアウト, ECONNRESET, fetch failed, socket)
-   サーバーエラー (5xx)
-   Cloudflare関連エラー

### 永続的エラー (再試行なし)

-   認証失敗 (無効なAPIキー, 権限なし)
-   設定または検証エラー
-   その他の非一時的エラー

### デフォルト動作 (設定なし)

**ワンショットジョブ (`schedule.kind: "at"`):**

-   一時的エラー時: 指数バックオフ (30s → 1m → 5m) で最大3回再試行。
-   永続的エラー時: 即時無効化。
-   成功またはスキップ時: 無効化（または `deleteAfterRun: true` の場合削除）。

**繰り返しジョブ (`cron` / `every`):**

-   エラー時: 次のスケジュール実行前に指数バックオフ (30s → 1m → 5m → 15m → 60m) を適用。
-   ジョブは有効のまま。次の成功実行後にバックオフはリセットされます。

これらのデフォルトをオーバーライドするには `cron.retry` を設定します（[設定](./cron-jobs.md#configuration) を参照）。

## 設定

```json
{
  cron: {
    enabled: true, // デフォルト true
    store: "~/.openclaw/cron/jobs.json",
    maxConcurrentRuns: 1, // デフォルト 1
    // オプション: ワンショットジョブのリトライポリシーをオーバーライド
    retry: {
      maxAttempts: 3,
      backoffMs: [60000, 120000, 300000],
      retryOn: ["rate_limit", "overloaded", "network", "server_error"],
    },
    webhook: "https://example.invalid/legacy", // 保存済み notify:true ジョブの非推奨フォールバック
    webhookToken: "replace-with-dedicated-webhook-token", // webhookモード用のオプションベアラートークン
    sessionRetention: "24h", // 期間文字列または false
    runLog: {
      maxBytes: "2mb", // デフォルト 2_000_000 バイト
      keepLines: 2000, // デフォルト 2000
    },
  },
}
```

実行ログプルーニング動作:

-   `cron.runLog.maxBytes`: プルーニング前の最大実行ログファイルサイズ。
-   `cron.runLog.keepLines`: プルーニング時、最新のN行のみ保持。
-   両方とも `cron/runs/.jsonl` ファイルに適用されます。

Webhook動作:

-   推奨: ジョブごとに `delivery.mode: "webhook"` と `delivery.to: "https://..."` を設定。
-   Webhook URLは有効な `http://` または `https://` URLである必要があります。
-   投稿時、ペイロードはcron完了イベントJSONです。
-   `cron.webhookToken` が設定されている場合、認証ヘッダーは `Authorization: Bearer <cron.webhookToken>` です。
-   `cron.webhookToken` が設定されていない場合、`Authorization` ヘッダーは送信されません。
-   非推奨フォールバック: `notify: true` が設定された保存済みレガシージョブは、存在する場合 `cron.webhook` を引き続き使用します。

Cron全体を無効化:

-   `cron.enabled: false` (設定)
-   `OPENCLAW_SKIP_CRON=1` (環境変数)

## メンテナンス

Cronには2つの組み込みメンテナンスパスがあります: 分離実行セッションの保持と実行ログのプルーニングです。

### デフォルト

-   `cron.sessionRetention`: `24h` (実行セッションプルーニングを無効化するには `false` を設定)
-   `cron.runLog.maxBytes`: `2_000_000` バイト
-   `cron.runLog.keepLines`: `2000`

### 仕組み

-   分離実行はセッションエントリ (`...:cron::run:`) とトランスクリプトファイルを作成します。
-   リーパーは `cron.sessionRetention` より古い期限切れの実行セッションエントリを削除します。
-   セッションストアから参照されなくなった削除された実行セッションについて、OpenClawはトランスクリプトファイルをアーカイブし、同じ保持ウィンドウで古い削除済みアーカイブをパージします。
-   各実行追加後、`cron/runs/.jsonl` はサイズチェックされます:
    -   ファイルサイズが `runLog.maxBytes` を超える場合、最新の `runLog.keepLines` 行にトリミングされます。

### 高頻度スケジューラのパフォーマンスに関する注意点

高頻度cron設定は、大きな実行セッションと実行ログのフットプリントを生成する可能性があります。メンテナンスは組み込まれていますが、緩い制限は依然として回避可能なIOとクリーンアップ作業を生み出す可能性があります。注意点:

-   多くの分離実行がある長い `cron.sessionRetention` ウィンドウ
-   大きな `runLog.maxBytes` と組み合わされた高い `cron.runLog.keepLines`
-   同じ `cron/runs/.jsonl` に書き込む多くのノイジーな繰り返しジョブ

対策:

-   デバッグ/監査ニーズに応じて `cron.sessionRetention` を可能な限り短く保つ
-   適度な `runLog.maxBytes` と `runLog.keepLines` で実行ログを制限する
-   ノイジーなバックグラウンドジョブを分離モードに移行し、不要なチャターを避ける配信ルールを設定する
-   `openclaw cron runs` で定期的に成長を確認し、ログが大きくなる前に保持期間を調整する

### カスタマイズ例

実行セッションを1週間保持し、より大きな実行ログを許可:

```json
{
  cron: {
    sessionRetention: "7d",
    runLog: {
      maxBytes: "10mb",
      keepLines: 5000,
    },
  },
}
```

分離実行セッションプルーニングを無効化し、実行ログプルーニングは保持:

```json
{
  cron: {
    sessionRetention: false,
    runLog: {
      maxBytes: "5mb",
      keepLines: 3000,
    },
  },
}
```

高頻度cron使用の調整 (例):

```json
{
  cron: {
    sessionRetention: "12h",
    runLog: {
      maxBytes: "3mb",
      keepLines: 1500,
    },
  },
}
```

## CLIクイックスタート

ワンショットリマインダー (UTC ISO, 成功後自動削除):

```bash
openclaw cron add \
  --name "Send reminder" \
  --at "2026-01-12T18:00:00Z" \
  --session main \
  --system-event "Reminder: submit expense report." \
  --wake now \
  --delete-after-run
```

ワンショットリマインダー (メインセッション, 即時起動):

```bash
openclaw cron add \
  --name "Calendar check" \
  --at "20m" \
  --session main \
  --system-event "Next heartbeat: check calendar." \
  --wake now
```

繰り返し分離ジョブ (WhatsAppにアナウンス):

```bash
openclaw cron add \
  --name "Morning status" \
  --cron "0 7 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize inbox + calendar for today." \
  --announce \
  --channel whatsapp \
  --to "+15551234567"
```

明示的な30秒スタッガー付き繰り返しcronジョブ:

```bash
openclaw cron add \
  --name "Minute watcher" \
  --cron "0 * * * * *" \
  --tz "UTC" \
  --stagger 30s \
  --session isolated \
  --message "Run minute watcher checks." \
  --announce
```

繰り返し分離ジョブ (Telegramトピックに配信):

```bash
openclaw cron add \
  --name "Nightly summary (topic)" \
  --cron "0 22 * * *" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Summarize today; send to the nightly topic." \
  --announce \
  --channel telegram \
  --to "-1001234567890:topic:123"
```

モデルと思考オーバーライド付き分離ジョブ:

```bash
openclaw cron add \
  --name "Deep analysis" \
  --cron "0 6 * * 1" \
  --tz "America/Los_Angeles" \
  --session isolated \
  --message "Weekly deep analysis of project progress." \
  --model "opus" \
  --thinking high \
  --announce \
  --channel whatsapp \
  --to "+15551234567"
```

エージェント選択 (マルチエージェント設定):

```bash
# ジョブをエージェント "ops" に固定（そのエージェントが欠落している場合はデフォルトにフォールバック）
openclaw cron add --name "Ops sweep" --cron "0 6 * * *" --session isolated --message "Check ops queue" --agent ops

# 既存ジョブのエージェントを切り替えまたはクリア
openclaw cron edit <jobId> --agent ops
openclaw cron edit <jobId> --clear-agent
```

手動実行 (forceがデフォルト、期限時のみ実行するには `--due` を使用):

```bash
openclaw cron run <jobId>
openclaw cron run <jobId> --due
```

既存ジョブの編集 (フィールドのパッチ):

```bash
openclaw cron edit <jobId> \
  --message "Updated prompt" \
  --model "opus" \
  --thinking low
```

既存cronジョブをスケジュール通り正確に実行 (スタッガーなし):

```bash
openclaw cron edit <jobId> --exact
```

実行履歴:

```bash
openclaw cron runs --id <jobId> --limit 50
```

ジョブを作成せずに即時システムイベント:

```bash
openclaw system event --mode now --text "Next heartbeat: check battery."
```

## Gateway APIサーフェス

-   `cron.list`, `cron.status`, `cron.add`, `cron.update`, `cron.remove`
-   `cron.run` (forceまたはdue),