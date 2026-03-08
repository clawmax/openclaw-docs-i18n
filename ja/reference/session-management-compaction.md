

  コンパクション内部

  
# セッション管理詳細解説

このドキュメントでは、OpenClawがセッションをエンドツーエンドで管理する方法を説明します：

-   **セッションルーティング** (着信メッセージが `sessionKey` にマッピングされる仕組み)
-   **セッションストア** (`sessions.json`) とその追跡内容
-   **トランスクリプト永続化** (`*.jsonl`) とその構造
-   **トランスクリプトの整理** (実行前のプロバイダー固有の修正)
-   **コンテキスト制限** (コンテキストウィンドウ vs 追跡トークン)
-   **コンパクション** (手動 + 自動コンパクション) と事前コンパクション作業をフックする場所
-   **サイレントハウスキーピング** (ユーザーに表示される出力を生成すべきでないメモリ書き込みなど)

より高レベルの概要を最初に知りたい場合は、以下から始めてください：

-   [/concepts/session](../concepts/session.md)
-   [/concepts/compaction](../concepts/compaction.md)
-   [/concepts/session-pruning](../concepts/session-pruning.md)
-   [/reference/transcript-hygiene](./transcript-hygiene.md)

* * *

## 信頼できる情報源: ゲートウェイ

OpenClawは、セッション状態を所有する単一の**ゲートウェイプロセス**を中心に設計されています。

-   UI (macOSアプリ、WebコントロールUI、TUI) は、セッションリストとトークン数を取得するためにゲートウェイにクエリを実行する必要があります。
-   リモートモードでは、セッションファイルはリモートホスト上にあります。「ローカルのMacファイルを確認する」ことは、ゲートウェイが使用しているものを反映しません。

* * *

## 2つの永続化レイヤー

OpenClawはセッションを2つのレイヤーで永続化します：

1.  **セッションストア (`sessions.json`)**
    -   キー/値マップ: `sessionKey -> SessionEntry`
    -   小さく、可変的、編集（またはエントリ削除）が安全
    -   セッションメタデータを追跡 (現在のセッションID、最終アクティビティ、トグル、トークンカウンターなど)
2.  **トランスクリプト (`.jsonl`)**
    -   ツリー構造を持つ追記専用のトランスクリプト (エントリは `id` + `parentId` を持つ)
    -   実際の会話 + ツール呼び出し + コンパクション要約を保存
    -   将来のターンのためにモデルコンテキストを再構築するために使用

* * *

## ディスク上の場所

エージェントごと、ゲートウェイホスト上：

-   ストア: `~/.openclaw/agents//sessions/sessions.json`
-   トランスクリプト: `~/.openclaw/agents//sessions/.jsonl`
    -   Telegramトピックセッション: `.../-topic-.jsonl`

OpenClawはこれらを `src/config/sessions.ts` 経由で解決します。

* * *

## ストアのメンテナンスとディスク制御

セッション永続化には、`sessions.json` とトランスクリプトアーティファクトのための自動メンテナンス制御 (`session.maintenance`) があります：

-   `mode`: `warn` (デフォルト) または `enforce`
-   `pruneAfter`: 古いエントリのカットオフ期間 (デフォルト `30d`)
-   `maxEntries`: `sessions.json` 内のエントリ数の上限 (デフォルト `500`)
-   `rotateBytes`: `sessions.json` が大きすぎる場合にローテート (デフォルト `10mb`)
-   `resetArchiveRetention`: `*.reset.` トランスクリプトアーカイブの保持期間 (デフォルト: `pruneAfter` と同じ; `false` でクリーンアップ無効化)
-   `maxDiskBytes`: オプションのセッションディレクトリ予算
-   `highWaterBytes`: クリーンアップ後のオプションの目標値 (デフォルト `maxDiskBytes` の `80%`)

ディスク予算クリーンアップの実施順序 (`mode: "enforce"`)：

1.  まず最も古いアーカイブ済みまたは孤立したトランスクリプトアーティファクトを削除。
2.  それでも目標値を超える場合、最も古いセッションエントリとそのトランスクリプトファイルを削除。
3.  使用量が `highWaterBytes` 以下になるまで続行。

`mode: "warn"` では、OpenClawは潜在的な削除を報告しますが、ストア/ファイルを変更しません。必要に応じてメンテナンスを実行：

```bash
openclaw sessions cleanup --dry-run
openclaw sessions cleanup --enforce
```

* * *

## Cronセッションと実行ログ

分離されたcron実行もセッションエントリ/トランスクリプトを作成し、専用の保持制御があります：

-   `cron.sessionRetention` (デフォルト `24h`) は、古い分離cron実行セッションをセッションストアから削除します (`false` で無効化)。
-   `cron.runLog.maxBytes` + `cron.runLog.keepLines` は `~/.openclaw/cron/runs/.jsonl` ファイルを削除します (デフォルト: `2_000_000` バイトと `2000` 行)。

* * *

## セッションキー (sessionKey)

`sessionKey` は、*どの会話バケット*にいるかを識別します (ルーティング + 分離)。一般的なパターン：

-   メイン/直接チャット (エージェントごと): `agent::` (デフォルト `main`)
-   グループ: `agent:::group:`
-   ルーム/チャンネル (Discord/Slack): `agent:::channel:` または `...:room:`
-   Cron: `cron:<job.id>`
-   Webhook: `hook:` (上書きされない限り)

正規のルールは [/concepts/session](../concepts/session.md) に文書化されています。

* * *

## セッションID (sessionId)

各 `sessionKey` は、現在の `sessionId` (会話を継続するトランスクリプトファイル) を指します。経験則：

-   **リセット** (`/new`, `/reset`) は、その `sessionKey` に対して新しい `sessionId` を作成します。
-   **日次リセット** (デフォルトはゲートウェイホストの現地時間で午前4:00) は、リセット境界後の次のメッセージで新しい `sessionId` を作成します。
-   **アイドル期限切れ** (`session.reset.idleMinutes` またはレガシー `session.idleMinutes`) は、アイドルウィンドウ後にメッセージが到着したときに新しい `sessionId` を作成します。日次とアイドルの両方が設定されている場合、どちらか早く期限切れになる方が優先されます。
-   **スレッド親フォークガード** (`session.parentForkMaxTokens`, デフォルト `100000`) は、親セッションが既に大きすぎる場合、親トランスクリプトのフォークをスキップします。新しいスレッドは新規に開始します。`0` に設定すると無効化されます。

実装詳細: 決定は `src/auto-reply/reply/session.ts` の `initSessionState()` で行われます。

* * *

## セッションストアスキーマ (sessions.json)

ストアの値の型は `src/config/sessions.ts` の `SessionEntry` です。主要なフィールド (網羅的ではありません)：

-   `sessionId`: 現在のトランスクリプトID (`sessionFile` が設定されていない限り、ファイル名はこれから派生)
-   `updatedAt`: 最終アクティビティタイムスタンプ
-   `sessionFile`: オプションの明示的なトランスクリプトパス上書き
-   `chatType`: `direct | group | room` (UIと送信ポリシーに役立つ)
-   `provider`, `subject`, `room`, `space`, `displayName`: グループ/チャンネルラベリングのためのメタデータ
-   トグル:
    -   `thinkingLevel`, `verboseLevel`, `reasoningLevel`, `elevatedLevel`
    -   `sendPolicy` (セッションごとの上書き)
-   モデル選択:
    -   `providerOverride`, `modelOverride`, `authProfileOverride`
-   トークンカウンター (ベストエフォート / プロバイダー依存):
    -   `inputTokens`, `outputTokens`, `totalTokens`, `contextTokens`
-   `compactionCount`: このセッションキーに対して自動コンパクションが完了した回数
-   `memoryFlushAt`: 最後の事前コンパクションメモリフラッシュのタイムスタンプ
-   `memoryFlushCompactionCount`: 最後のフラッシュが実行されたときのコンパクション回数

ストアは編集しても安全ですが、ゲートウェイが権限を持ちます：セッション実行時にエントリを書き換えたり再ハイドレートしたりする可能性があります。

* * *

## トランスクリプト構造 (\*.jsonl)

トランスクリプトは `@mariozechner/pi-coding-agent` の `SessionManager` によって管理されます。ファイルはJSONL形式です：

-   最初の行: セッションヘッダー (`type: "session"`, `id`, `cwd`, `timestamp`, オプション `parentSession` を含む)
-   その後: `id` + `parentId` を持つセッションエントリ (ツリー)

注目すべきエントリタイプ：

-   `message`: ユーザー/アシスタント/toolResultメッセージ
-   `custom_message`: モデルコンテキストに入る拡張機能注入メッセージ (UIから非表示にできる)
-   `custom`: モデルコンテキストに入らない拡張機能状態
-   `compaction`: `firstKeptEntryId` と `tokensBefore` を持つ永続化されたコンパクション要約
-   `branch_summary`: ツリーブランチをナビゲートする際の永続化された要約

OpenClawは意図的にトランスクリプトを「修正」しません。ゲートウェイは `SessionManager` を使用してトランスクリプトを読み書きします。

* * *

## コンテキストウィンドウ vs 追跡トークン

2つの異なる概念が重要です：

1.  **モデルコンテキストウィンドウ**: モデルごとのハードキャップ (モデルから見えるトークン)
2.  **セッションストアカウンター**: `sessions.json` に書き込まれるローリング統計 (/status とダッシュボードで使用)

制限を調整する場合：

-   コンテキストウィンドウはモデルカタログから来ます (設定で上書き可能)。
-   ストア内の `contextTokens` は実行時の推定/報告値です。厳密な保証として扱わないでください。

詳細は [/token-use](./token-use.md) を参照してください。

* * *

## コンパクション: その概要

コンパクションは、古い会話をトランスクリプト内の永続化された `compaction` エントリに要約し、最近のメッセージはそのまま保持します。コンパクション後、将来のターンでは以下が見えます：

-   コンパクション要約
-   `firstKeptEntryId` 以降のメッセージ

コンパクションは**永続的**です (セッションプルーニングとは異なります)。 [/concepts/session-pruning](../concepts/session-pruning.md) を参照してください。

* * *

## 自動コンパクションが発生するタイミング (Piランタイム)

組み込みPiエージェントでは、自動コンパクションは2つのケースでトリガーされます：

1.  **オーバーフロー回復**: モデルがコンテキストオーバーフローエラーを返す → コンパクト → 再試行。
2.  **しきい値メンテナンス**: 成功したターンの後、以下の場合：

`contextTokens > contextWindow - reserveTokens` ここで：

-   `contextWindow` はモデルのコンテキストウィンドウ
-   `reserveTokens` はプロンプト + 次のモデル出力のために確保されたヘッドルーム

これらはPiランタイムのセマンティクスです (OpenClawはイベントを消費しますが、いつコンパクトするかはPiが決定します)。

* * *

## コンパクション設定 (reserveTokens, keepRecentTokens)

Piのコンパクション設定はPi設定内にあります：

```json
{
  compaction: {
    enabled: true,
    reserveTokens: 16384,
    keepRecentTokens: 20000,
  },
}
```

OpenClawは組み込み実行のための安全な下限も強制します：

-   `compaction.reserveTokens < reserveTokensFloor` の場合、OpenClawはそれを引き上げます。
-   デフォルトの下限は `20000` トークンです。
-   `agents.defaults.compaction.reserveTokensFloor: 0` を設定すると下限を無効化します。
-   既に高い場合は、OpenClawはそのままにします。

理由：コンパクションが避けられなくなる前に、マルチターンの「ハウスキーピング」(メモリ書き込みなど) のための十分なヘッドルームを残すため。実装：`src/agents/pi-settings.ts` の `ensurePiCompactionReserveTokens()` (`src/agents/pi-embedded-runner.ts` から呼び出されます)。

* * *

## ユーザーに見える部分

コンパクションとセッション状態は以下で観察できます：

-   `/status` (任意のチャットセッション内)
-   `openclaw status` (CLI)
-   `openclaw sessions` / `sessions --json`
-   詳細モード: `🧹 Auto-compaction complete` + コンパクション回数

* * *

## サイレントハウスキーピング (NO_REPLY)

OpenClawは、ユーザーが中間出力を見るべきではないバックグラウンドタスクのための「サイレント」ターンをサポートします。慣例：

-   アシスタントはその出力を `NO_REPLY` で開始し、「ユーザーに返信を配信しない」ことを示します。
-   OpenClawは配信レイヤーでこれを除去/抑制します。

`2026.1.10` 時点で、OpenClawは部分チャンクが `NO_REPLY` で始まる場合の**ドラフト/タイピングストリーミング**も抑制するため、サイレント操作中に部分出力が漏れることはありません。

* * *

## 事前コンパクション「メモリフラッシュ」 (実装済み)

目標：自動コンパクションが発生する前に、耐久性のある状態をディスクに書き込む (例：エージェントワークスペース内の `memory/YYYY-MM-DD.md`) サイレントなエージェントターンを実行し、コンパクションが重要なコンテキストを消去できないようにする。OpenClawは**事前しきい値フラッシュ**アプローチを使用します：

1.  セッションコンテキスト使用量を監視。
2.  「ソフトしきい値」 (Piのコンパクションしきい値より下) を超えたら、エージェントへのサイレントな「今すぐメモリを書き込む」指示を実行。
3.  `NO_REPLY` を使用してユーザーには何も見せない。

設定 (`agents.defaults.compaction.memoryFlush`)：

-   `enabled` (デフォルト: `true`)
-   `softThresholdTokens` (デフォルト: `4000`)
-   `prompt` (フラッシュターンのユーザーメッセージ)
-   `systemPrompt` (フラッシュターンに追加される追加システムプロンプト)

注意点：

-   デフォルトのプロンプト/システムプロンプトには、配信を抑制するための `NO_REPLY` ヒントが含まれています。
-   フラッシュはコンパクションサイクルごとに1回実行されます (`sessions.json` で追跡)。
-   フラッシュは組み込みPiセッションに対してのみ実行されます (CLIバックエンドはスキップ)。
-   セッションワークスペースが読み取り専用 (`workspaceAccess: "ro"` または `"none"`) の場合、フラッシュはスキップされます。
-   ワークスペースファイルレイアウトと書き込みパターンについては [Memory](../concepts/memory.md) を参照してください。

Piは拡張APIに `session_before_compact` フックも公開していますが、OpenClawのフラッシュロジックは現在ゲートウェイ側にあります。

* * *

## トラブルシューティングチェックリスト

-   セッションキーが間違っている？ [/concepts/session](../concepts/session.md) から始めて、`/status` の `sessionKey` を確認してください。
-   ストアとトランスクリプトが一致しない？ `openclaw status` からゲートウェイホストとストアパスを確認してください。
-   コンパクションが頻繁に発生する？ 以下を確認：
    -   モデルコンテキストウィンドウ (小さすぎる)
    -   コンパクション設定 (`reserveTokens` がモデルウィンドウに対して高すぎると、早期コンパクションが発生する可能性あり)
    -   ツール結果の肥大化：セッションプルーニングを有効化/調整
-   サイレントターンが漏れている？ 返信が `NO_REPLY` (正確なトークン) で始まっていること、およびストリーミング抑制修正を含むビルドを使用していることを確認してください。

[Node.js](../install/node.md)[セットアップ](../start/setup.md)