

  概要

  
# ツール

OpenClaw は、ブラウザ、キャンバス、ノード、cron 向けの **ファーストクラスエージェントツール** を公開しています。これらは古い `openclaw-*` スキルを置き換えるものです：ツールは型付けされており、シェル呼び出しは不要で、エージェントは直接これらに依存すべきです。

## ツールの無効化

`openclaw.json` の `tools.allow` / `tools.deny` を使用して、ツールをグローバルに許可/拒否できます（deny が優先されます）。これにより、許可されていないツールがモデルプロバイダーに送信されるのを防ぎます。

```json
{
  tools: { deny: ["browser"] },
}
```

注意点:

-   マッチングは大文字小文字を区別しません。
-   `*` ワイルドカードがサポートされています（`"*"` はすべてのツールを意味します）。
-   `tools.allow` が未知またはロードされていないプラグインツール名のみを参照している場合、OpenClaw は警告をログに記録し、許可リストを無視するため、コアツールは利用可能なままです。

## ツールプロファイル (基本許可リスト)

`tools.profile` は、`tools.allow`/`tools.deny` の前に適用される **基本ツール許可リスト** を設定します。エージェントごとのオーバーライド: `agents.list[].tools.profile`。プロファイル:

-   `minimal`: `session_status` のみ
-   `coding`: `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`
-   `messaging`: `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status`
-   `full`: 制限なし（未設定と同じ）

例 (デフォルトでメッセージングのみ、Slack + Discord ツールも許可):

```json
{
  tools: {
    profile: "messaging",
    allow: ["slack", "discord"],
  },
}
```

例 (コーディングプロファイル、ただし exec/process をどこでも拒否):

```json
{
  tools: {
    profile: "coding",
    deny: ["group:runtime"],
  },
}
```

例 (グローバルコーディングプロファイル、メッセージングのみのサポートエージェント):

```json
{
  tools: { profile: "coding" },
  agents: {
    list: [
      {
        id: "support",
        tools: { profile: "messaging", allow: ["slack"] },
      },
    ],
  },
}
```

## プロバイダー固有のツールポリシー

`tools.byProvider` を使用して、グローバルデフォルトを変更せずに、特定のプロバイダー（または単一の `provider/model`）に対してツールを **さらに制限** できます。エージェントごとのオーバーライド: `agents.list[].tools.byProvider`。これは基本ツールプロファイルの **後**、許可/拒否リストの **前** に適用されるため、ツールセットを狭めることしかできません。プロバイダーキーは `provider`（例: `google-antigravity`）または `provider/model`（例: `openai/gpt-5.2`）のいずれかを受け入れます。例 (グローバルコーディングプロファイルを維持、ただし Google Antigravity には最小限のツール):

```json
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" },
    },
  },
}
```

例 (不安定なエンドポイント向けのプロバイダー/モデル固有の許可リスト):

```json
{
  tools: {
    allow: ["group:fs", "group:runtime", "sessions_list"],
    byProvider: {
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] },
    },
  },
}
```

例 (単一プロバイダー向けのエージェント固有オーバーライド):

```json
{
  agents: {
    list: [
      {
        id: "support",
        tools: {
          byProvider: {
            "google-antigravity": { allow: ["message", "sessions_list"] },
          },
        },
      },
    ],
  },
}
```

## ツールグループ (短縮形)

ツールポリシー（グローバル、エージェント、サンドボックス）は、複数のツールに展開される `group:*` エントリをサポートします。`tools.allow` / `tools.deny` でこれらを使用してください。利用可能なグループ:

-   `group:runtime`: `exec`, `bash`, `process`
-   `group:fs`: `read`, `write`, `edit`, `apply_patch`
-   `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
-   `group:memory`: `memory_search`, `memory_get`
-   `group:web`: `web_search`, `web_fetch`
-   `group:ui`: `browser`, `canvas`
-   `group:automation`: `cron`, `gateway`
-   `group:messaging`: `message`
-   `group:nodes`: `nodes`
-   `group:openclaw`: すべての組み込み OpenClaw ツール（プロバイダープラグインを除く）

例 (ファイルツール + ブラウザのみを許可):

```json
{
  tools: {
    allow: ["group:fs", "browser"],
  },
}
```

## プラグイン + ツール

プラグインは、コアセットを超えて **追加のツール**（および CLI コマンド）を登録できます。インストールと設定については [プラグイン](./tools/plugin.md) を、ツール使用ガイダンスがプロンプトに注入される方法については [スキル](./tools/skills.md) を参照してください。一部のプラグインは、ツールと一緒に独自のスキルを同梱しています（例: ボイスコールプラグイン）。オプションのプラグインツール:

-   [Lobster](./tools/lobster.md): 再開可能な承認を備えた型付きワークフローランタイム（ゲートウェイホストに Lobster CLI が必要）。
-   [LLM Task](./tools/llm-task.md): 構造化ワークフロー出力のための JSON 専用 LLM ステップ（オプションのスキーマ検証付き）。
-   [Diffs](./tools/diffs.md): 読み取り専用の差分ビューアーと、前後テキストまたは unified パッチ用の PNG または PDF ファイルレンダラー。

## ツールインベントリ

### apply\_patch

構造化パッチを1つ以上のファイルに適用します。複数ハンクの編集に使用します。実験的: `tools.exec.applyPatch.enabled` で有効化（OpenAI モデルのみ）。`tools.exec.applyPatch.workspaceOnly` はデフォルトで `true`（ワークスペース内のみ）。`apply_patch` にワークスペースディレクトリ外への書き込み/削除を意図的に行わせたい場合のみ、これを `false` に設定してください。

### exec

ワークスペース内でシェルコマンドを実行します。コアパラメータ:

-   `command` (必須)
-   `yieldMs` (タイムアウト後の自動バックグラウンド化、デフォルト 10000)
-   `background` (即時バックグラウンド化)
-   `timeout` (秒; 超過するとプロセスを強制終了、デフォルト 1800)
-   `elevated` (bool; 昇格モードが有効/許可されている場合、ホスト上で実行; エージェントがサンドボックス化されている場合のみ動作が変化)
-   `host` (`sandbox | gateway | node`)
-   `security` (`deny | allowlist | full`)
-   `ask` (`off | on-miss | always`)
-   `node` (`host=node` 用のノード ID/名前)
-   本物の TTY が必要ですか？ `pty: true` を設定してください。

注意点:

-   バックグラウンド化されると、`sessionId` 付きで `status: "running"` を返します。
-   バックグラウンドセッションのポーリング/ログ取得/書き込み/強制終了/クリアには `process` を使用してください。
-   `process` が許可されていない場合、`exec` は同期的に実行され、`yieldMs`/`background` は無視されます。
-   `elevated` は `tools.elevated` と任意の `agents.list[].tools.elevated` オーバーライドによって制御され（両方が許可する必要があります）、`host=gateway` + `security=full` のエイリアスです。
-   `elevated` はエージェントがサンドボックス化されている場合のみ動作が変化します（それ以外は無効）。
-   `host=node` は、macOS コンパニオンアプリまたはヘッドレスノードホスト（`openclaw node run`）をターゲットにできます。
-   ゲートウェイ/ノードの承認と許可リスト: [Exec 承認](./tools/exec-approvals.md)。

### process

バックグラウンド exec セッションを管理します。コアアクション:

-   `list`, `poll`, `log`, `write`, `kill`, `clear`, `remove`

注意点:

-   `poll` は、完了時に新しい出力と終了ステータスを返します。
-   `log` は行ベースの `offset`/`limit` をサポートします（`offset` を省略すると最後の N 行を取得）。
-   `process` はエージェントごとにスコープされます；他のエージェントからのセッションは表示されません。

### loop-detection (ツール呼び出しループガードレール)

OpenClaw は最近のツール呼び出し履歴を追跡し、反復的な進捗のないループを検出したときにブロックまたは警告します。`tools.loopDetection.enabled: true` で有効化（デフォルトは `false`）。

```json
{
  tools: {
    loopDetection: {
      enabled: true,
      warningThreshold: 10,
      criticalThreshold: 20,
      globalCircuitBreakerThreshold: 30,
      historySize: 30,
      detectors: {
        genericRepeat: true,
        knownPollNoProgress: true,
        pingPong: true,
      },
    },
  },
}
```

-   `genericRepeat`: 同じツール + 同じパラメータの呼び出しパターンの繰り返し。
-   `knownPollNoProgress`: 同一出力を持つポーリング類似ツールの繰り返し。
-   `pingPong`: 交互の `A/B/A/B` 進捗のないパターン。
-   エージェントごとのオーバーライド: `agents.list[].tools.loopDetection`。

### web\_search

Perplexity、Brave、Gemini、Grok、または Kimi を使用してウェブを検索します。コアパラメータ:

-   `query` (必須)
-   `count` (1–10; デフォルトは `tools.web.search.maxResults` から)

注意点:

-   選択したプロバイダーの API キーが必要です（推奨: `openclaw configure --section web`）。
-   `tools.web.search.enabled` で有効化。
-   応答はキャッシュされます（デフォルト 15 分）。
-   設定については [ウェブツール](./tools/web.md) を参照。

### web\_fetch

URL から読み取り可能なコンテンツを取得して抽出します（HTML → マークダウン/テキスト）。コアパラメータ:

-   `url` (必須)
-   `extractMode` (`markdown` | `text`)
-   `maxChars` (長いページを切り詰め)

注意点:

-   `tools.web.fetch.enabled` で有効化。
-   `maxChars` は `tools.web.fetch.maxCharsCap` で制限されます（デフォルト 50000）。
-   応答はキャッシュされます（デフォルト 15 分）。
-   JS が多用されるサイトには、ブラウザツールを優先してください。
-   設定については [ウェブツール](./tools/web.md) を参照。
-   オプションのアンチボットフォールバックについては [Firecrawl](./tools/firecrawl.md) を参照。

### browser

専用の OpenClaw 管理ブラウザを制御します。コアアクション:

-   `status`, `start`, `stop`, `tabs`, `open`, `focus`, `close`
-   `snapshot` (aria/ai)
-   `screenshot` (画像ブロック + `MEDIA:` を返す)
-   `act` (UI アクション: click/type/press/hover/drag/select/fill/resize/wait/evaluate)
-   `navigate`, `console`, `pdf`, `upload`, `dialog`

プロファイル管理:

-   `profiles` — ステータス付きのすべてのブラウザプロファイルを一覧表示
-   `create-profile` — 自動割り当てポート（または `cdpUrl`）で新しいプロファイルを作成
-   `delete-profile` — ブラウザを停止、ユーザーデータを削除、設定から削除（ローカルのみ）
-   `reset-profile` — プロファイルのポート上の孤立プロセスを強制終了（ローカルのみ）

共通パラメータ:

-   `profile` (オプション; デフォルトは `browser.defaultProfile`)
-   `target` (`sandbox` | `host` | `node`)
-   `node` (オプション; 特定のノード ID/名前を選択) 注意点:
-   `browser.enabled=true` が必要（デフォルトは `true`；無効化するには `false` を設定）。
-   すべてのアクションは、マルチインスタンスサポートのためのオプションの `profile` パラメータを受け入れます。
-   `profile` が省略された場合、`browser.defaultProfile` を使用します（デフォルトは "chrome"）。
-   プロファイル名: 小文字英数字 + ハイフンのみ（最大 64 文字）。
-   ポート範囲: 18800-18899（最大約100プロファイル）。
-   リモートプロファイルはアタッチのみ（開始/停止/リセット不可）。
-   ブラウザ対応ノードが接続されている場合、ツールは自動的にそこにルーティングする可能性があります（`target` を固定しない限り）。
-   `snapshot` は Playwright がインストールされている場合、デフォルトで `ai` を使用；アクセシビリティツリーには `aria` を使用。
-   `snapshot` はロールスナップショットオプション（`interactive`, `compact`, `depth`, `selector`）もサポートし、`e12` のような参照を返します。
-   `act` には `snapshot` からの `ref` が必要（AI スナップショットからの数値 `12`、またはロールスナップショットからの `e12`）；稀な CSS セレクタの必要性には `evaluate` を使用。
-   デフォルトでは `act` → `wait` を避ける；例外的な場合（待機する信頼できる UI 状態がない）のみ使用。
-   `upload` は、アーミング後に自動クリックするためにオプションで `ref` を渡せます。
-   `upload` は `inputRef`（aria ref）または `element`（CSS セレクタ）もサポートし、`` を直接設定します。

### canvas

ノード Canvas を駆動します（表示、評価、スナップショット、A2UI）。コアアクション:

-   `present`, `hide`, `navigate`, `eval`
-   `snapshot` (画像ブロック + `MEDIA:` を返す)
-   `a2ui_push`, `a2ui_reset`

注意点:

-   内部的にゲートウェイ `node.invoke` を使用。
-   `node` が指定されていない場合、ツールはデフォルトを選択します（単一接続ノードまたはローカル mac ノード）。
-   A2UI は v0.8 のみ（`createSurface` なし）；CLI は v0.9 JSONL をラインエラーで拒否します。
-   簡単なテスト: `openclaw nodes canvas a2ui push --node  --text "Hello from A2UI"`。

### nodes

ペアリングされたノードを発見してターゲットにします；通知を送信；カメラ/画面をキャプチャ。コアアクション:

-   `status`, `describe`
-   `pending`, `approve`, `reject` (ペアリング)
-   `notify` (macOS `system.notify`)
-   `run` (macOS `system.run`)
-   `camera_list`, `camera_snap`, `camera_clip`, `screen_record`
-   `location_get`, `notifications_list`, `notifications_action`
-   `device_status`, `device_info`, `device_permissions`, `device_health`

注意点:

-   カメラ/画面コマンドは、ノードアプリがフォアグラウンド化されている必要があります。
-   画像は画像ブロック + `MEDIA:` を返します。
-   動画は `FILE:` (mp4) を返します。
-   位置情報は JSON ペイロード（緯度/経度/精度/タイムスタンプ）を返します。
-   `run` パラメータ: `command` argv 配列；オプション `cwd`, `env` (`KEY=VAL`), `commandTimeoutMs`, `invokeTimeoutMs`, `needsScreenRecording`。

例 (`run`):

```json
{
  "action": "run",
  "node": "office-mac",
  "command": ["echo", "Hello"],
  "env": ["FOO=bar"],
  "commandTimeoutMs": 12000,
  "invokeTimeoutMs": 45000,
  "needsScreenRecording": false
}
```

### image

設定された画像モデルで画像を分析します。コアパラメータ:

-   `image` (必須パスまたは URL)
-   `prompt` (オプション; デフォルトは "画像を説明してください。")
-   `model` (オプションのオーバーライド)
-   `maxBytesMb` (オプションのサイズ制限)

注意点:

-   `agents.defaults.imageModel` が設定されている場合（プライマリまたはフォールバック）、またはデフォルトモデル + 設定された認証から暗黙的な画像モデルが推測できる場合（ベストエフォートペアリング）にのみ利用可能。
-   画像モデルを直接使用します（メインチャットモデルとは独立）。

### pdf

1つ以上の PDF ドキュメントを分析します。完全な動作、制限、設定、例については [PDF ツール](./tools/pdf.md) を参照。

### message

Discord/Google Chat/Slack/Telegram/WhatsApp/Signal/iMessage/MS Teams を横断してメッセージとチャネルアクションを送信します。コアアクション:

-   `send` (テキスト + オプションのメディア; MS Teams は Adaptive Cards 用に `card` もサポート)
-   `poll` (WhatsApp/Discord/MS Teams の投票)
-   `react` / `reactions` / `read` / `edit` / `delete`
-   `pin` / `unpin` / `list-pins`
-   `permissions`
-   `thread-create` / `thread-list` / `thread-reply`
-   `search`
-   `sticker`
-   `member-info` / `role-info`
-   `emoji-list` / `emoji-upload` / `sticker-upload`
-   `role-add` / `role-remove`
-   `channel-info` / `channel-list`
-   `voice-status`
-   `event-list` / `event-create`
-   `timeout` / `kick` / `ban`

注意点:

-   `send` は WhatsApp をゲートウェイ経由でルーティング；他のチャネルは直接。
-   `poll` は WhatsApp と MS Teams にゲートウェイを使用；Discord の投票は直接。
-   メッセージツール呼び出しがアクティブなチャットセッションにバインドされている場合、送信はそのセッションのターゲットに制限され、コンテキスト間の漏洩を防ぎます。

### cron

ゲートウェイ cron ジョブとウェイクアップを管理します。コアアクション:

-   `status`, `list`
-   `add`, `update`, `remove`, `run`, `runs`
-   `wake` (システムイベントをエンキュー + オプションの即時ハートビート)

注意点:

-   `add` は完全な cron ジョブオブジェクトを期待します（`cron.add` RPC と同じスキーマ）。
-   `update` は `{ jobId, patch }` を使用します（互換性のために `id` も受け入れます）。

### gateway

実行中のゲートウェイプロセスを再起動または更新を適用します（インプレース）。コアアクション:

-   `restart` (承認 + インプロセス再起動のために `SIGUSR1` を送信；`openclaw gateway` インプレース再起動)
-   `config.schema.lookup` (プロンプトコンテキストに完全なスキーマをロードせずに、一度に1つの設定パスを検査)
-   `config.get`
-   `config.apply` (検証 + 設定書き込み + 再起動 + ウェイク)
-   `config.patch` (部分更新をマージ + 再起動 + ウェイク)
-   `update.run` (更新実行 + 再起動 + ウェイク)

注意点:

-   `config.schema.lookup` は `gateway.auth` や `agents.list.*.heartbeat` などのターゲット設定パスを期待します。
-   パスには、`plugins.entries.` を扱う場合、スラッシュ区切りのプラグイン ID を含めることができます。例: `plugins.entries.pack/one.config`。
-   `delayMs`（デフォルト 2000）を使用して、進行中の返信を中断しないようにします。
-   `config.schema` は内部の Control UI フローでは引き続き利用可能ですが、エージェント `gateway` ツールを通じては公開されません。
-   `restart` はデフォルトで有効；無効化するには `commands.restart: false` を設定。

### sessions\_list / sessions\_history / sessions\_send / sessions\_spawn / session\_status

セッションを一覧表示、トランスクリプト履歴を検査、または別のセッションに送信します。コアパラメータ:

-   `sessions_list`: `kinds?`, `limit?`, `activeMinutes?`, `messageLimit?` (0 = なし)
-   `sessions_history`: `sessionKey` (または `sessionId`), `limit?`, `includeTools?`
-   `sessions_send`: `sessionKey` (または `sessionId`), `message`, `timeoutSeconds?` (0 = ファイアアンドフォーゲット)
-   `sessions_spawn`: `task`, `label?`, `runtime?`, `agentId?`, `model?`, `thinking?`, `cwd?`, `runTimeoutSeconds?`, `thread?`, `mode?`, `cleanup?`, `sandbox?`, `streamTo?`, `attachments?`, `attachAs?`
-   `session_status`: `sessionKey?` (デフォルト現在; `sessionId` を受け入れ), `model?` (`default` はオーバーライドをクリア)

注意点:

-   `main` は正規の直接チャットキー；global/unknown は非表示。
-   `messageLimit > 0` はセッションごとに最後の N メッセージを取得します（ツールメッセージはフィルタリング）。
-   セッションターゲティングは `tools.sessions.visibility` で制御されます（デフォルト `tree`: 現在のセッション + 生成されたサブエージェントセッション）。複数ユーザーで共有エージェントを実行する場合、セッション間閲覧を防ぐために `tools.sessions.visibility: "self"` の設定を検討してください。
-   `sessions_send` は `timeoutSeconds > 0` の場合、最終完了を待機します。
-   配信/アナウンスは完了後に発生し、ベストエフォートです；`status: "ok"` はエージェント実行が終了したことを確認するものであり、アナウンスが配信されたことを確認するものではありません。
-   `sessions_spawn` は `runtime: "subagent" | "acp"` をサポート（`subagent` デフォルト）。ACP ランタイムの動作については [ACP エージェント](./tools/acp-agents.md) を参照。
-   ACP ランタイムの場合、`streamTo: "parent"` は初期実行の進捗概要を、直接の子配信ではなく、リクエスタセッションにシステムイベントとしてルーティングバックします。
-   `sessions_spawn` はサブエージェント実行を開始し、アナウンス返信をリクエスタチャットに投稿します。
    -   ワンショットモード（`mode: "run"`）と永続的なスレッドバインドモード（`mode: "session"` と `thread: true`）をサポート。
    -   `thread: true` で `mode` が省略された場合、モードはデフォルトで `session`。
    -   `mode: "session"` には `thread: true` が必要。
    -   `runTimeoutSeconds` が省略された場合、OpenClaw は設定されていれば `agents.defaults.subagents.runTimeoutSeconds` を使用；それ以外の場合、タイムアウトはデフォルトで `0`（タイムアウトなし）。
    -   Discord スレッドバインドフローは `session.threadBindings.*` と `channels.discord.threadBindings.*` に依存。
    -   返信形式には `Status`, `Result`, およびコンパクトな統計が含まれます。
    -   `Result` はアシスタント完了テキスト；欠落している場合、最新の `toolResult` がフォールバックとして使用されます。
-   手動完了モードのスポーンは、最初に直接送信し、一時的な失敗に対してキューへのフォールバックと再試行を行います（`status: "ok"` は実行が終了したことを意味し、アナウンスが配信されたことを意味しません）。
-   `sessions_spawn` はサブエージェントランタイムのみのインラインファイル添付をサポート（ACP は拒否）。各添付には `name`, `content`, およびオプションの `encoding` (`utf8` または `base64`) と `mimeType` があります。ファイルは `.openclaw/attachments//` に `.manifest.json` メタデータファイルとともに具体化されます。ツールは `count`, `totalBytes`, ファイルごとの `sha256`, `relDir` を含むレシートを返します。添付コンテンツはトランスクリプト永続化から自動的に編集されます。
    -   制限は `tools.sessions_spawn.attachments` で設定（`enabled`, `maxTotalBytes`, `maxFiles`, `maxFileBytes`, `retainOnSessionKeep`）。
    -   `attachAs.mountPath` は将来のマウント実装のための予約ヒントです。
-   `sessions_spawn` は非ブロッキングで、直ちに `status: "accepted"` を返します。
-   ACP `streamTo: "parent"` 応答には、進捗履歴を追跡するための `streamLogPath`（セッションスコープ `*.acp-stream.jsonl`）が含まれる場合があります。
-   `sessions_send` は返信バックピンポンを実行します（返信 `REPLY_SKIP` で停止；最大ターン数は `session.agentToAgent.maxPingPongTurns`, 0–5）。
-   ピンポンの後、ターゲットエージェントは **アナウンスステップ** を実行します；返信 `ANNOUNCE_SKIP` でアナウンスを抑制。
-   サンドボックス制限: 現在のセッションがサンドボックス化されていて `agents.defaults.sandbox.sessionToolsVisibility: "spawned"` の場合、OpenClaw は `tools.sessions.visibility` を `tree` に制限します。

### agents\_list

現在のセッションが `sessions_spawn` でターゲットにできるエージェント ID を一覧表示します。注意点:

-   結果はエージェントごとの許可リスト（`agents.list[].subagents.allowAgents`）に制限されます。
-   `["*"]` が設定されている場合、ツールはすべての設定済みエージェントを含め、`allowAny: true` をマークします。

## パラメータ (共通)

ゲートウェイバックドツール (`canvas`, `nodes`, `cron`):

-   `gatewayUrl` (デフォルト `ws://127.0.0.1:18789`)
-   `gatewayToken` (認証が有効な場合)
-   `timeoutMs`

注意: `gatewayUrl` が設定されている場合、`gatewayToken` を明示的に含めてください。ツールはオーバーライドのために設定または環境認証情報を継承せず、明示的な認証情報の欠落はエラーです。ブラウザツール:

-   `profile` (オプション; デフォルトは `browser.defaultProfile`)
-   `target` (`sandbox` | `host` | `node`)
-   `node` (オプション; 特定のノード ID/名前を固定)

## 推奨エージェントフロー

ブラウザ自動化:

1.  `browser` → `status` / `start`
2.  `snapshot` (ai または aria)
3.  `act` (click/type/press)
4.  視覚的確認が必要な場合は `screenshot`

キャンバスレンダリング:

1.  `canvas` → `present`
2.  `a2ui_push` (オプション)
3.  `snapshot`

ノードターゲティング:

1.  `nodes` → `status`
2.  選択したノードで `describe`
3.  `notify` / `run` / `camera_snap` / `screen_record`

## 安全性

-   直接の `system.run` は避ける；明示的なユーザー同意がある場合のみ `nodes` → `run` を使用。
-   カメラ/画面キャプチャのユーザー同意を尊重。
-   メディアコマンドを呼び出す前に、`status/describe` を使用して権限を確認。

## ツールがエージェントに提示される方法

ツールは2つの並列チャネルで公開されます:

1.  **システムプロンプトテキスト**: 人間が読めるリスト + ガイダンス。
2.  **ツールスキーマ**: モデル API に送信される構造化関数定義。

つまり、エージェントは「どのツールが存在するか」と「それらを呼び出す方法」の両方を認識します。ツールがシステムプロンプトまたはスキーマに表示されない場合、モデルはそれを呼び出せません。

[apply\_patch ツール](./tools/apply-patch.md)