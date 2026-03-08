

  内置工具

  
# LLM Task

`llm-task` 是一个**可选的插件工具**，它运行一个仅输出 JSON 的 LLM 任务，并返回结构化输出（可选择根据 JSON Schema 进行验证）。这对于 Lobster 等工作流引擎非常理想：你可以添加一个单独的 LLM 步骤，而无需为每个工作流编写自定义的 OpenClaw 代码。

## 启用插件

1.  启用插件：

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  }
}
```

2.  将工具加入允许列表（该工具注册时带有 `optional: true` 属性）：

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

## 配置（可选）

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

`allowedModels` 是一个 `provider/model` 字符串的允许列表。如果设置了此列表，任何不在列表中的请求都将被拒绝。

## 工具参数

-   `prompt` (字符串，必需)
-   `input` (任意类型，可选)
-   `schema` (对象，可选的 JSON Schema)
-   `provider` (字符串，可选)
-   `model` (字符串，可选)
-   `authProfileId` (字符串，可选)
-   `temperature` (数字，可选)
-   `maxTokens` (数字，可选)
-   `timeoutMs` (数字，可选)

## 输出

返回 `details.json`，其中包含解析后的 JSON（并在提供 `schema` 时进行验证）。

## 示例：Lobster 工作流步骤

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

## 安全注意事项

-   该工具**仅输出 JSON**，并指示模型只输出 JSON（没有代码块，没有注释）。
-   此运行不会向模型暴露任何其他工具。
-   除非使用 `schema` 进行验证，否则应将输出视为不可信。
-   在任何具有副作用（发送、发布、执行）的步骤之前，请设置审批环节。

[Firecrawl](./firecrawl.md)[Lobster](./lobster.md)

---