

  オートメーション

  
# フック

フックは、エージェントコマンドやイベントに応じてアクションを自動化するための拡張可能なイベント駆動システムを提供します。フックはディレクトリから自動的に検出され、OpenClaw でのスキルの動作と同様に、CLI コマンドを介して管理できます。

## 概要を把握する

フックは、何かが発生したときに実行される小さなスクリプトです。2種類あります：

-   **フック**（このページ）：`/new`、`/reset`、`/stop` などのエージェントイベントが発生したときに Gateway 内部で実行されます。
-   **ウェブフック**：外部システムが OpenClaw で作業をトリガーできるようにする外部 HTTP ウェブフックです。[ウェブフックフック](./webhook.md)を参照するか、Gmail ヘルパーコマンドには `openclaw webhooks` を使用してください。

フックはプラグイン内にバンドルすることもできます。[プラグイン](../tools/plugin.md#plugin-hooks)を参照してください。一般的な用途：

-   セッションをリセットするときにメモリスナップショットを保存する
-   トラブルシューティングやコンプライアンスのためのコマンドの監査証跡を保持する
-   セッションの開始または終了時にフォローアップオートメーションをトリガーする
-   イベント発生時にエージェントワークスペースにファイルを書き込む、または外部 API を呼び出す

小さな TypeScript 関数を書けるなら、フックを書くことができます。フックは自動的に検出され、CLI を介して有効化または無効化できます。

## 概要

フックシステムにより、以下が可能になります：

-   `/new` が発行されたときにセッションコンテキストをメモリに保存する
-   監査のためのすべてのコマンドをログに記録する
-   エージェントライフサイクルイベントでカスタムオートメーションをトリガーする
-   コアコードを変更せずに OpenClaw の動作を拡張する

## はじめに

### バンドル済みフック

OpenClaw には、自動的に検出される4つのバンドル済みフックが付属しています：

-   **💾 session-memory**：`/new` を発行したときに、セッションコンテキストをエージェントワークスペース（デフォルト `~/.openclaw/workspace/memory/`）に保存します
-   **📎 bootstrap-extra-files**：`agent:bootstrap` 中に、設定された glob/パスパターンから追加のワークスペースブートストラップファイルを注入します
-   **📝 command-logger**：すべてのコマンドイベントを `~/.openclaw/logs/commands.log` に記録します
-   **🚀 boot-md**：ゲートウェイ起動時に `BOOT.md` を実行します（内部フックが有効である必要があります）

利用可能なフックを一覧表示：

```bash
openclaw hooks list
```

フックを有効化：

```bash
openclaw hooks enable session-memory
```

フックの状態を確認：

```bash
openclaw hooks check
```

詳細情報を取得：

```bash
openclaw hooks info session-memory
```

### オンボーディング

オンボーディング中（`openclaw onboard`）に、推奨フックを有効にするよう促されます。ウィザードは適格なフックを自動的に検出し、選択肢として提示します。

## フックの検出

フックは3つのディレクトリから自動的に検出されます（優先順位順）：

1.  **ワークスペースフック**：`/hooks/`（エージェントごと、最高優先度）
2.  **管理フック**：`~/.openclaw/hooks/`（ユーザーインストール、ワークスペース間で共有）
3.  **バンドル済みフック**：`/dist/hooks/bundled/`（OpenClaw に付属）

管理フックディレクトリは、**単一のフック**または**フックパック**（パッケージディレクトリ）のいずれかです。各フックは以下を含むディレクトリです：

```
my-hook/
├── HOOK.md          # メタデータ + ドキュメント
└── handler.ts       # ハンドラ実装
```

## フックパック (npm/アーカイブ)

フックパックは、`package.json` の `openclaw.hooks` を介して1つ以上のフックをエクスポートする標準的な npm パッケージです。以下でインストールします：

```bash
openclaw hooks install <path-or-spec>
```

Npm 仕様はレジストリのみです（パッケージ名 + オプションの正確なバージョンまたは dist-tag）。Git/URL/ファイル仕様と semver 範囲は拒否されます。ベア仕様と `@latest` は安定版トラックに留まります。npm がこれらのいずれかをプレリリースに解決した場合、OpenClaw は停止し、`@beta`/`@rc` などのプレリリースタグまたは正確なプレリリースバージョンを明示的にオプトインするよう要求します。`package.json` の例：

```json
{
  "name": "@acme/my-hooks",
  "version": "0.1.0",
  "openclaw": {
    "hooks": ["./hooks/my-hook", "./hooks/other-hook"]
  }
}
```

各エントリは、`HOOK.md` と `handler.ts`（または `index.ts`）を含むフックディレクトリを指します。フックパックは依存関係を同梱できます。これらは `~/.openclaw/hooks/` の下にインストールされます。各 `openclaw.hooks` エントリは、シンボリックリンク解決後もパッケージディレクトリ内に留まる必要があります。エスケープするエントリは拒否されます。セキュリティ注意：`openclaw hooks install` は `npm install --ignore-scripts`（ライフサイクルスクリプトなし）で依存関係をインストールします。フックパックの依存関係ツリーを「純粋な JS/TS」に保ち、`postinstall` ビルドに依存するパッケージを避けてください。

## フック構造

### HOOK.md フォーマット

`HOOK.md` ファイルには、YAML フロントマターのメタデータと Markdown ドキュメントが含まれます：

```
---
name: my-hook
description: "このフックが何をするかの短い説明"
homepage: https://docs.openclaw.ai/automation/hooks#my-hook
metadata:
  { "openclaw": { "emoji": "🔗", "events": ["command:new"], "requires": { "bins": ["node"] } } }
---

# My Hook

詳細なドキュメントはここに...

## 機能

- `/new` コマンドをリッスンします
- 何らかのアクションを実行します
- 結果をログに記録します

## 要件

- Node.js がインストールされている必要があります

## 設定

設定は不要です。
```

### メタデータフィールド

`metadata.openclaw` オブジェクトは以下をサポートします：

-   **`emoji`**：CLI 用の表示絵文字（例：`"💾"`）
-   **`events`**：リッスンするイベントの配列（例：`["command:new", "command:reset"]`）
-   **`export`**：使用する名前付きエクスポート（デフォルトは `"default"`）
-   **`homepage`**：ドキュメント URL
-   **`requires`**：オプションの要件
    -   **`bins`**：PATH 上に必要なバイナリ（例：`["git", "node"]`）
    -   **`anyBins`**：これらのバイナリの少なくとも1つが存在する必要があります
    -   **`env`**：必要な環境変数
    -   **`config`**：必要な設定パス（例：`["workspace.dir"]`）
    -   **`os`**：必要なプラットフォーム（例：`["darwin", "linux"]`）
-   **`always`**：適格性チェックをバイパス（ブール値）
-   **`install`**：インストール方法（バンドル済みフックの場合：`[{"id":"bundled","kind":"bundled"}]`）

### ハンドラ実装

`handler.ts` ファイルは `HookHandler` 関数をエクスポートします：

```typescript
const myHandler = async (event) => {
  // 'new' コマンドのみトリガー
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  console.log(`[my-hook] New command triggered`);
  console.log(`  Session: ${event.sessionKey}`);
  console.log(`  Timestamp: ${event.timestamp.toISOString()}`);

  // ここにカスタムロジックを記述

  // オプションでユーザーにメッセージを送信
  event.messages.push("✨ My hook executed!");
};

export default myHandler;
```

#### イベントコンテキスト

各イベントには以下が含まれます：

```json
{
  type: 'command' | 'session' | 'agent' | 'gateway' | 'message',
  action: string,              // 例: 'new', 'reset', 'stop', 'received', 'sent'
  sessionKey: string,          // セッション識別子
  timestamp: Date,             // イベント発生時刻
  messages: string[],          // ここにメッセージをプッシュしてユーザーに送信
  context: {
    // コマンドイベント:
    sessionEntry?: SessionEntry,
    sessionId?: string,
    sessionFile?: string,
    commandSource?: string,    // 例: 'whatsapp', 'telegram'
    senderId?: string,
    workspaceDir?: string,
    bootstrapFiles?: WorkspaceBootstrapFile[],
    cfg?: OpenClawConfig,
    // メッセージイベント (詳細は Message Events セクションを参照):
    from?: string,             // message:received
    to?: string,               // message:sent
    content?: string,
    channelId?: string,
    success?: boolean,         // message:sent
  }
}
```

## イベントタイプ

### コマンドイベント

エージェントコマンドが発行されたときにトリガーされます：

-   **`command`**：すべてのコマンドイベント（一般的なリスナー）
-   **`command:new`**：`/new` コマンドが発行されたとき
-   **`command:reset`**：`/reset` コマンドが発行されたとき
-   **`command:stop`**：`/stop` コマンドが発行されたとき

### セッションイベント

-   **`session:compact:before`**：圧縮が履歴を要約する直前に発生
-   **`session:compact:after`**：要約メタデータを含む圧縮完了後に発生

内部フックペイロードはこれらを `type: "session"` と `action: "compact:before"` / `action: "compact:after"` として出力します。リスナーは上記の結合キーで購読します。特定のハンドラ登録には、リテラルキーフォーマット `${type}:${action}` を使用します。これらのイベントでは、`session:compact:before` と `session:compact:after` を登録します。

### エージェントイベント

-   **`agent:bootstrap`**：ワークスペースブートストラップファイルが注入される前（フックは `context.bootstrapFiles` を変更可能）

### ゲートウェイイベント

ゲートウェイ起動時にトリガーされます：

-   **`gateway:startup`**：チャネル起動後、フックがロードされた後に発生

### メッセージイベント

メッセージが受信または送信されたときにトリガーされます：

-   **`message`**：すべてのメッセージイベント（一般的なリスナー）
-   **`message:received`**：任意のチャネルからインバウンドメッセージが受信されたとき。メディア理解前の処理の早い段階で発生します。コンテンツには、まだ処理されていないメディア添付ファイルの `<media:audio>` のような生のプレースホルダーが含まれる場合があります。
-   **`message:transcribed`**：オーディオ文字起こしやリンク理解を含むメッセージが完全に処理されたとき。この時点で、`transcript` にはオーディオメッセージの完全な文字起こしテキストが含まれます。文字起こしされたオーディオコンテンツにアクセスする必要がある場合は、このフックを使用します。
-   **`message:preprocessed`**：すべてのメディア + リンク理解が完了した後のすべてのメッセージに対して発生し、エージェントが認識する前に完全にエンリッチされた本文（文字起こし、画像の説明、リンクの要約）にフックがアクセスできるようにします。
-   **`message:sent`**：アウトバウンドメッセージが正常に送信されたとき

#### メッセージイベントコンテキスト

メッセージイベントには、メッセージに関する豊富なコンテキストが含まれます：

```
// message:received コンテキスト
{
  from: string,           // 送信者識別子（電話番号、ユーザーIDなど）
  content: string,        // メッセージコンテンツ
  timestamp?: number,     // 受信時の Unix タイムスタンプ
  channelId: string,      // チャネル（例: "whatsapp", "telegram", "discord"）
  accountId?: string,     // マルチアカウント設定のプロバイダーアカウントID
  conversationId?: string, // チャット/会話ID
  messageId?: string,     // プロバイダーからのメッセージID
  metadata?: {            // 追加のプロバイダー固有データ
    to?: string,
    provider?: string,
    surface?: string,
    threadId?: string,
    senderId?: string,
    senderName?: string,
    senderUsername?: string,
    senderE164?: string,
  }
}

// message:sent コンテキスト
{
  to: string,             // 受信者識別子
  content: string,        // 送信されたメッセージコンテンツ
  success: boolean,       // 送信が成功したかどうか
  error?: string,         // 送信失敗時のエラーメッセージ
  channelId: string,      // チャネル（例: "whatsapp", "telegram", "discord"）
  accountId?: string,     // プロバイダーアカウントID
  conversationId?: string, // チャット/会話ID
  messageId?: string,     // プロバイダーが返したメッセージID
  isGroup?: boolean,      // このアウトバウンドメッセージがグループ/チャネルコンテキストに属するかどうか
  groupId?: string,       // message:received との相関のためのグループ/チャネル識別子
}

// message:transcribed コンテキスト
{
  body?: string,          // エンリッチメント前の生のインバウンド本文
  bodyForAgent?: string,  // エージェントが認識するエンリッチされた本文
  transcript: string,     // オーディオ文字起こしテキスト
  channelId: string,      // チャネル（例: "telegram", "whatsapp"）
  conversationId?: string,
  messageId?: string,
}

// message:preprocessed コンテキスト
{
  body?: string,          // 生のインバウンド本文
  bodyForAgent?: string,  // メディア/リンク理解後の最終的なエンリッチされた本文
  transcript?: string,    // オーディオが存在した場合の文字起こし
  channelId: string,      // チャネル（例: "telegram", "whatsapp"）
  conversationId?: string,
  messageId?: string,
  isGroup?: boolean,
  groupId?: string,
}
```

#### 例：メッセージロガーフック

```typescript
const isMessageReceivedEvent = (event: { type: string; action: string }) =>
  event.type === "message" && event.action === "received";
const isMessageSentEvent = (event: { type: string; action: string }) =>
  event.type === "message" && event.action === "sent";

const handler = async (event) => {
  if (isMessageReceivedEvent(event as { type: string; action: string })) {
    console.log(`[message-logger] Received from ${event.context.from}: ${event.context.content}`);
  } else if (isMessageSentEvent(event as { type: string; action: string })) {
    console.log(`[message-logger] Sent to ${event.context.to}: ${event.context.content}`);
  }
};

export default handler;
```

### ツール結果フック（プラグイン API）

これらのフックはイベントストリームリスナーではありません。プラグインが OpenClaw がそれらを永続化する前にツール結果を同期的に調整できるようにします。

-   **`tool_result_persist`**：ツール結果がセッショントランスクリプトに書き込まれる前に変換します。同期的である必要があります。更新されたツール結果ペイロードを返すか、`undefined` を返してそのまま保持します。[エージェントループ](../concepts/agent-loop.md)を参照してください。

### プラグインフックイベント

プラグインフックランナーを通じて公開される圧縮ライフサイクルフック：

-   **`before_compaction`**：カウント/トークンメタデータを使用して圧縮前に実行
-   **`after_compaction`**：圧縮要約メタデータを使用して圧縮後に実行

### 将来のイベント

計画中のイベントタイプ：

-   **`session:start`**：新しいセッションが開始されたとき
-   **`session:end`**：セッションが終了したとき
-   **`agent:error`**：エージェントがエラーに遭遇したとき

## カスタムフックの作成

### 1\. 場所を選択

-   **ワークスペースフック**（`/hooks/`）：エージェントごと、最高優先度
-   **管理フック**（`~/.openclaw/hooks/`）：ワークスペース間で共有

### 2\. ディレクトリ構造を作成

```bash
mkdir -p ~/.openclaw/hooks/my-hook
cd ~/.openclaw/hooks/my-hook
```

### 3\. HOOK.md を作成

```
---
name: my-hook
description: "何か便利なことをします"
metadata: { "openclaw": { "emoji": "🎯", "events": ["command:new"] } }
---

# カスタムフック

このフックは、`/new` を発行したときに何か便利なことをします。
```

### 4\. handler.ts を作成

```typescript
const handler = async (event) => {
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  console.log("[my-hook] Running!");
  // ここにロジックを記述
};

export default handler;
```

### 5\. 有効化してテスト

```bash
# フックが検出されることを確認
openclaw hooks list

# 有効化
openclaw hooks enable my-hook

# ゲートウェイプロセスを再起動（macOS ではメニューバーアプリを再起動、または開発プロセスを再起動）

# イベントをトリガー
# メッセージングチャネル経由で /new を送信
```

## 設定

### 新しい設定フォーマット（推奨）

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "session-memory": { "enabled": true },
        "command-logger": { "enabled": false }
      }
    }
  }
}
```

### フックごとの設定

フックはカスタム設定を持つことができます：

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "my-hook": {
          "enabled": true,
          "env": {
            "MY_CUSTOM_VAR": "value"
          }
        }
      }
    }
  }
}
```

### 追加ディレクトリ

追加ディレクトリからフックをロード：

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "load": {
        "extraDirs": ["/path/to/more/hooks"]
      }
    }
  }
}
```

### レガシー設定フォーマット（まだサポート）

古い設定フォーマットは後方互換性のためにまだ機能します：

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts",
          "export": "default"
        }
      ]
    }
  }
}
```

注意：`module` はワークスペース相対パスである必要があります。絶対パスとワークスペース外へのトラバーサルは拒否されます。**移行**：新しいフックには新しい検出ベースのシステムを使用してください。レガシーハンドラはディレクトリベースのフックの後にロードされます。

## CLI コマンド

### フック一覧

```bash
# すべてのフックを一覧表示
openclaw hooks list

# 適格なフックのみ表示
openclaw hooks list --eligible

# 詳細出力（不足要件を表示）
openclaw hooks list --verbose

# JSON 出力
openclaw hooks list --json
```

### フック情報

```bash
# フックの詳細情報を表示
openclaw hooks info session-memory

# JSON 出力
openclaw hooks info session-memory --json
```

### 適格性チェック

```bash
# 適格性サマリーを表示
openclaw hooks check

# JSON 出力
openclaw hooks check --json
```

### 有効化/無効化

```bash
# フックを有効化
openclaw hooks enable session-memory

# フックを無効化
openclaw hooks disable command-logger
```

## バンドル済みフックリファレンス

### session-memory

`/new` を発行したときにセッションコンテキストをメモリに保存します。**イベント**：`command:new` **要件**：`workspace.dir` が設定されている必要があります **出力**：`/memory/YYYY-MM-DD-slug.md`（デフォルトは `~/.openclaw/workspace`）**機能**：

1.  リセット前のセッションエントリを使用して正しいトランスクリプトを特定
2.  会話の最後の15行を抽出
3.  LLM を使用して説明的なファイル名スラグを生成
4.  セッションメタデータを日付付きメモリファイルに保存

**出力例**：

```bash
# Session: 2026-01-16 14:30:00 UTC

- **Session Key**: agent:main:main
- **Session ID**: abc123def456
- **Source**: telegram
```

**ファイル名例**：

-   `2026-01-16-vendor-pitch.md`
-   `2026-01-16-api-design.md`
-   `2026-01-16-1430.md`（スラグ生成失敗時のフォールバックタイムスタンプ）

**有効化**：

```bash
openclaw hooks enable session-memory
```

### bootstrap-extra-files

`agent:bootstrap` 中に追加のブートストラップファイル（例えばモノレポローカルの `AGENTS.md` / `TOOLS.md`）を注入します。**イベント**：`agent:bootstrap` **要件**：`workspace.dir` が設定されている必要があります **出力**：ファイルは書き込まれません。ブートストラップコンテキストはメモリ内でのみ変更されます。**設定**：

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "bootstrap-extra-files": {
          "enabled": true,
          "paths": ["packages/*/AGENTS.md", "packages/*/TOOLS.md"]
        }
      }
    }
  }
}
```

**注意点**：

-   パスはワークスペースに対して相対的に解決されます。
-   ファイルはワークスペース内に留まる必要があります（realpath チェック済み）。
-   認識されたブートストラップベース名のみがロードされます。
-   サブエージェント許可リストが保持されます（`AGENTS.md` と `TOOLS.md` のみ）。

**有効化**：

```bash
openclaw hooks enable bootstrap-extra-files
```

### command-logger

すべてのコマンドイベントを一元化された監査ファイルに記録します。**イベント**：`command` **要件**：なし **出力**：`~/.openclaw/logs/commands.log` **機能**：

1.  イベント詳細（コマンドアクション、タイムスタンプ、セッションキー、送信者ID、ソース）をキャプチャ
2.  JSONL フォーマットでログファイルに追加
3.  バックグラウンドでサイレントに実行

**ログエントリ例**：

```json
{"timestamp":"2026-01-16T14:30:00.000Z","action":"new","sessionKey":"agent:main:main","senderId":"+1234567890","source":"telegram"}
{"timestamp":"2026-01-16T15:45:22.000Z","action":"stop","sessionKey":"agent:main:main","senderId":"user@example.com","source":"whatsapp"}
```

**ログ表示**：

```bash
# 最近のコマンドを表示
tail -n 20 ~/.openclaw/logs/commands.log

# jq で整形表示
cat ~/.openclaw/logs/commands.log | jq .

# アクションでフィルタ
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**有効化**：

```bash
openclaw hooks enable command-logger
```

### boot-md

ゲートウェイ起動時に（チャネル起動後）`BOOT.md` を実行します。これが実行されるには内部フックが有効である必要があります。**イベント**：`gateway:startup` **要件**：`workspace.dir` が設定されている必要があります **機能**：

1.  ワークスペースから `BOOT.md` を読み取る
2.  エージェントランナー経由で指示を実行
3.  メッセージツール経由で要求されたアウトバウンドメッセージを送信

**有効化**：

```bash
openclaw hooks enable boot-md
```

## ベストプラクティス

### ハンドラを高速に保つ

フックはコマンド処理中に実行されます。軽量に保ちます：

```
// ✓ 良い例 - 非同期作業、即座に返す
const handler: HookHandler = async (event) => {
  void processInBackground(event); // 発射して忘れる
};

// ✗ 悪い例 - コマンド処理をブロック
const handler: HookHandler = async (event) => {
  await slowDatabaseQuery(event);
  await evenSlowerAPICall(event);
};
```

### エラーを適切に処理する

常にリスクのある操作をラップします：

```typescript
const handler: HookHandler = async (event) => {
  try {
    await riskyOperation(event);
  } catch (err) {
    console.error("[my-handler] Failed:", err instanceof Error ? err.message : String(err));
    // スローしない - 他のハンドラを実行させる
  }
};
```

### イベントを早期にフィルタリングする

イベントが関連しない場合は早期に返します：

```typescript
const handler: HookHandler = async (event) => {
  // 'new' コマンドのみ処理
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  // ここにロジックを記述
};
```

### 特定のイベントキーを使用する

可能な限りメタデータに正確なイベントを指定します：

```
metadata: { "openclaw": { "events": ["command:new"] } } # 特定
```

以下ではなく：

```
metadata: { "openclaw": { "events": ["command"] } } # 一般的 - オーバーヘッドが大きい
```

## デバッグ

### フックロギングを有効化

ゲートウェイは起動時にフックロードをログに記録します：

```bash
Registered hook: session-memory -> command:new
Registered hook: bootstrap-extra-files -> agent:bootstrap
Registered hook: command-logger -> command
Registered hook: boot-md -> gateway:startup
```

### 検出を確認

検出されたすべてのフックを一覧表示：

```bash
openclaw hooks list --verbose
```

### 登録を確認

ハンドラ内で、呼び出されたときにログに記録します：

```typescript
const handler: HookHandler = async (event) => {
  console.log("[my-handler] Triggered:", event.type, event.action);
  // ロジック
};
```

### 適格性を確認

フックが適格でない理由を確認：

```bash
openclaw hooks info my-hook
```

出力で不足要件を確認します。

## テスト

### ゲートウェイログ

フック実行を確認するためにゲートウェイログを監視：

```bash
# macOS
./scripts/clawlog.sh -f

# その他のプラットフォーム
tail -f ~/.openclaw/gateway.log
```

### フックを直接テスト

ハンドラを単体でテスト：

```typescript
import { test } from "vitest";
import myHandler from "./hooks/my-hook/handler.js";

test("my handler works", async () => {
  const event = {
    type: "command",
    action: "new",
    sessionKey: "test-session",
    timestamp: new Date(),
    messages: [],
    context: { foo: "bar" },
  };

  await myHandler(event);

  // 副作用をアサート
});
```

## アーキテクチャ

### コアコンポーネント

-   **`src/hooks/types.ts`**：型定義
-   **`src/hooks/workspace.ts`**：ディレクトリスキャンとロード
-   **`src/hooks/frontmatter.ts`**：HOOK.md メタデータ解析
-   **`src/hooks/config.ts`**：適格性チェック
-   **`src/hooks/hooks-status.ts`**：ステータスレポート
-   **`src/hooks/loader.ts`**：動的モジュールローダー
-   **`src/cli/hooks-cli.ts`**：CLI コマンド
-   **`src/gateway/server-startup.ts`**：ゲートウェイ起動時にフックをロード
-   **`src/auto-reply/reply/commands-core.ts`**：コマンドイベントをトリガー

### 検出フロー

```
ゲートウェイ起動
    ↓
ディレクトリスキャン（ワークスペース → 管理 → バンドル済み）
    ↓
HOOK.md ファイルを解析
    ↓
適格性チェック（bins, env, config, os）
    ↓
適格なフックからハンドラをロード
    ↓
イベント用にハンドラを登録
```

### イベントフロー

```
ユーザーが /new を送信
    ↓
コマンド検証
    ↓
フックイベントを作成
    ↓
フックをトリガー（すべての登録済みハンドラ）