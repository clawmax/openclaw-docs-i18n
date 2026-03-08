

  基本

  
# Pi 統合アーキテクチャ

このドキュメントでは、OpenClawが [pi-coding-agent](https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent) およびその関連パッケージ (`pi-ai`, `pi-agent-core`, `pi-tui`) と統合してAIエージェント機能を実現する方法について説明します。

## 概要

OpenClawは、pi SDKを使用してAIコーディングエージェントをメッセージングゲートウェイアーキテクチャに埋め込みます。piをサブプロセスとして起動したりRPCモードを使用する代わりに、OpenClawはpiの `AgentSession` を `createAgentSession()` を介して直接インポートおよびインスタンス化します。この埋め込みアプローチにより、以下が提供されます:

-   セッションライフサイクルとイベントハンドリングの完全な制御
-   カスタムツールの注入（メッセージング、サンドボックス、チャネル固有のアクション）
-   チャネル/コンテキストごとのシステムプロンプトのカスタマイズ
-   ブランチ/圧縮をサポートしたセッション永続化
-   フェイルオーバーを伴うマルチアカウント認証プロファイルのローテーション
-   プロバイダーに依存しないモデル切り替え

## パッケージ依存関係

```json
{
  "@mariozechner/pi-agent-core": "0.49.3",
  "@mariozechner/pi-ai": "0.49.3",
  "@mariozechner/pi-coding-agent": "0.49.3",
  "@mariozechner/pi-tui": "0.49.3"
}
```

| パッケージ | 目的 |
| --- | --- |
| `pi-ai` | コアLLM抽象化: `Model`, `streamSimple`, メッセージタイプ, プロバイダーAPI |
| `pi-agent-core` | エージェントループ, ツール実行, `AgentMessage` タイプ |
| `pi-coding-agent` | 高レベルSDK: `createAgentSession`, `SessionManager`, `AuthStorage`, `ModelRegistry`, 組み込みツール |
| `pi-tui` | ターミナルUIコンポーネント (OpenClawのローカルTUIモードで使用) |

## ファイル構造

```
src/agents/
├── pi-embedded-runner.ts          # pi-embedded-runner/ からの再エクスポート
├── pi-embedded-runner/
│   ├── run.ts                     # メインエントリ: runEmbeddedPiAgent()
│   ├── run/
│   │   ├── attempt.ts             # セッション設定を含む単一試行ロジック
│   │   ├── params.ts              # RunEmbeddedPiAgentParams タイプ
│   │   ├── payloads.ts            # 実行結果から応答ペイロードを構築
│   │   ├── images.ts              # ビジョンモデルへの画像注入
│   │   └── types.ts               # EmbeddedRunAttemptResult
│   ├── abort.ts                   # 中止エラー検出
│   ├── cache-ttl.ts               # コンテキスト剪定のためのキャッシュTTL追跡
│   ├── compact.ts                 # 手動/自動圧縮ロジック
│   ├── extensions.ts              # 埋め込み実行用にpi拡張機能をロード
│   ├── extra-params.ts            # プロバイダー固有のストリームパラメータ
│   ├── google.ts                  # Google/Gemini ターン順序修正
│   ├── history.ts                 # 履歴制限 (DM vs グループ)
│   ├── lanes.ts                   # セッション/グローバルコマンドレーン
│   ├── logger.ts                  # サブシステムロガー
│   ├── model.ts                   # ModelRegistry によるモデル解決
│   ├── runs.ts                    # アクティブな実行の追跡、中止、キュー
│   ├── sandbox-info.ts            # システムプロンプトのためのサンドボックス情報
│   ├── session-manager-cache.ts   # SessionManager インスタンスキャッシュ
│   ├── session-manager-init.ts    # セッションファイル初期化
│   ├── system-prompt.ts           # システムプロンプトビルダー
│   ├── tool-split.ts              # ツールを builtIn と custom に分割
│   ├── types.ts                   # EmbeddedPiAgentMeta, EmbeddedPiRunResult
│   └── utils.ts                   # ThinkLevel マッピング、エラー説明
├── pi-embedded-subscribe.ts       # セッションイベント購読/ディスパッチ
├── pi-embedded-subscribe.types.ts # SubscribeEmbeddedPiSessionParams
├── pi-embedded-subscribe.handlers.ts # イベントハンドラーファクトリ
├── pi-embedded-subscribe.handlers.lifecycle.ts
├── pi-embedded-subscribe.handlers.types.ts
├── pi-embedded-block-chunker.ts   # ストリーミングブロック返信チャンキング
├── pi-embedded-messaging.ts       # メッセージングツール送信追跡
├── pi-embedded-helpers.ts         # エラー分類、ターンバリデーション
├── pi-embedded-helpers/           # ヘルパーモジュール
├── pi-embedded-utils.ts           # フォーマットユーティリティ
├── pi-tools.ts                    # createOpenClawCodingTools()
├── pi-tools.abort.ts              # ツール用AbortSignalラッピング
├── pi-tools.policy.ts             # ツール許可リスト/拒否リストポリシー
├── pi-tools.read.ts               # Read ツールカスタマイズ
├── pi-tools.schema.ts             # ツールスキーマ正規化
├── pi-tools.types.ts              # AnyAgentTool タイプエイリアス
├── pi-tool-definition-adapter.ts  # AgentTool -> ToolDefinition アダプター
├── pi-settings.ts                 # 設定オーバーライド
├── pi-extensions/                 # カスタムpi拡張機能
│   ├── compaction-safeguard.ts    # セーフガード拡張機能
│   ├── compaction-safeguard-runtime.ts
│   ├── context-pruning.ts         # キャッシュTTLコンテキスト剪定拡張機能
│   └── context-pruning/
├── model-auth.ts                  # 認証プロファイル解決
├── auth-profiles.ts               # プロファイルストア、クールダウン、フェイルオーバー
├── model-selection.ts             # デフォルトモデル解決
├── models-config.ts               # models.json 生成
├── model-catalog.ts               # モデルカタログキャッシュ
├── context-window-guard.ts        # コンテキストウィンドウ検証
├── failover-error.ts              # FailoverError クラス
├── defaults.ts                    # DEFAULT_PROVIDER, DEFAULT_MODEL
├── system-prompt.ts               # buildAgentSystemPrompt()
├── system-prompt-params.ts        # システムプロンプトパラメータ解決
├── system-prompt-report.ts        # デバッグレポート生成
├── tool-summaries.ts              # ツール説明要約
├── tool-policy.ts                 # ツールポリシー解決
├── transcript-policy.ts           # トランスクリプト検証ポリシー
├── skills.ts                      # スキルスナップショット/プロンプト構築
├── skills/                        # スキルサブシステム
├── sandbox.ts                     # サンドボックスコンテキスト解決
├── sandbox/                       # サンドボックスサブシステム
├── channel-tools.ts               # チャネル固有ツール注入
├── openclaw-tools.ts              # OpenClaw固有ツール
├── bash-tools.ts                  # exec/process ツール
├── apply-patch.ts                 # apply_patch ツール (OpenAI)
├── tools/                         # 個々のツール実装
│   ├── browser-tool.ts
│   ├── canvas-tool.ts
│   ├── cron-tool.ts
│   ├── discord-actions*.ts
│   ├── gateway-tool.ts
│   ├── image-tool.ts
│   ├── message-tool.ts
│   ├── nodes-tool.ts
│   ├── session*.ts
│   ├── slack-actions.ts
│   ├── telegram-actions.ts
│   ├── web-*.ts
│   └── whatsapp-actions.ts
└── ...
```

## コア統合フロー

### 1\. 埋め込みエージェントの実行

メインエントリポイントは `pi-embedded-runner/run.ts` 内の `runEmbeddedPiAgent()` です:

```typescript
import { runEmbeddedPiAgent } from "./agents/pi-embedded-runner.js";

const result = await runEmbeddedPiAgent({
  sessionId: "user-123",
  sessionKey: "main:whatsapp:+1234567890",
  sessionFile: "/path/to/session.jsonl",
  workspaceDir: "/path/to/workspace",
  config: openclawConfig,
  prompt: "Hello, how are you?",
  provider: "anthropic",
  model: "claude-sonnet-4-20250514",
  timeoutMs: 120_000,
  runId: "run-abc",
  onBlockReply: async (payload) => {
    await sendToChannel(payload.text, payload.mediaUrls);
  },
});
```

### 2\. セッション作成

`runEmbeddedAttempt()` ( `runEmbeddedPiAgent()` から呼び出される) 内部で、pi SDKが使用されます:

```typescript
import {
  createAgentSession,
  DefaultResourceLoader,
  SessionManager,
  SettingsManager,
} from "@mariozechner/pi-coding-agent";

const resourceLoader = new DefaultResourceLoader({
  cwd: resolvedWorkspace,
  agentDir,
  settingsManager,
  additionalExtensionPaths,
});
await resourceLoader.reload();

const { session } = await createAgentSession({
  cwd: resolvedWorkspace,
  agentDir,
  authStorage: params.authStorage,
  modelRegistry: params.modelRegistry,
  model: params.model,
  thinkingLevel: mapThinkingLevel(params.thinkLevel),
  tools: builtInTools,
  customTools: allCustomTools,
  sessionManager,
  settingsManager,
  resourceLoader,
});

applySystemPromptOverrideToSession(session, systemPromptOverride);
```

### 3\. イベント購読

`subscribeEmbeddedPiSession()` はpiの `AgentSession` イベントを購読します:

```typescript
const subscription = subscribeEmbeddedPiSession({
  session: activeSession,
  runId: params.runId,
  verboseLevel: params.verboseLevel,
  reasoningMode: params.reasoningLevel,
  toolResultFormat: params.toolResultFormat,
  onToolResult: params.onToolResult,
  onReasoningStream: params.onReasoningStream,
  onBlockReply: params.onBlockReply,
  onPartialReply: params.onPartialReply,
  onAgentEvent: params.onAgentEvent,
});
```

処理されるイベントには以下が含まれます:

-   `message_start` / `message_end` / `message_update` (ストリーミングテキスト/思考)
-   `tool_execution_start` / `tool_execution_update` / `tool_execution_end`
-   `turn_start` / `turn_end`
-   `agent_start` / `agent_end`
-   `auto_compaction_start` / `auto_compaction_end`

### 4\. プロンプト送信

セットアップ後、セッションにプロンプトが送信されます:

```typescript
await session.prompt(effectivePrompt, { images: imageResult.images });
```

SDKは完全なエージェントループを処理します: LLMへの送信、ツール呼び出しの実行、応答のストリーミング。画像注入はプロンプトローカルです: OpenClawは現在のプロンプトから画像参照をロードし、そのターンのみ `images` を介して渡します。古い履歴ターンを再スキャンして画像ペイロードを再注入することはありません。

## ツールアーキテクチャ

### ツールパイプライン

1.  **ベースツール**: piの `codingTools` (read, bash, edit, write)
2.  **カスタム置換**: OpenClawはbashを `exec`/`process` で置き換え、サンドボックス用にread/edit/writeをカスタマイズ
3.  **OpenClawツール**: メッセージング、ブラウザ、キャンバス、セッション、cron、ゲートウェイなど
4.  **チャネルツール**: Discord/Telegram/Slack/WhatsApp固有のアクションツール
5.  **ポリシーフィルタリング**: プロファイル、プロバイダー、エージェント、グループ、サンドボックスポリシーでフィルタリングされたツール
6.  **スキーマ正規化**: Gemini/OpenAIの癖に対応するためのスキーマクリーンアップ
7.  **AbortSignalラッピング**: 中止信号を尊重するようにラップされたツール

### ツール定義アダプター

pi-agent-coreの `AgentTool` は、pi-coding-agentの `ToolDefinition` とは異なる `execute` シグネチャを持っています。 `pi-tool-definition-adapter.ts` 内のアダプターはこれを橋渡しします:

```bash
export function toToolDefinitions(tools: AnyAgentTool[]): ToolDefinition[] {
  return tools.map((tool) => ({
    name: tool.name,
    label: tool.label ?? name,
    description: tool.description ?? "",
    parameters: tool.parameters,
    execute: async (toolCallId, params, onUpdate, _ctx, signal) => {
      // pi-coding-agent シグネチャは pi-agent-core と異なる
      return await tool.execute(toolCallId, params, signal, onUpdate);
    },
  }));
}
```

### ツール分割戦略

`splitSdkTools()` はすべてのツールを `customTools` 経由で渡します:

```bash
export function splitSdkTools(options: { tools: AnyAgentTool[]; sandboxEnabled: boolean }) {
  return {
    builtInTools: [], // 空。すべてをオーバーライドする
    customTools: toToolDefinitions(options.tools),
  };
}
```

これにより、OpenClawのポリシーフィルタリング、サンドボックス統合、および拡張ツールセットがプロバイダー間で一貫して維持されます。

## システムプロンプト構築

システムプロンプトは `buildAgentSystemPrompt()` (`system-prompt.ts`) で構築されます。ツール、ツール呼び出しスタイル、安全ガードレール、OpenClaw CLIリファレンス、スキル、ドキュメント、ワークスペース、サンドボックス、メッセージング、返信タグ、音声、サイレント返信、ハートビート、ランタイムメタデータ、さらに有効な場合はメモリとリアクションを含む完全なプロンプトを組み立てます。セクションは、サブエージェントで使用される最小プロンプトモード用にトリミングされます。プロンプトは、セッション作成後に `applySystemPromptOverrideToSession()` を介して適用されます:

```typescript
const systemPromptOverride = createSystemPromptOverride(appendPrompt);
applySystemPromptOverrideToSession(session, systemPromptOverride);
```

## セッション管理

### セッションファイル

セッションはツリー構造 (id/parentIdリンク) を持つJSONLファイルです。Piの `SessionManager` が永続性を処理します:

```typescript
const sessionManager = SessionManager.open(params.sessionFile);
```

OpenClawはツール結果の安全性のために `guardSessionManager()` でこれをラップします。

### セッションキャッシュ

`session-manager-cache.ts` はSessionManagerインスタンスをキャッシュして、ファイルの繰り返し解析を回避します:

```typescript
await prewarmSessionFile(params.sessionFile);
sessionManager = SessionManager.open(params.sessionFile);
trackSessionManagerAccess(params.sessionFile);
```

### 履歴制限

`limitHistoryTurns()` はチャネルタイプ (DM vs グループ) に基づいて会話履歴をトリミングします。

### 圧縮

自動圧縮はコンテキストオーバーフロー時にトリガーされます。 `compactEmbeddedPiSessionDirect()` は手動圧縮を処理します:

```typescript
const compactResult = await compactEmbeddedPiSessionDirect({
  sessionId, sessionFile, provider, model, ...
});
```

## 認証とモデル解決

### 認証プロファイル

OpenClawはプロバイダーごとに複数のAPIキーを持つ認証プロファイルストアを維持します:

```typescript
const authStore = ensureAuthProfileStore(agentDir, { allowKeychainPrompt: false });
const profileOrder = resolveAuthProfileOrder({ cfg, store: authStore, provider, preferredProfile });
```

プロファイルは失敗時にローテーションされ、クールダウン追跡が行われます:

```typescript
await markAuthProfileFailure({ store, profileId, reason, cfg, agentDir });
const rotated = await advanceAuthProfile();
```

### モデル解決

```typescript
import { resolveModel } from "./pi-embedded-runner/model.js";

const { model, error, authStorage, modelRegistry } = resolveModel(
  provider,
  modelId,
  agentDir,
  config,
);

// piの ModelRegistry と AuthStorage を使用
authStorage.setRuntimeApiKey(model.provider, apiKeyInfo.apiKey);
```

### フェイルオーバー

`FailoverError` は設定時にモデルフォールバックをトリガーします:

```
if (fallbackConfigured && isFailoverErrorMessage(errorText)) {
  throw new FailoverError(errorText, {
    reason: promptFailoverReason ?? "unknown",
    provider,
    model: modelId,
    profileId,
    status: resolveFailoverStatus(promptFailoverReason),
  });
}
```

## Pi 拡張機能

OpenClawは特殊な動作のためにカスタムpi拡張機能をロードします:

### 圧縮セーフガード

`src/agents/pi-extensions/compaction-safeguard.ts` は圧縮にガードレールを追加します。適応型トークンバジェット、ツール失敗、ファイル操作サマリーを含みます:

```
if (resolveCompactionMode(params.cfg) === "safeguard") {
  setCompactionSafeguardRuntime(params.sessionManager, { maxHistoryShare });
  paths.push(resolvePiExtensionPath("compaction-safeguard"));
}
```

### コンテキスト剪定

`src/agents/pi-extensions/context-pruning.ts` はキャッシュTTLベースのコンテキスト剪定を実装します:

```
if (cfg?.agents?.defaults?.contextPruning?.mode === "cache-ttl") {
  setContextPruningRuntime(params.sessionManager, {
    settings,
    contextWindowTokens,
    isToolPrunable,
    lastCacheTouchAt,
  });
  paths.push(resolvePiExtensionPath("context-pruning"));
}
```

## ストリーミングとブロック返信

### ブロックチャンキング

`EmbeddedBlockChunker` はストリーミングテキストを個別の返信ブロックに管理します:

```typescript
const blockChunker = blockChunking ? new EmbeddedBlockChunker(blockChunking) : null;
```

### 思考/最終タグ除去

ストリーミング出力は ``/`` ブロックを除去し、 `` コンテンツを抽出するために処理されます:

```typescript
const stripBlockTags = (text: string, state: { thinking: boolean; final: boolean }) => {
  // <think>...</think> コンテンツを除去
  // enforceFinalTag が true の場合、<final>...</final> コンテンツのみを返す
};
```

### 返信ディレクティブ

`[[media:url]]`, `[[voice]]`, `[[reply:id]]` などの返信ディレクティブが解析および抽出されます:

```typescript
const { text: cleanedText, mediaUrls, audioAsVoice, replyToId } = consumeReplyDirectives(chunk);
```

## エラーハンドリング

### エラー分類

`pi-embedded-helpers.ts` は適切な処理のためにエラーを分類します:

```
isContextOverflowError(errorText)     // コンテキストが大きすぎる
isCompactionFailureError(errorText)   // 圧縮失敗
isAuthAssistantError(lastAssistant)   // 認証失敗
isRateLimitAssistantError(...)        // レート制限
isFailoverAssistantError(...)         // フェイルオーバーすべき
classifyFailoverReason(errorText)     // "auth" | "rate_limit" | "quota" | "timeout" | ...
```

### 思考レベルフォールバック

思考レベルがサポートされていない場合、フォールバックします:

```typescript
const fallbackThinking = pickFallbackThinkingLevel({
  message: errorText,
  attempted: attemptedThinking,
});
if (fallbackThinking) {
  thinkLevel = fallbackThinking;
  continue;
}
```

## サンドボックス統合

サンドボックスモードが有効な場合、ツールとパスは制約されます:

```typescript
const sandbox = await resolveSandboxContext({
  config: params.config,
  sessionKey: sandboxSessionKey,
  workspaceDir: resolvedWorkspace,
});

if (sandboxRoot) {
  // サンドボックス化された read/edit/write ツールを使用
  // Exec はコンテナ内で実行
  // Browser はブリッジURLを使用
}
```

## プロバイダー固有の処理

### Anthropic

-   拒否マジック文字列のスクラビング
-   連続したロールのターンバリデーション
-   Claude Code パラメータ互換性

### Google/Gemini

-   ターン順序修正 (`applyGoogleTurnOrderingFix`)
-   ツールスキーマサニタイズ (`sanitizeToolsForGoogle`)
-   セッション履歴サニタイズ (`sanitizeSessionHistory`)

### OpenAI

-   Codexモデル用の `apply_patch` ツール
-   思考レベルダウングレード処理

## TUI統合

OpenClawはまた、pi-tuiコンポーネントを直接使用するローカルTUIモードを持っています:

```bash
// src/tui/tui.ts
import { ... } from "@mariozechner/pi-tui";
```

これは、piのネイティブモードと同様のインタラクティブなターミナルエクスペリエンスを提供します。

## Pi CLIとの主な違い

| 側面 | Pi CLI | OpenClaw 埋め込み |
| --- | --- | --- |
| 呼び出し | `pi` コマンド / RPC | SDK via `createAgentSession()` |
| ツール | デフォルトコーディングツール | カスタムOpenClawツールスイート |
| システムプロンプト | AGENTS.md + プロンプト | チャネル/コンテキストごとの動的プロンプト |
| セッションストレージ | `~/.pi/agent/sessions/` | `~/.openclaw/agents//sessions/` (または `$OPENCLAW_STATE_DIR/agents//sessions/`) |
| 認証 | 単一認証情報 | ローテーション付きマルチプロファイル |
| 拡張機能 | ディスクからロード | プログラム的 + ディスクパス |
| イベント処理 | TUIレンダリング | コールバックベース (onBlockReply など) |

## 将来の検討事項

再作業の可能性がある領域:

1.  **ツールシグネチャの調整**: 現在、pi-agent-coreとpi-coding-agentのシグネチャ間を適応中
2.  **セッションマネージャーラッピング**: `guardSessionManager` は安全性を追加するが複雑さを増す
3.  **拡張機能ロード**: piの `ResourceLoader` をより直接的に使用できる可能性
4.  **ストリーミングハンドラーの複雑さ**: `subscribeEmbeddedPiSession` が大きくなりすぎている
5.  **プロバイダーの癖**: piが潜在的に処理できる可能性のある多くのプロバイダー固有のコードパス

## テスト

Pi統合のカバレッジは以下のスイートに及びます:

-   `src/agents/pi-*.test.ts`
-   `src/agents/pi-auth-json.test.ts`
-   `src/agents/pi-embedded-*.test.ts`
-   `src/agents/pi-embedded-helpers*.test.ts`
-   `src/agents/pi-embedded-runner*.test.ts`
-   `src/agents/pi-embedded-runner/**/*.test.ts`
-   `src/agents/pi-embedded-subscribe*.test.ts`
-   `src/agents/pi-tools*.test.ts`
-   `src/agents/pi-tool-definition-adapter*.test.ts`
-   `src/agents/pi-settings.test.ts`
-   `src/agents/pi-extensions/**/*.test.ts`

ライブ/オプトイン:

-   `src/agents/pi-embedded-runner-extraparams.live.test.ts` (`OPENCLAW_LIVE_TEST=1` を有効化)

現在の実行コマンドについては、[Pi 開発ワークフロー](./pi-dev.md) を参照してください。

[ゲートウェイアーキテクチャ](./concepts/architecture.md)