

  実験

  
# ACPスレッドバウンドエージェント

## 概要

この計画は、OpenClawがスレッド対応チャネル（まずはDiscord）でACPコーディングエージェントを、プロダクションレベルのライフサイクルとリカバリでサポートすべき方法を定義します。関連ドキュメント:

-   [統合ランタイムストリーミングリファクタ計画](./acp-unified-streaming-refactor.md)

目標とするユーザー体験:

-   ユーザーがスレッド内でACPセッションをスポーンまたはフォーカスする
-   そのスレッド内のユーザーメッセージは、バインドされたACPセッションにルーティングされる
-   エージェントの出力は同じスレッドのペルソナにストリーミングで戻る
-   セッションは永続的またはワンショットで、明示的なクリーンアップ制御が可能

## 決定サマリー

長期的な推奨はハイブリッドアーキテクチャです:

-   OpenClawコアはACPコントロールプレーンの関心事を所有する
    -   セッションIDとメタデータ
    -   スレッドバインディングとルーティング決定
    -   配信不変条件と重複抑制
    -   ライフサイクルクリーンアップとリカバリセマンティクス
-   ACPランタイムバックエンドはプラグイン可能
    -   最初のバックエンドはacpxベースのプラグインサービス
    -   ランタイムはACPトランスポート、キューイング、キャンセル、再接続を処理

OpenClawはACPトランスポートの内部実装をコアで再実装すべきではありません。OpenClawはルーティングのための純粋なプラグインのみのインターセプションパスに依存すべきではありません。

## 北極星アーキテクチャ（理想形）

ACPをOpenClaw内の第一級コントロールプレーンとして扱い、プラグイン可能なランタイムアダプターを持つものとします。交渉の余地のない不変条件:

-   すべてのACPスレッドバインディングは有効なACPセッションレコードを参照する
-   すべてのACPセッションは明示的なライフサイクル状態を持つ (`creating`, `idle`, `running`, `cancelling`, `closed`, `error`)
-   すべてのACP実行は明示的な実行状態を持つ (`queued`, `running`, `completed`, `failed`, `cancelled`)
-   スポーン、バインド、および初期エンキューはアトミック
-   コマンドリトライはべき等（重複実行や重複Discord出力なし）
-   バインドされたスレッドチャネル出力はACP実行イベントの投影であり、アドホックな副作用ではない

長期的所有権モデル:

-   `AcpSessionManager` は唯一のACPライターおよびオーケストレーター
-   マネージャーは最初にゲートウェイプロセス内に存在；同じインターフェースの背後で専用サイドカーに後で移動可能
-   ACPセッションキーごとに、マネージャーは1つのインメモリアクターを所有（シリアライズされたコマンド実行）
-   アダプター (`acpx`, 将来のバックエンド) はトランスポート/ランタイム実装のみ

長期的永続化モデル:

-   ACPコントロールプレーン状態を専用SQLiteストア（WALモード）に移動し、OpenClaw状態ディレクトリ下に配置
-   移行中は互換性投影として `SessionEntry.acp` を保持し、信頼できる情報源としない
-   ACPイベントを追記のみで保存し、リプレイ、クラッシュリカバリ、決定論的配信をサポート

### 配信戦略（理想形への架け橋）

-   短期架け橋
    -   現在のスレッドバインディングメカニズムと既存のACP設定表面を維持
    -   メタデータギャップのバグを修正し、ACPターンを単一のコアACPブランチ経由でルーティング
    -   べき等キーとフェイルクローズルーティングチェックを直ちに追加
-   長期切り替え
    -   ACPの信頼できる情報源をコントロールプレーンDB + アクターに移動
    -   バインドされたスレッド配信を純粋にイベント投影ベースにする
    -   日和見的なセッションエントリメタデータに依存するレガシーフォールバック動作を削除

## 純粋プラグインのみがダメな理由

現在のプラグインフックは、コア変更なしではエンドツーエンドACPセッションルーティングには不十分です。

-   スレッドバインディングからのインバウンドルーティングは、コアディスパッチで最初にセッションキーに解決される
-   メッセージフックはファイアアンドフォーゲットであり、メイン返信パスを短絡できない
-   プラグインコマンドは制御操作には適しているが、コアのターンごとのディスパッチフローを置き換えるには不十分

結果:

-   ACPランタイムはプラグイン化可能
-   ACPルーティングブランチはコアに存在しなければならない

## 再利用する既存の基盤

既に実装済みで正規として残すべきもの:

-   スレッドバインディングターゲットは `subagent` と `acp` をサポート
-   インバウンドスレッドルーティングオーバーライドは、通常のディスパッチ前にバインディングで解決
-   返信配信でのウェブフック経由のアウトバウンドスレッドID
-   `/focus` と `/unfocus` フローとACPターゲット互換性
-   起動時の復元を伴う永続的バインディングストア
-   アーカイブ、削除、フォーカス解除、リセット、削除時のアンバインドライフサイクル

この計画はその基盤を置き換えるのではなく拡張します。

## アーキテクチャ

### 境界モデル

コア（OpenClawコア内に存在しなければならない）:

-   返信パイプライン内のACPセッションモードディスパッチブランチ
-   親とスレッドの重複を避けるための配信調停
-   ACPコントロールプレーン永続化（移行中は `SessionEntry.acp` 互換性投影付き）
-   セッションリセット/削除に紐づくライフサイクルアンバインドとランタイムデタッチセマンティクス

プラグインバックエンド（acpx実装）:

-   ACPランタイムワーカー監視
-   acpxプロセス呼び出しとイベント解析
-   ACPコマンドハンドラー (`/acp ...`) とオペレーターUX
-   バックエンド固有の設定デフォルトと診断

### ランタイム所有権モデル

-   1つのゲートウェイプロセスがACPオーケストレーション状態を所有
-   ACP実行はacpxバックエンド経由で監視された子プロセスで実行
-   プロセス戦略はメッセージごとではなく、アクティブなACPセッションキーごとに長寿命

これにより、すべてのプロンプトでの起動コストを回避し、キャンセルと再接続セマンティクスを確実に保ちます。

### コアランタイム契約

コアACPランタイム契約を追加し、ルーティングコードがCLIの詳細に依存せず、ディスパッチロジックを変更せずにバックエンドを切り替えられるようにします:

```bash
export type AcpRuntimePromptMode = "prompt" | "steer";

export type AcpRuntimeHandle = {
  sessionKey: string;
  backend: string;
  runtimeSessionName: string;
};

export type AcpRuntimeEvent =
  | { type: "text_delta"; stream: "output" | "thought"; text: string }
  | { type: "tool_call"; name: string; argumentsText: string }
  | { type: "done"; usage?: Record<string, number> }
  | { type: "error"; code: string; message: string; retryable?: boolean };

export interface AcpRuntime {
  ensureSession(input: {
    sessionKey: string;
    agent: string;
    mode: "persistent" | "oneshot";
    cwd?: string;
    env?: Record<string, string>;
    idempotencyKey: string;
  }): Promise<AcpRuntimeHandle>;

  submit(input: {
    handle: AcpRuntimeHandle;
    text: string;
    mode: AcpRuntimePromptMode;
    idempotencyKey: string;
  }): Promise<{ runtimeRunId: string }>;

  stream(input: {
    handle: AcpRuntimeHandle;
    runtimeRunId: string;
    onEvent: (event: AcpRuntimeEvent) => Promise<void> | void;
    signal?: AbortSignal;
  }): Promise<void>;

  cancel(input: {
    handle: AcpRuntimeHandle;
    runtimeRunId?: string;
    reason?: string;
    idempotencyKey: string;
  }): Promise<void>;

  close(input: { handle: AcpRuntimeHandle; reason: string; idempotencyKey: string }): Promise<void>;

  health?(): Promise<{ ok: boolean; details?: string }>;
}
```

実装詳細:

-   最初のバックエンド: プラグインサービスとして出荷される `AcpxRuntime`
-   コアはレジストリ経由でランタイムを解決し、ACPランタイムバックエンドが利用できない場合は明示的なオペレーターエラーで失敗

### コントロールプレーンデータモデルと永続化

長期的な信頼できる情報源は専用ACP SQLiteデータベース（WALモード）であり、トランザクション更新とクラッシュセーフなリカバリのため:

-   `acp_sessions`
    -   `session_key` (pk), `backend`, `agent`, `mode`, `cwd`, `state`, `created_at`, `updated_at`, `last_error`
-   `acp_runs`
    -   `run_id` (pk), `session_key` (fk), `state`, `requester_message_id`, `idempotency_key`, `started_at`, `ended_at`, `error_code`, `error_message`
-   `acp_bindings`
    -   `binding_key` (pk), `thread_id`, `channel_id`, `account_id`, `session_key` (fk), `expires_at`, `bound_at`
-   `acp_events`
    -   `event_id` (pk), `run_id` (fk), `seq`, `kind`, `payload_json`, `created_at`
-   `acp_delivery_checkpoint`
    -   `run_id` (pk/fk), `last_event_seq`, `last_discord_message_id`, `updated_at`
-   `acp_idempotency`
    -   `scope`, `idempotency_key`, `result_json`, `created_at`, unique `(scope, idempotency_key)`

```bash
export type AcpSessionMeta = {
  backend: string;
  agent: string;
  runtimeSessionName: string;
  mode: "persistent" | "oneshot";
  cwd?: string;
  state: "idle" | "running" | "error";
  lastActivityAt: number;
  lastError?: string;
};
```

保存ルール:

-   移行中は互換性投影として `SessionEntry.acp` を保持
-   プロセスIDとソケットはメモリ内のみに保持
-   耐久性のあるライフサイクルと実行ステータスは汎用セッションJSONではなくACP DBに保存
-   ランタイム所有者が死亡した場合、ゲートウェイはACP DBから再ハイドレートし、チェックポイントから再開

### ルーティングと配信

インバウンド:

-   現在のスレッドバインディング検索を最初のルーティングステップとして維持
-   バインドされたターゲットがACPセッションの場合、`getReplyFromConfig` の代わりにACPランタイムブランチにルーティング
-   明示的な `/acp steer` コマンドは `mode: "steer"` を使用

アウトバウンド:

-   ACPイベントストリームはOpenClaw返信チャンクに正規化される
-   配信ターゲットは既存のバインドされた宛先パス経由で解決される
-   そのセッションターンに対してアクティブなバインドされたスレッドがある場合、親チャネルの完了は抑制される

ストリーミングポリシー:

-   結合ウィンドウ付きで部分出力をストリーミング
-   Discordレート制限内に収まるように設定可能な最小間隔と最大チャンクバイト数
-   完了または失敗時に最終メッセージを常に出力

### 状態機械とトランザクション境界

セッション状態機械:

-   `creating -> idle -> running -> idle`
-   `running -> cancelling -> idle | error`
-   `idle -> closed`
-   `error -> idle | closed`

実行状態機械:

-   `queued -> running -> completed`
-   `running -> failed | cancelled`
-   `queued -> cancelled`

必要なトランザクション境界:

-   スポーントランザクション
    -   ACPセッション行を作成
    -   ACPスレッドバインディング行を作成/更新
    -   初期実行行をエンキュー
-   クローズトランザクション
    -   セッションをクローズ済みとしてマーク
    -   バインディング行を削除/期限切れにする
    -   最終クローズイベントを書き込み
-   キャンセルトランザクション
    -   ターゲット実行をべき等キー付きでキャンセリング/キャンセル済みとしてマーク

これらの境界を跨いだ部分的な成功は許可されません。

### セッションごとのアクターモデル

`AcpSessionManager` はACPセッションキーごとに1つのアクターを実行:

-   アクターメールボックスは `submit`, `cancel`, `close`, `stream` 副作用をシリアライズ
-   アクターはランタイムハンドルのハイドレーションと、そのセッションのランタイムアダプタープロセスライフサイクルを所有
-   アクターはDiscord配信前に実行イベントを順序通り (`seq`) に書き込む
-   アクターは成功したアウトバウンド送信後に配信チェックポイントを更新

これによりターン間の競合が除去され、重複または順序外のスレッド出力が防止されます。

### べき等性と配信投影

すべての外部ACPアクションはべき等キーを持たなければならない:

-   スポーンべき等キー
-   プロンプト/ステアべき等キー
-   キャンセルべき等キー
-   クローズべき等キー

配信ルール:

-   Discordメッセージは `acp_events` と `acp_delivery_checkpoint` から導出される
-   リトライはチェックポイントから再開し、既に配信されたチャンクを再送信しない
-   最終返信出力は投影ロジックから実行ごとに厳密に一度

### リカバリと自己修復

ゲートウェイ起動時:

-   非終了ACPセッション (`creating`, `idle`, `running`, `cancelling`, `error`) をロード
-   最初のインバウンドイベント時に遅延的に、または設定された上限下で積極的にアクターを再作成
-   ハートビートが欠落している `running` 実行を調整し、`failed` としてマークするかアダプター経由でリカバリ

インバウンドDiscordスレッドメッセージ時:

-   バインディングが存在するがACPセッションが欠落している場合、明示的な古いバインディングメッセージでフェイルクローズ
-   オプションでオペレーター安全な検証後に古いバインディングを自動アンバインド
-   古いACPバインディングを通常のLLMパスに黙ってルーティングしない

### ライフサイクルと安全性

サポートされる操作:

-   現在の実行をキャンセル: `/acp cancel`
-   スレッドをアンバインド: `/unfocus`
-   ACPセッションをクローズ: `/acp close`
-   アイドルセッションを実効TTLで自動クローズ

TTLポリシー:

-   実効TTLは以下の最小値
    -   グローバル/セッションTTL
    -   DiscordスレッドバインディングTTL
    -   ACPランタイム所有者TTL

安全性制御:

-   名前によるACPエージェントの許可リスト
-   ACPセッションのワークスペースルート制限
-   環境変数許可リストパススルー
-   アカウントごとおよびグローバルな最大同時ACPセッション数
-   ランタイムクラッシュのための制限付き再起動バックオフ

## 設定表面

コアキー:

-   `acp.enabled`
-   `acp.dispatch.enabled` (独立したACPルーティング停止スイッチ)
-   `acp.backend` (デフォルト `acpx`)
-   `acp.defaultAgent`
-   `acp.allowedAgents[]`
-   `acp.maxConcurrentSessions`
-   `acp.stream.coalesceIdleMs`
-   `acp.stream.maxChunkChars`
-   `acp.runtime.ttlMinutes`
-   `acp.controlPlane.store` (`sqlite` デフォルト)
-   `acp.controlPlane.storePath`
-   `acp.controlPlane.recovery.eagerActors`
-   `acp.controlPlane.recovery.reconcileRunningAfterMs`
-   `acp.controlPlane.checkpoint.flushEveryEvents`
-   `acp.controlPlane.checkpoint.flushEveryMs`
-   `acp.idempotency.ttlHours`
-   `channels.discord.threadBindings.spawnAcpSessions`

プラグイン/バックエンドキー（acpxプラグインセクション）:

-   バックエンドコマンド/パスオーバーライド
-   バックエンド環境変数許可リスト
-   バックエンドエージェントごとのプリセット
-   バックエンド起動/停止タイムアウト
-   バックエンドセッションごとの最大インフライト実行数

## 実装仕様

### コントロールプレーンモジュール（新規）

コアに専用ACPコントロールプレーンモジュールを追加:

-   `src/acp/control-plane/manager.ts`
    -   ACPアクター、ライフサイクル遷移、コマンドシリアライゼーションを所有
-   `src/acp/control-plane/store.ts`
    -   SQLiteスキーマ管理、トランザクション、クエリヘルパー
-   `src/acp/control-plane/events.ts`
    -   型付きACPイベント定義とシリアライゼーション
-   `src/acp/control-plane/checkpoint.ts`
    -   耐久性のある配信チェックポイントとリプレイカーソル
-   `src/acp/control-plane/idempotency.ts`
    -   べき等キー予約と応答リプレイ
-   `src/acp/control-plane/recovery.ts`
    -   起動時の調整とアクター再ハイドレート計画

互換性ブリッジモジュール:

-   `src/acp/runtime/session-meta.ts`
    -   一時的に `SessionEntry.acp` への投影のために残す
    -   移行切り替え後は信頼できる情報源であることを停止しなければならない

### 必要な不変条件（コードで強制必須）

-   ACPセッション作成とスレッドバインドはアトミック（単一トランザクション）
-   ACPセッションアクターごとに同時にアクティブな実行は最大1つ
-   イベント `seq` は実行ごとに厳密に増加
-   配信チェックポイントは最後にコミットされたイベントを超えて進まない
-   べき等リプレイは重複コマンドキーに対して以前の成功ペイロードを返す
-   古い/欠落したACPメタデータは通常の非ACP返信パスにルーティングできない

### コアタッチポイント

変更するコアファイル:

-   `src/auto-reply/reply/dispatch-from-config.ts`
    -   ACPブランチは `AcpSessionManager.submit` とイベント投影配信を呼び出す
    -   コントロールプレーン不変条件をバイパスする直接ACPフォールバックを削除
-   `src/auto-reply/reply/inbound-context.ts` (または最も近い正規化されたコンテキスト境界)
    -   ACPコントロールプレーンのための正規化されたルーティングキーとべき等シードを公開
-   `src/config/sessions/types.ts`
    -   `SessionEntry.acp` を投影のみの互換性フィールドとして保持
-   `src/gateway/server-methods/sessions.ts`
    -   リセット/削除/アーカイブはACPマネージャークローズ/アンバインドトランザクションパスを呼び出さなければならない
-   `src/infra/outbound/bound-delivery-router.ts`
    -   ACPバインドセッションターンのためのフェイルクローズ宛先動作を強制
-   `src/discord/monitor/thread-bindings.ts`
    -   ACP古いバインディング検証ヘルパーを追加し、コントロールプレーン検索に配線
-   `src/auto-reply/reply/commands-acp.ts`
    -   スポーン/キャンセル/クローズ/ステアをACPマネージャーAPI経由でルーティング
-   `src/agents/acp-spawn.ts`
    -   アドホックなメタデータ書き込みを停止；ACPマネージャースポーントランザクションを呼び出す
-   `src/plugin-sdk/**` およびプラグインランタイムブリッジ
    -   ACPバックエンド登録とヘルスセマンティクスをクリーンに公開

明示的に置き換えないコアファイル:

-   `src/discord/monitor/message-handler.preflight.ts`
    -   スレッドバインディングオーバーライド動作を正規のセッションキー解決器として保持

### ACPランタイムレジストリAPI

コアレジストリモジュールを追加:

-   `src/acp/runtime/registry.ts`

必要なAPI:

```bash
export type AcpRuntimeBackend = {
  id: string;
  runtime: AcpRuntime;
  healthy?: () => boolean;
};

export function registerAcpRuntimeBackend(backend: AcpRuntimeBackend): void;
export function unregisterAcpRuntimeBackend(id: string): void;
export function getAcpRuntimeBackend(id?: string): AcpRuntimeBackend | null;
export function requireAcpRuntimeBackend(id?: string): AcpRuntimeBackend;
```

動作:

-   `requireAcpRuntimeBackend` は利用不可時に型付きACPバックエンド欠落エラーをスロー
-   プラグインサービスは `start` 時にバックエンドを登録し、`stop` 時に登録解除
-   ランタイム検索は読み取り専用でプロセスローカル

### acpxランタイムプラグイン契約（実装詳細）

最初のプロダクションバックエンド (`extensions/acpx`) では、OpenClawとacpxは厳密なコマンド契約で接続される:

-   バックエンドID: `acpx`
-   プラグインサービスID: `acpx-runtime`
-   ランタイムハンドルエンコーディング: `runtimeSessionName = acpx:v1:<base64url(json)>`
-   エンコードされたペイロードフィールド:
    -   `name` (acpx名前付きセッション；OpenClaw `sessionKey` を使用)
    -   `agent` (acpxエージェントコマンド)
    -   `cwd` (セッションワークスペースルート)
    -   `mode` (`persistent | oneshot`)

コマンドマッピング:

-   セッション確保:
    -   `acpx --format json --json-strict --cwd   sessions ensure --name `
-   プロンプトターン:
    -   `acpx --format json --json-strict --cwd   prompt --session  --file -`
-   キャンセル:
    -   `acpx --format json --json-strict --cwd   cancel --session `
-   クローズ:
    -   `acpx --format json --json-strict --cwd   sessions close `

ストリーミング:

-   OpenClawは `acpx --format json --json-strict` からのndjsonイベントを消費
-   `text` => `text_delta/output`
-   `thought` => `text_delta/thought`
-   `tool_call` => `tool_call`
-   `done` => `done`
-   `error` => `error`

### セッションスキーマパッチ

`src/config/sessions/types.ts` の `SessionEntry` をパッチ:

```typescript
type SessionAcpMeta = {
  backend: string;
  agent: string;
  runtimeSessionName: string;
  mode: "persistent" | "oneshot";
  cwd?: string;
  state: "idle" | "running" | "error";
  lastActivityAt: number;
  lastError?: string;
};
```

永続化フィールド:

-   `SessionEntry.acp?: SessionAcpMeta`

移行ルール:

-   フェーズA: 二重書き込み (`acp` 投影 + ACP SQLite信頼できる情報源)
-   フェーズB: ACP SQLiteからプライマリ読み取り、レガシー `SessionEntry.acp` からフォールバック読み取り
-   フェーズC: 移行コマンドが有効なレガシーエントリから欠落ACP行をバックフィル
-   フェーズD: フォールバック読み取りを削除し、UXのみのために投影をオプションとして保持
-   レガシーフィールド (`cliSessionIds`, `claudeCliSessionId`) は変更なしで残す

### エラー契約

安定したACPエラーコードとユーザー向けメッセージを追加:

-   `ACP_BACKEND_MISSING`
    -   メッセージ: `ACPランタイムバックエンドが設定されていません。acpxランタイムプラグインをインストールして有効にしてください。`
-   `ACP_BACKEND_UNAVAILABLE`
    -   メッセージ: `ACPランタイムバックエンドは現在利用できません。しばらくしてから再試行してください。`
-   `ACP_SESSION_INIT_FAILED`
    -   メッセージ: `ACPセッションランタイムを初期化できませんでした。`
-   `ACP_TURN_FAILED`
    -   メッセージ: `ACPターンは完了前に失敗しました。`

ルール:

-   スレッド内で実行可能なユーザー安全なメッセージを返す
-   詳細なバックエンド/システムエラーはランタイムログのみに記録
-   ACPルーティングが明示的に選択された場合、通常のLLMパスに黙ってフォールバックしない

### 重複配信調停

ACPバインドターンのための単一ルーティングルール:

-   ターゲットACPセッションと要求者コンテキストに対してアクティブなスレッドバインディングが存在する場合、そのバインドされたスレッドにのみ配信
-   同じターンに対して親チャネルにも送信しない
-   バインドされた宛先選択があいまいな場合、明示的なエラーでフェイルクローズ（暗黙の親フォールバックなし）
-   アクティブなバインディングが存在しない場合、通常のセッション宛先動作を使用

### 可観測性と運用準備

必要なメトリクス:

-   バックエンドとエラーコード別ACPスポーン成功/失敗数
-   ACP実行レイテンシ百分位数（キュー待機、ランタイムターン時間、配信投影時間）
-   ACPアクター再起動数と再起動理由
-   古いバインディング検出数
-   べき等リプレイヒット率
-   Discord配信リトライとレート制限カウンター

必要なログ:

-   `sessionKey`, `runId`, `backend`, `threadId`, `idempotencyKey` でキー付けされた構造化ログ
-   セッションと実行状態機械のための明示的な状態遷移ログ
-   編集安全な引数と終了サマリー付きアダプターコマンドログ

必要な診断:

-   `/acp sessions` は状態、アクティブな実行、最後のエラー、バインディングステータスを含む
-   `/acp doctor` (または同等) はバックエンド登録、ストアヘルス、古いバインディングを検証

### 設定優先順位と実効値

ACP有効化優先順位:

-   アカウントオーバーライド: `channels.discord.accounts..threadBindings.spawnAcpSessions`
-   チャネルオーバーライド: `channels.discord.threadBindings.spawnAcpSessions`
-   グローバルACPゲート: `acp.enabled`
-   ディスパッチゲート: `acp.dispatch.enabled`
-   バックエンド可用性: `acp.backend` の登録済みバックエンド

自動有効化動作:

-   ACPが設定されている場合 (`acp.enabled=true`, `acp.dispatch.enabled=true`, または `acp.backend=acpx`)、プラグイン自動有効化は `plugins.entries.acpx.enabled=true` をマーク（拒否リストまたは明示的に無効化されていない限り）

TTL実効値:

-   `min(セッション ttl, discordスレッドバインディング ttl, acpランタイム ttl)`

### テストマップ

単体テスト:

-   `src/acp/runtime/registry.test.ts` (新規)
-   `src/auto-reply/reply/dispatch-from-config.acp.test.ts` (新規)
-   `src/infra/outbound/bound-delivery-router.test.ts` (ACPフェイルクローズケースを拡張)
-   `src/config/sessions/types.test.ts` または最も近いセッションストアテスト（ACPメタデータ永続化）

統合テスト:

-   `src/discord/monitor/reply-delivery.test.ts` (バインドACP配信ターゲット動作)
-   `src/discord/monitor/message-handler.preflight*.test.ts` (バインドACPセッションキールーティング連続性)
-   バックエンドパッケージ内のacpxプラグインランタイムテスト（サービス登録/開始/停止 + イベント正規化）

ゲートウェイe2eテスト:

-   `src/gateway/server.sessions.gateway-server-sessions-a.e2e.test.ts` (ACPリセット/削除ライフサイクルカバレッジを拡張)
-   スポーン、メッセージ、ストリーム、キャンセル、フォーカス解除、再起動リカバリのためのACPスレッドターンラウンドトリップe2e

### ロールアウトガード

独立したACPディスパッチ停止スイッチを追加:

-   `acp.dispatch.enabled` デフォルト `false` (最初