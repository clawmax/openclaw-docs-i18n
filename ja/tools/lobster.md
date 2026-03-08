

  組み込みツール

  
# Lobster

Lobster は、OpenClaw が明示的な承認チェックポイントを持つ単一の決定論的操作として、マルチステップのツールシーケンスを実行できるようにするワークフローシェルです。

## フック

あなたのアシスタントは、自分自身を管理するツールを構築できます。ワークフローを要求すると、30分後には1回の呼び出しで実行されるCLIとパイプラインが手に入ります。Lobster はその欠けていたピースです：決定論的パイプライン、明示的な承認、再開可能な状態。

## なぜ必要なのか

現在、複雑なワークフローには多くの双方向のツール呼び出しが必要です。各呼び出しはトークンを消費し、LLMはすべてのステップをオーケストレーションしなければなりません。Lobster はそのオーケストレーションを型付きランタイムに移します：

-   **多数ではなく1回の呼び出し**: OpenClaw は1回の Lobster ツール呼び出しを実行し、構造化された結果を得ます。
-   **組み込みの承認**: 副作用（メール送信、コメント投稿）は、明示的に承認されるまでワークフローを停止させます。
-   **再開可能**: 停止したワークフローはトークンを返します；すべてを再実行することなく、承認して再開できます。

## なぜプレーンなプログラムではなくDSLなのか？

Lobster は意図的に小さく設計されています。目標は「新しい言語」ではなく、第一級の承認と再開トークンを備えた、予測可能でAIに優しいパイプライン仕様です。

-   **承認/再開が組み込まれている**: 通常のプログラムは人間にプロンプトを出すことはできますが、永続的なトークンで*一時停止と再開*することは、自分でそのランタイムを発明しない限りできません。
-   **決定性 + 監査可能性**: パイプラインはデータなので、ログ記録、差分比較、再生、レビューが容易です。
-   **AIのための制約された表面**: 小さな文法 + JSON パイプにより、「創造的」なコードパスが減り、検証が現実的になります。
-   **安全性ポリシーが組み込まれている**: タイムアウト、出力制限、サンドボックスチェック、許可リストは、各スクリプトではなくランタイムによって強制されます。
-   **それでもプログラマブル**: 各ステップは任意のCLIやスクリプトを呼び出せます。JS/TSが欲しい場合は、コードから `.lobster` ファイルを生成できます。

## 仕組み

OpenClaw はローカルの `lobster` CLI を**ツールモード**で起動し、標準出力からJSONエンベロープを解析します。パイプラインが承認のために一時停止した場合、ツールは後で続行できるように `resumeToken` を返します。

## パターン: 小さなCLI + JSONパイプ + 承認

JSONを話す小さなコマンドを構築し、それらを単一の Lobster 呼び出しに連結します。（以下のコマンド名は例です — 独自のものに置き換えてください。）

```bash
inbox list --json
inbox categorize --json
inbox apply --json
```

```json
{
  "action": "run",
  "pipeline": "exec --json --shell 'inbox list --json' | exec --stdin json --shell 'inbox categorize --json' | exec --stdin json --shell 'inbox apply --json' | approve --preview-from-stdin --limit 5 --prompt 'Apply changes?'",
  "timeoutMs": 30000
}
```

パイプラインが承認を要求した場合、トークンを使って再開します：

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

AIがワークフローをトリガーし、Lobsterがステップを実行します。承認ゲートにより、副作用が明示的かつ監査可能になります。例：入力アイテムをツール呼び出しにマッピング：

```
gog.gmail.search --query 'newer_than:1d' \
  | openclaw.invoke --tool message --action send --each --item-key message --args-json '{"provider":"telegram","to":"..."}'
```

## JSONのみのLLMステップ (llm-task)

**構造化されたLLMステップ**が必要なワークフローの場合は、オプションの `llm-task` プラグインツールを有効にし、Lobsterから呼び出します。これにより、ワークフローを決定論的に保ちながら、モデルを使って分類/要約/下書きを作成できます。ツールを有効にします：

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  },
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": { "allow": ["llm-task"] }
      }
    ]
  }
}
```

パイプラインで使用します：

```
openclaw.invoke --tool llm-task --action json --args-json '{
  "prompt": "Given the input email, return intent and draft.",
  "input": { "subject": "Hello", "body": "Can you help?" },
  "schema": {
    "type": "object",
    "properties": {
      "intent": { "type": "string" },
      "draft": { "type": "string" }
    },
    "required": ["intent", "draft"],
    "additionalProperties": false
  }
}'
```

詳細と設定オプションについては、[LLM Task](./llm-task.md) を参照してください。

## ワークフローファイル (.lobster)

Lobster は `name`、`args`、`steps`、`env`、`condition`、`approval` フィールドを持つYAML/JSONワークフローファイルを実行できます。OpenClaw ツール呼び出しでは、`pipeline` をファイルパスに設定します。

```yaml
name: inbox-triage
args:
  tag:
    default: "family"
steps:
  - id: collect
    command: inbox list --json
  - id: categorize
    command: inbox categorize --json
    stdin: $collect.stdout
  - id: approve
    command: inbox apply --approve
    stdin: $categorize.stdout
    approval: required
  - id: execute
    command: inbox apply --execute
    stdin: $categorize.stdout
    condition: $approve.approved
```

注記：

-   `stdin: $step.stdout` と `stdin: $step.json` は、前のステップの出力を渡します。
-   `condition` (または `when`) は、`$step.approved` に基づいてステップをゲートできます。

## Lobster のインストール

**OpenClaw Gateway を実行するのと同じホスト**に Lobster CLI をインストールし（[Lobster リポジトリ](https://github.com/openclaw/lobster)を参照）、`lobster` が `PATH` 上にあることを確認してください。

## ツールの有効化

Lobster は**オプション**のプラグインツールです（デフォルトでは有効になっていません）。推奨（追加的、安全）：

```json
{
  "tools": {
    "alsoAllow": ["lobster"]
  }
}
```

またはエージェントごとに：

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": {
          "alsoAllow": ["lobster"]
        }
      }
    ]
  }
}
```

制限的な許可リストモードで実行する意図がない限り、`tools.allow: ["lobster"]` の使用は避けてください。注記：許可リストはオプションプラグインに対してオプトインです。許可リストがプラグインツール（`lobster` など）のみを指定している場合、OpenClaw はコアツールを有効に保ちます。コアツールを制限するには、許可リストに必要なコアツールやグループも含めてください。

## 例: メールトリアージ

Lobster なし：

```
ユーザー: 「私のメールをチェックして返信を下書きして」
→ openclaw が gmail.list を呼び出し
→ LLM が要約
→ ユーザー: 「#2 と #5 に返信を下書きして」
→ LLM が下書き
→ ユーザー: 「#2 を送信して」
→ openclaw が gmail.send を呼び出し
(毎日繰り返し、何がトリアージされたかの記憶なし)
```

Lobster あり：

```json
{
  "action": "run",
  "pipeline": "email.triage --limit 20",
  "timeoutMs": 30000
}
```

JSONエンベロープを返します（切り詰め）：

```json
{
  "ok": true,
  "status": "needs_approval",
  "output": [{ "summary": "5 need replies, 2 need action" }],
  "requiresApproval": {
    "type": "approval_request",
    "prompt": "Send 2 draft replies?",
    "items": [],
    "resumeToken": "..."
  }
}
```

ユーザーが承認 → 再開：

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

一つのワークフロー。決定論的。安全。

## ツールパラメータ

### run

ツールモードでパイプラインを実行します。

```json
{
  "action": "run",
  "pipeline": "gog.gmail.search --query 'newer_than:1d' | email.triage",
  "cwd": "workspace",
  "timeoutMs": 30000,
  "maxStdoutBytes": 512000
}
```

引数を指定してワークフローファイルを実行：

```json
{
  "action": "run",
  "pipeline": "/path/to/inbox-triage.lobster",
  "argsJson": "{\"tag\":\"family\"}"
}
```

### resume

承認後に停止したワークフローを続行します。

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

### オプション入力

-   `cwd`: パイプラインの相対作業ディレクトリ（現在のプロセスの作業ディレクトリ内に留まる必要があります）。
-   `timeoutMs`: この期間を超えた場合にサブプロセスを強制終了します（デフォルト: 20000）。
-   `maxStdoutBytes`: 標準出力がこのサイズを超えた場合にサブプロセスを強制終了します（デフォルト: 512000）。
-   `argsJson`: `lobster run --args-json` に渡されるJSON文字列（ワークフローファイルのみ）。

## 出力エンベロープ

Lobster は3つのステータスのいずれかを持つJSONエンベロープを返します：

-   `ok` → 正常に完了
-   `needs_approval` → 一時停止；再開には `requiresApproval.resumeToken` が必要
-   `cancelled` → 明示的に拒否またはキャンセル

ツールはエンベロープを `content`（整形されたJSON）と `details`（生のオブジェクト）の両方に表示します。

## 承認

`requiresApproval` が存在する場合、プロンプトを検査して決定します：

-   `approve: true` → 再開して副作用を続行
-   `approve: false` → キャンセルしてワークフローを終了

カスタムのjq/heredocグルーなしで、承認リクエストにJSONプレビューを添付するには `approve --preview-from-stdin --limit N` を使用します。再開トークンはコンパクトになりました：Lobster はワークフローの再開状態をその状態ディレクトリの下に保存し、小さなトークンキーを返します。

## OpenProse

OpenProse は Lobster とよく組み合わさります：`/prose` を使用してマルチエージェントの準備をオーケストレーションし、決定論的な承認のために Lobster パイプラインを実行します。Prose プログラムが Lobster を必要とする場合、`tools.subagents.tools` を介してサブエージェントに `lobster` ツールを許可します。[OpenProse](../prose.md) を参照してください。

## 安全性

-   **ローカルサブプロセスのみ** — プラグイン自体からのネットワーク呼び出しはありません。
-   **シークレットなし** — Lobster はOAuthを管理しません；それを実行するOpenClawツールを呼び出します。
-   **サンドボックス対応** — ツールコンテキストがサンドボックス化されている場合は無効になります。
-   **強化済み** — `PATH` 上の固定された実行可能ファイル名 (`lobster`)；タイムアウトと出力制限が強制されます。

## トラブルシューティング

-   **`lobster subprocess timed out`** → `timeoutMs` を増やすか、長いパイプラインを分割します。
-   **`lobster output exceeded maxStdoutBytes`** → `maxStdoutBytes` を上げるか、出力サイズを減らします。
-   **`lobster returned invalid JSON`** → パイプラインがツールモードで実行され、JSONのみを出力していることを確認します。
-   **`lobster failed (code …)`** → 同じパイプラインをターミナルで実行して標準エラー出力を検査します。

## 詳細情報

-   [プラグイン](./plugin.md)
-   [プラグインツールの作成](../plugins/agent-tools.md)

## ケーススタディ: コミュニティワークフロー

一つの公開例：3つのMarkdownボールト（個人、パートナー、共有）を管理する「セカンドブレイン」CLI + Lobster パイプライン。CLIは統計、受信箱リスト、古くなったスキャンのJSONを出力し、Lobster はそれらのコマンドを `weekly-review`、`inbox-triage`、`memory-consolidation`、`shared-task-sync` などのワークフローに連結し、それぞれに承認ゲートがあります。AIは利用可能な場合は判断（分類）を処理し、利用できない場合は決定論的なルールにフォールバックします。

-   スレッド: [https://x.com/plattenschieber/status/2014508656335770033](https://x.com/plattenschieber/status/2014508656335770033)
-   リポジトリ: [https://github.com/bloomedai/brain-cli](https://github.com/bloomedai/brain-cli)

[LLM Task](./llm-task.md)[Tool-loop detection](./loop-detection.md)