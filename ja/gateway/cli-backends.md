

  プロトコルとAPI

  
# CLI バックエンド

OpenClawは、APIプロバイダーがダウンしている、レート制限されている、または一時的に動作不良の際に、**ローカルAI CLI**を**テキスト専用フォールバック**として実行できます。これは意図的に保守的な設計です：

-   **ツールは無効化されます**（ツール呼び出しなし）。
-   **テキスト入力 → テキスト出力**（信頼性重視）。
-   **セッションがサポートされます**（フォローアップのターンでも一貫性を維持）。
-   **画像はパススルー可能**（CLIが画像パスを受け入れる場合）。

これは主要な経路ではなく、**安全策**として設計されています。外部APIに依存せずに「常に動作する」テキスト応答が必要な場合に使用してください。

## 初心者向けクイックスタート

Claude Code CLIは**設定なしで使用可能**です（OpenClawにデフォルト設定が同梱されています）：

```bash
openclaw agent --message "hi" --model claude-cli/opus-4.6
```

Codex CLIもそのまま動作します：

```bash
openclaw agent --message "hi" --model codex-cli/gpt-5.4
```

ゲートウェイがlaunchd/systemd下で実行され、PATHが最小限の場合は、コマンドパスだけを追加します：

```json
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude",
        },
      },
    },
  },
}
```

以上です。CLI自体を超える追加の認証設定やキーは必要ありません。

## フォールバックとして使用する

プライマリモデルが失敗した場合のみCLIバックエンドが実行されるように、フォールバックリストに追加します：

```json
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["claude-cli/opus-4.6", "claude-cli/opus-4.5"],
      },
      models: {
        "anthropic/claude-opus-4-6": { alias: "Opus" },
        "claude-cli/opus-4.6": {},
        "claude-cli/opus-4.5": {},
      },
    },
  },
}
```

注意点：

-   `agents.defaults.models`（許可リスト）を使用する場合は、`claude-cli/...`を含める必要があります。
-   プライマリプロバイダーが失敗した場合（認証、レート制限、タイムアウト）、OpenClawは次にCLIバックエンドを試行します。

## 設定概要

すべてのCLIバックエンドは以下の下に配置されます：

```
agents.defaults.cliBackends
```

各エントリは**プロバイダーID**（例：`claude-cli`、`my-cli`）でキー付けされます。プロバイダーIDはモデル参照の左側になります：

```
<プロバイダー>/<モデル>
```

### 設定例

```json
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude",
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          input: "arg",
          modelArg: "--model",
          modelAliases: {
            "claude-opus-4-6": "opus",
            "claude-opus-4-5": "opus",
            "claude-sonnet-4-5": "sonnet",
          },
          sessionArg: "--session",
          sessionMode: "existing",
          sessionIdFields: ["session_id", "conversation_id"],
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat",
          serialize: true,
        },
      },
    },
  },
}
```

## 動作の仕組み

1.  **バックエンドを選択**：プロバイダープレフィックス（`claude-cli/...`）に基づいて選択。
2.  **システムプロンプトを構築**：同じOpenClawプロンプト + ワークスペースコンテキストを使用。
3.  **CLIを実行**：セッションID（サポートされている場合）を使用して実行し、履歴の一貫性を維持。
4.  **出力を解析**：（JSONまたはプレーンテキスト）最終テキストを抽出して返却。
5.  **セッションIDを永続化**：バックエンドごとに保存し、フォローアップで同じCLIセッションを再利用。

## セッション

-   CLIがセッションをサポートする場合、`sessionArg`（例：`--session-id`）または`sessionArgs`（プレースホルダー `{sessionId}`）を設定します（IDを複数のフラグに挿入する必要がある場合）。
-   CLIが**resumeサブコマンド**を使用し、異なるフラグを持つ場合は、`resumeArgs`（再開時に`args`を置き換え）とオプションで`resumeOutput`（非JSON再開用）を設定します。
-   `sessionMode`：
    -   `always`：常にセッションIDを送信（保存されていない場合は新しいUUID）。
    -   `existing`：以前に保存されたセッションIDがある場合のみ送信。
    -   `none`：セッションIDを送信しない。

## 画像（パススルー）

CLIが画像パスを受け入れる場合は、`imageArg`を設定します：

```yaml
imageArg: "--image",
imageMode: "repeat"
```

OpenClawはbase64画像を一時ファイルに書き込みます。`imageArg`が設定されている場合、それらのパスはCLI引数として渡されます。`imageArg`がない場合、OpenClawはファイルパスをプロンプトに追加します（パスインジェクション）。これは、プレーンパスからローカルファイルを自動ロードするCLI（Claude Code CLIの動作）には十分です。

## 入力 / 出力

-   `output: "json"`（デフォルト）はJSONの解析を試み、テキストとセッションIDを抽出します。
-   `output: "jsonl"`はJSONLストリーム（Codex CLI `--json`）を解析し、最後のエージェントメッセージと存在する場合は`thread_id`を抽出します。
-   `output: "text"`は標準出力を最終応答として扱います。

入力モード：

-   `input: "arg"`（デフォルト）はプロンプトを最後のCLI引数として渡します。
-   `input: "stdin"`はプロンプトを標準入力経由で送信します。
-   プロンプトが非常に長く、`maxPromptArgChars`が設定されている場合は、標準入力が使用されます。

## デフォルト（組み込み）

OpenClawは`claude-cli`用のデフォルトを同梱しています：

-   `command: "claude"`
-   `args: ["-p", "--output-format", "json", "--permission-mode", "bypassPermissions"]`
-   `resumeArgs: ["-p", "--output-format", "json", "--permission-mode", "bypassPermissions", "--resume", "{sessionId}"]`
-   `modelArg: "--model"`
-   `systemPromptArg: "--append-system-prompt"`
-   `sessionArg: "--session-id"`
-   `systemPromptWhen: "first"`
-   `sessionMode: "always"`

OpenClawは`codex-cli`用のデフォルトも同梱しています：

-   `command: "codex"`
-   `args: ["exec","--json","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
-   `resumeArgs: ["exec","resume","{sessionId}","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
-   `output: "jsonl"`
-   `resumeOutput: "text"`
-   `modelArg: "--model"`
-   `imageArg: "--image"`
-   `sessionMode: "existing"`

必要な場合のみ上書きしてください（一般的な例：絶対パスの`command`）。

## 制限事項

-   **OpenClawツールは使用不可**（CLIバックエンドはツール呼び出しを受け取りません）。一部のCLIは独自のエージェントツールを実行する場合があります。
-   **ストリーミング不可**（CLI出力は収集されてから返却されます）。
-   **構造化出力**はCLIのJSON形式に依存します。
-   **Codex CLIセッション**はテキスト出力（JSONLなし）で再開されるため、初期の`--json`実行よりも構造化されていません。OpenClawセッションは通常通り動作します。

## トラブルシューティング

-   **CLIが見つからない**：`command`をフルパスに設定してください。
-   **モデル名が間違っている**：`modelAliases`を使用して`provider/model` → CLIモデルをマッピングしてください。
-   **セッションの継続性がない**：`sessionArg`が設定され、`sessionMode`が`none`でないことを確認してください（Codex CLIは現在、JSON出力で再開できません）。
-   **画像が無視される**：`imageArg`を設定してください（CLIがファイルパスをサポートしていることを確認してください）。

[ツール呼び出しAPI](./tools-invoke-http-api.md)[ローカルモデル](./local-models.md)