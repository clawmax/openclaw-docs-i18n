

  セッションとメモリ

  
# セッションツール

目標: エージェントがセッションを一覧表示し、履歴を取得し、別のセッションに送信できる、小さく誤用しにくいツールセット。

## ツール名

-   `sessions_list`
-   `sessions_history`
-   `sessions_send`
-   `sessions_spawn`

## キーモデル

-   メインの直接チャットバケットは常にリテラルキー `"main"` です（現在のエージェントのメインキーに解決されます）。
-   グループチャットは `agent:::group:` または `agent:::channel:` を使用します（完全なキーを渡します）。
-   クローンジョブは `cron:<job.id>` を使用します。
-   フックは明示的に設定されていない限り `hook:` を使用します。
-   ノードセッションは明示的に設定されていない限り `node-` を使用します。

`global` と `unknown` は予約済みの値であり、一覧表示されることはありません。`session.scope = "global"` の場合、すべてのツールで `main` にエイリアスされるため、呼び出し側が `global` を見ることはありません。

## sessions_list

セッションを行の配列として一覧表示します。パラメータ:

-   `kinds?: string[]` フィルター: `"main" | "group" | "cron" | "hook" | "node" | "other"` のいずれか
-   `limit?: number` 最大行数 (デフォルト: サーバーのデフォルト、例: 200にクランプ)
-   `activeMinutes?: number` N分以内に更新されたセッションのみ
-   `messageLimit?: number` 0 = メッセージなし (デフォルト 0); >0 = 最後のN件のメッセージを含める

動作:

-   `messageLimit > 0` の場合、セッションごとに `chat.history` を取得し、最後のN件のメッセージを含めます。
-   ツール結果はリスト出力から除外されます。ツールメッセージには `sessions_history` を使用してください。
-   **サンドボックス化された** エージェントセッションで実行する場合、セッションツールはデフォルトで **スポーンされたセッションのみの可視性** になります（下記参照）。

行の形状 (JSON):

-   `key`: セッションキー (文字列)
-   `kind`: `main | group | cron | hook | node | other`
-   `channel`: `whatsapp | telegram | discord | signal | imessage | webchat | internal | unknown`
-   `displayName` (グループ表示ラベル、利用可能な場合)
-   `updatedAt` (ミリ秒)
-   `sessionId`
-   `model`, `contextTokens`, `totalTokens`
-   `thinkingLevel`, `verboseLevel`, `systemSent`, `abortedLastRun`
-   `sendPolicy` (セッションで設定されている場合のオーバーライド)
-   `lastChannel`, `lastTo`
-   `deliveryContext` (利用可能な場合の正規化された `{ channel, to, accountId }`)
-   `transcriptPath` (ストアディレクトリ + sessionId から導出されたベストエフォートのパス)
-   `messages?` (`messageLimit > 0` の場合のみ)

## sessions_history

1つのセッションのトランスクリプトを取得します。パラメータ:

-   `sessionKey` (必須; セッションキーまたは `sessions_list` からの `sessionId` を受け入れます)
-   `limit?: number` 最大メッセージ数 (サーバーでクランプ)
-   `includeTools?: boolean` (デフォルト false)

動作:

-   `includeTools=false` の場合、`role: "toolResult"` メッセージをフィルタリングします。
-   生のトランスクリプト形式でメッセージ配列を返します。
-   `sessionId` が指定された場合、OpenClaw はそれを対応するセッションキーに解決します（存在しないIDはエラー）。

## sessions_send

別のセッションにメッセージを送信します。パラメータ:

-   `sessionKey` (必須; セッションキーまたは `sessions_list` からの `sessionId` を受け入れます)
-   `message` (必須)
-   `timeoutSeconds?: number` (デフォルト >0; 0 = ファイアアンドフォーゲット)

動作:

-   `timeoutSeconds = 0`: キューに入れ、`{ runId, status: "accepted" }` を返します。
-   `timeoutSeconds > 0`: 完了まで最大N秒待機し、`{ runId, status: "ok", reply }` を返します。
-   待機がタイムアウトした場合: `{ runId, status: "timeout", error }`。実行は継続されます。後で `sessions_history` を呼び出してください。
-   実行が失敗した場合: `{ runId, status: "error", error }`。
-   アナウンス配信はプライマリ実行完了後に実行され、ベストエフォートです。`status: "ok"` はアナウンスが配信されたことを保証しません。
-   待機はゲートウェイ `agent.wait` (サーバーサイド) を介して行われるため、再接続しても待機が中断されません。
-   エージェント間メッセージコンテキストはプライマリ実行のために注入されます。
-   セッション間メッセージは `message.provenance.kind = "inter_session"` で永続化されるため、トランスクリプトリーダーはルーティングされたエージェント指示と外部ユーザー入力を区別できます。
-   プライマリ実行が完了した後、OpenClaw は **返信ループ** を実行します:
    -   ラウンド2以降は、リクエスタとターゲットエージェントの間で交互に行われます。
    -   正確に `REPLY_SKIP` と返信すると、ピンポンを停止します。
    -   最大ターン数は `session.agentToAgent.maxPingPongTurns` です (0–5, デフォルト 5)。
-   ループが終了すると、OpenClaw は **エージェント間アナウンスステップ** を実行します (ターゲットエージェントのみ):
    -   正確に `ANNOUNCE_SKIP` と返信すると沈黙を保ちます。
    -   それ以外の返信はターゲットチャネルに送信されます。
    -   アナウンスステップには、元のリクエスト + ラウンド1の返信 + 最新のピンポン返信が含まれます。

## チャネルフィールド

-   グループの場合、`channel` はセッションエントリに記録されたチャネルです。
-   直接チャットの場合、`channel` は `lastChannel` からマッピングされます。
-   クローン/フック/ノードの場合、`channel` は `internal` です。
-   欠落している場合、`channel` は `unknown` です。

## セキュリティ / 送信ポリシー

チャネル/チャットタイプに基づくポリシーベースのブロック (セッションIDごとではありません)。

```json
{
  "session": {
    "sendPolicy": {
      "rules": [
        {
          "match": { "channel": "discord", "chatType": "group" },
          "action": "deny"
        }
      ],
      "default": "allow"
    }
  }
}
```

ランタイムオーバーライド (セッションエントリごと):

-   `sendPolicy: "allow" | "deny"` (未設定 = 設定を継承)
-   `sessions.patch` または所有者のみの `/send on|off|inherit` (スタンドアロンメッセージ) で設定可能。

適用ポイント:

-   `chat.send` / `agent` (ゲートウェイ)
-   自動返信配信ロジック

## sessions_spawn

隔離されたセッションでサブエージェント実行をスポーンし、結果をリクエスタのチャットチャネルにアナウンスします。パラメータ:

-   `task` (必須)
-   `label?` (オプション; ログ/UIに使用)
-   `agentId?` (オプション; 許可されている場合、別のエージェントIDの下でスポーン)
-   `model?` (オプション; サブエージェントモデルをオーバーライド; 無効な値はエラー)
-   `thinking?` (オプション; サブエージェント実行の思考レベルをオーバーライド)
-   `runTimeoutSeconds?` (設定されている場合は `agents.defaults.subagents.runTimeoutSeconds` にデフォルト、それ以外は `0`; 設定されている場合、N秒後にサブエージェント実行を中止)
-   `thread?` (デフォルト false; チャネル/プラグインでサポートされている場合、このスポーンに対してスレッドバインドルーティングをリクエスト)
-   `mode?` (`run|session`; デフォルトは `run`、ただし `thread=true` の場合はデフォルトで `session`; `mode="session"` には `thread=true` が必要)
-   `cleanup?` (`delete|keep`, デフォルト `keep`)
-   `sandbox?` (`inherit|require`, デフォルト `inherit`; `require` はターゲット子ランタイムがサンドボックス化されていない限りスポーンを拒否)
-   `attachments?` (オプションのインラインファイル配列; サブエージェントランタイムのみ、ACPは拒否)。各エントリ: `{ name, content, encoding?: "utf8" | "base64", mimeType? }`。ファイルは子ワークスペースの `.openclaw/attachments//` に具体化されます。ファイルごとにsha256を含むレシートを返します。
-   `attachAs?` (オプション; 将来のマウント実装用に予約された `{ mountPath? }` ヒント)

許可リスト:

-   `agents.list[].subagents.allowAgents`: `agentId` を介して許可されるエージェントIDのリスト (`["*"]` で任意を許可)。デフォルト: リクエスタエージェントのみ。
-   サンドボックス継承ガード: リクエスタセッションがサンドボックス化されている場合、`sessions_spawn` はサンドボックス化されていないターゲットを拒否します。

探索:

-   `agents_list` を使用して、`sessions_spawn` で許可されているエージェントIDを発見します。

動作:

-   `deliver: false` で新しい `agent::subagent:` セッションを開始します。
-   サブエージェントはデフォルトで **セッションツールを除く** 完全なツールセットを使用します (`tools.subagents.tools` で設定可能)。
-   サブエージェントは `sessions_spawn` を呼び出すことができません (サブエージェント → サブエージェントのスポーンは不可)。
-   常に非ブロッキング: 直ちに `{ status: "accepted", runId, childSessionKey }` を返します。
-   `thread=true` の場合、チャネルプラグインは配信/ルーティングをスレッドターゲットにバインドできます (Discord サポートは `session.threadBindings.*` および `channels.discord.threadBindings.*` で制御されます)。
-   完了後、OpenClaw はサブエージェント **アナウンスステップ** を実行し、結果をリクエスタのチャットチャネルに投稿します。
    -   アシスタントの最終返信が空の場合、サブエージェント履歴からの最新の `toolResult` が `Result` として含まれます。
-   アナウンスステップ中に正確に `ANNOUNCE_SKIP` と返信すると沈黙を保ちます。
-   アナウンス返信は `Status`/`Result`/`Notes` に正規化されます。`Status` はランタイム結果から来ます (モデルテキストではありません)。
-   サブエージェントセッションは `agents.defaults.subagents.archiveAfterMinutes` (デフォルト: 60) 後に自動アーカイブされます。
-   アナウンス返信には統計行 (ランタイム、トークン、sessionKey/sessionId、トランスクリプトパス、オプションのコスト) が含まれます。

## サンドボックスセッション可視性

セッションツールは、セッション間アクセスを減らすためにスコープを設定できます。デフォルト動作:

-   `tools.sessions.visibility` はデフォルトで `tree` (現在のセッション + スポーンされたサブエージェントセッション) です。
-   サンドボックス化されたセッションの場合、`agents.defaults.sandbox.sessionToolsVisibility` で可視性を強制的にクランプできます。

設定:

```json
{
  tools: {
    sessions: {
      // "self" | "tree" | "agent" | "all"
      // デフォルト: "tree"
      visibility: "tree",
    },
  },
  agents: {
    defaults: {
      sandbox: {
        // デフォルト: "spawned"
        sessionToolsVisibility: "spawned", // または "all"
      },
    },
  },
}
```

注記:

-   `self`: 現在のセッションキーのみ。
-   `tree`: 現在のセッション + 現在のセッションによってスポーンされたセッション。
-   `agent`: 現在のエージェントIDに属する任意のセッション。
-   `all`: 任意のセッション (エージェント間アクセスには依然として `tools.agentToAgent` が必要)。
-   セッションがサンドボックス化されていて `sessionToolsVisibility="spawned"` の場合、`tools.sessions.visibility="all"` を設定しても、OpenClaw は可視性を `tree` にクランプします。

[セッションプルーニング](./session-pruning.md)[メモリ](./memory.md)