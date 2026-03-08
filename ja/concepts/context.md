

  基本

  
# コンテキスト

「コンテキスト」とは、**OpenClawが1回の実行のためにモデルに送信するすべてのもの**です。これはモデルの**コンテキストウィンドウ**（トークン制限）によって制限されます。初心者のためのメンタルモデル：

-   **システムプロンプト** (OpenClaw組み込み): ルール、ツール、スキルリスト、時間/ランタイム、および注入されたワークスペースファイル。
-   **会話履歴**: このセッションにおけるあなたのメッセージとアシスタントのメッセージ。
-   **ツール呼び出し/結果 + 添付ファイル**: コマンド出力、ファイル読み取り、画像/音声など。

コンテキストは「メモリ」と同じもの*ではありません*：メモリはディスクに保存され後で再読み込みできますが、コンテキストはモデルの現在のウィンドウ内にあるものです。

## クイックスタート (コンテキストの検査)

-   `/status` → 「私のウィンドウはどれくらい埋まっているか？」の簡易ビュー + セッション設定。
-   `/context list` → 何が注入されているか + おおよそのサイズ (ファイルごと + 合計)。
-   `/context detail` → 詳細な内訳: ファイルごと、ツールスキーマサイズごと、スキルエントリサイズごと、システムプロンプトサイズ。
-   `/usage tokens` → 通常の返信に返信ごとの使用量フッターを追加。
-   `/compact` → 古い履歴を要約してコンパクトなエントリにし、ウィンドウの空き容量を確保。

関連項目: [スラッシュコマンド](../tools/slash-commands.md), [トークン使用量とコスト](../reference/token-use.md), [コンパクション](./compaction.md)。

## 出力例

値はモデル、プロバイダー、ツールポリシー、およびワークスペースの内容によって異なります。

### /context list

```
🧠 Context breakdown
Workspace: <workspaceDir>
Bootstrap max/file: 20,000 chars
Sandbox: mode=non-main sandboxed=false
System prompt (run): 38,412 chars (~9,603 tok) (Project Context 23,901 chars (~5,976 tok))

Injected workspace files:
- AGENTS.md: OK | raw 1,742 chars (~436 tok) | injected 1,742 chars (~436 tok)
- SOUL.md: OK | raw 912 chars (~228 tok) | injected 912 chars (~228 tok)
- TOOLS.md: TRUNCATED | raw 54,210 chars (~13,553 tok) | injected 20,962 chars (~5,241 tok)
- IDENTITY.md: OK | raw 211 chars (~53 tok) | injected 211 chars (~53 tok)
- USER.md: OK | raw 388 chars (~97 tok) | injected 388 chars (~97 tok)
- HEARTBEAT.md: MISSING | raw 0 | injected 0
- BOOTSTRAP.md: OK | raw 0 chars (~0 tok) | injected 0 chars (~0 tok)

Skills list (system prompt text): 2,184 chars (~546 tok) (12 skills)
Tools: read, edit, write, exec, process, browser, message, sessions_send, …
Tool list (system prompt text): 1,032 chars (~258 tok)
Tool schemas (JSON): 31,988 chars (~7,997 tok) (counts toward context; not shown as text)
Tools: (same as above)

Session tokens (cached): 14,250 total / ctx=32,000
```

### /context detail

```
🧠 Context breakdown (detailed)
…
Top skills (prompt entry size):
- frontend-design: 412 chars (~103 tok)
- oracle: 401 chars (~101 tok)
… (+10 more skills)

Top tools (schema size):
- browser: 9,812 chars (~2,453 tok)
- exec: 6,240 chars (~1,560 tok)
… (+N more tools)
```

## コンテキストウィンドウにカウントされるもの

モデルが受け取るすべてのものがカウントされます。これには以下が含まれます：

-   システムプロンプト (すべてのセクション)。
-   会話履歴。
-   ツール呼び出し + ツール結果。
-   添付ファイル/トランスクリプト (画像/音声/ファイル)。
-   コンパクション要約とプルーニングのアーティファクト。
-   プロバイダーの「ラッパー」や隠れたヘッダー (表示されませんが、カウントされます)。

## OpenClawがシステムプロンプトを構築する方法

システムプロンプトは**OpenClawが所有**しており、実行ごとに再構築されます。これには以下が含まれます：

-   ツールリスト + 短い説明。
-   スキルリスト (メタデータのみ。詳細は以下を参照)。
-   ワークスペースの場所。
-   時間 (UTC + 設定されている場合はユーザー時間に変換)。
-   ランタイムメタデータ (ホスト/OS/モデル/思考)。
-   **プロジェクトコンテキスト**の下に注入されたワークスペースブートストラップファイル。

詳細な内訳: [システムプロンプト](./system-prompt.md)。

## 注入されるワークスペースファイル (プロジェクトコンテキスト)

デフォルトでは、OpenClawは固定のワークスペースファイルセットを (存在する場合) 注入します：

-   `AGENTS.md`
-   `SOUL.md`
-   `TOOLS.md`
-   `IDENTITY.md`
-   `USER.md`
-   `HEARTBEAT.md`
-   `BOOTSTRAP.md` (初回実行のみ)

大きなファイルは、`agents.defaults.bootstrapMaxChars` (デフォルト `20000` 文字) を使用してファイルごとに切り詰められます。OpenClawはまた、`agents.defaults.bootstrapTotalMaxChars` (デフォルト `150000` 文字) でファイル全体の合計ブートストラップ注入上限を強制します。`/context` は**生サイズと注入サイズ**、および切り詰めが発生したかどうかを表示します。切り詰めが発生した場合、ランタイムはプロジェクトコンテキストの下にプロンプト内警告ブロックを注入できます。これは `agents.defaults.bootstrapPromptTruncationWarning` (`off`, `once`, `always`; デフォルト `once`) で設定します。

## スキル: 注入されるものとオンデマンドで読み込まれるもの

システムプロンプトにはコンパクトな**スキルリスト** (名前 + 説明 + 場所) が含まれます。このリストには実際のオーバーヘッドがあります。スキルの指示はデフォルトでは含まれ*ません*。モデルは、必要に応じてスキルの `SKILL.md` を `read` することが期待されています。

## ツール: 2つのコストがある

ツールは2つの方法でコンテキストに影響します：

1.  **ツールリストテキスト** (システムプロンプト内の「ツーリング」として表示されるもの)。
2.  **ツールスキーマ** (JSON)。これらはモデルがツールを呼び出せるように送信されます。これらはプレーンテキストとして表示されませんが、コンテキストにカウントされます。

`/context detail` は最大のツールスキーマを内訳するので、何が支配的か確認できます。

## コマンド、ディレクティブ、および「インラインショートカット」

スラッシュコマンドはゲートウェイによって処理されます。いくつかの異なる動作があります：

-   **スタンドアロンコマンド**: `/...` のみのメッセージはコマンドとして実行されます。
-   **ディレクティブ**: `/think`, `/verbose`, `/reasoning`, `/elevated`, `/model`, `/queue` は、モデルがメッセージを見る前に取り除かれます。
    -   ディレクティブのみのメッセージはセッション設定を保持します。
    -   通常のメッセージ内のインラインディレクティブは、メッセージごとのヒントとして機能します。
-   **インラインショートカット** (許可された送信者のみ): 通常のメッセージ内の特定の `/...` トークンは即座に実行でき (例: 「hey /status」)、モデルが残りのテキストを見る前に取り除かれます。

詳細: [スラッシュコマンド](../tools/slash-commands.md)。

## セッション、コンパクション、およびプルーニング (何が永続するか)

メッセージ間で何が永続するかは、メカニズムによって異なります：

-   **通常の履歴**は、ポリシーによってコンパクト化/プルーニングされるまで、セッショントランスクリプト内に永続します。
-   **コンパクション**は要約をトランスクリプトに永続させ、最近のメッセージはそのまま保持します。
-   **プルーニング**は、実行のための*メモリ内*プロンプトから古いツール結果を削除しますが、トランスクリプトを書き換えません。

ドキュメント: [セッション](./session.md), [コンパクション](./compaction.md), [セッションプルーニング](./session-pruning.md)。デフォルトでは、OpenClawは組み込みの `legacy` コンテキストエンジンをアセンブリとコンパクションに使用します。`kind: "context-engine"` を提供するプラグインをインストールし、`plugins.slots.contextEngine` でそれを選択した場合、OpenClawはコンテキストアセンブリ、`/compact`、および関連するサブエージェントコンテキストライフサイクルフックをそのエンジンに委譲します。

## /context が実際に報告するもの

`/context` は、利用可能な場合、最新の**実行構築済み**システムプロンプトレポートを優先します：

-   `System prompt (run)` = 最後の組み込み (ツール対応) 実行からキャプチャされ、セッションストアに永続化されたもの。
-   `System prompt (estimate)` = 実行レポートが存在しない場合 (またはレポートを生成しないCLIバックエンド経由で実行している場合) にその場で計算されたもの。

いずれの場合も、サイズと主要な貢献要素を報告します。**完全なシステムプロンプトやツールスキーマをダンプすることはありません**。

[システムプロンプト](./system-prompt.md)[エージェントワークスペース](./agent-workspace.md)