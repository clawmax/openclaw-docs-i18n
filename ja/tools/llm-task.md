

  組み込みツール

  
# LLMタスク

`llm-task`は、JSONのみのLLMタスクを実行し、構造化された出力を返す（オプションでJSONスキーマに対して検証する）**オプションのプラグインツール**です。これはLobsterのようなワークフローエンジンに理想的です：各ワークフローごとにカスタムOpenClawコードを書くことなく、単一のLLMステップを追加できます。

## プラグインを有効にする

1.  プラグインを有効にします：

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  }
}
```

2.  ツールを許可リストに追加します（`optional: true`で登録されています）：

```json
{
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

## 設定（オプション）

```json
{
  "plugins": {
    "entries": {
      "llm-task": {
        "enabled": true,
        "config": {
          "defaultProvider": "openai-codex",
          "defaultModel": "gpt-5.4",
          "defaultAuthProfileId": "main",
          "allowedModels": ["openai-codex/gpt-5.4"],
          "maxTokens": 800,
          "timeoutMs": 30000
        }
      }
    }
  }
}
```

`allowedModels`は、`provider/model`文字列の許可リストです。設定されている場合、リスト外のリクエストは拒否されます。

## ツールパラメータ

-   `prompt` (文字列, 必須)
-   `input` (任意の型, オプション)
-   `schema` (オブジェクト, オプションのJSONスキーマ)
-   `provider` (文字列, オプション)
-   `model` (文字列, オプション)
-   `authProfileId` (文字列, オプション)
-   `temperature` (数値, オプション)
-   `maxTokens` (数値, オプション)
-   `timeoutMs` (数値, オプション)

## 出力

解析されたJSONを含む`details.json`を返します（`schema`が提供された場合はそれに対して検証します）。

## 例：Lobsterワークフローステップ

```
openclaw.invoke --tool llm-task --action json --args-json '{
  "prompt": "Given the input email, return intent and draft.",
  "input": {
    "subject": "Hello",
    "body": "Can you help?"
  },
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

## 安全性に関する注意点

-   このツールは**JSON専用**であり、モデルにJSONのみを出力するよう指示します（コードフェンスや解説は含みません）。
-   この実行では、モデルにツールは公開されません。
-   `schema`で検証しない限り、出力は信頼できないものとして扱ってください。
-   副作用を伴うステップ（送信、投稿、実行）の前に承認を設定してください。

[Firecrawl](./firecrawl.md)[Lobster](./lobster.md)