

  内置工具

  
# Lobster

Lobster 是一个工作流 Shell，它允许 OpenClaw 将多步骤工具序列作为一次确定性的操作运行，并包含明确的审批检查点。

## 核心理念

您的助手可以构建管理自身的工具。请求一个工作流，30 分钟后您就拥有了一个 CLI 和可以单次调用运行的管道。Lobster 就是缺失的那一环：确定性管道、明确审批和可恢复状态。

## 为什么需要它

如今，复杂的工作流需要多次来回的工具调用。每次调用都消耗 Token，并且 LLM 必须编排每个步骤。Lobster 将这种编排移入一个类型化的运行时：

-   **一次调用替代多次调用**：OpenClaw 运行一次 Lobster 工具调用并获得结构化结果。
-   **内置审批**：副作用（发送邮件、发布评论）会暂停工作流，直到获得明确批准。
-   **可恢复**：暂停的工作流返回一个令牌；批准后可以恢复，无需重新运行所有内容。

## 为什么使用 DSL 而不是普通程序？

Lobster 有意设计得很小巧。目标不是“创造一门新语言”，而是创建一个可预测、对 AI 友好的管道规范，并内置一流的审批和恢复令牌功能。

-   **审批/恢复是内置的**：普通程序可以提示用户，但无法*暂停并使用持久令牌恢复*，除非您自己实现这个运行时。
-   **确定性 + 可审计性**：管道是数据，因此易于记录、比较、重放和审查。
-   **为 AI 约束接口**：微小的语法 + JSON 管道减少了“创造性”代码路径，使验证变得切实可行。
-   **内建安全策略**：超时、输出限制、沙箱检查和允许列表由运行时强制执行，而不是每个脚本。
-   **仍然可编程**：每个步骤可以调用任何 CLI 或脚本。如果您需要 JS/TS，可以从代码生成 `.lobster` 文件。

## 工作原理

OpenClaw 在**工具模式**下启动本地 `lobster` CLI，并从 stdout 解析 JSON 信封。如果管道因等待审批而暂停，该工具会返回一个 `resumeToken`，以便您稍后继续。

## 模式：小型 CLI + JSON 管道 + 审批

构建输出 JSON 的小型命令，然后将它们链接到一次 Lobster 调用中。（以下示例命令名称 — 可替换为您自己的。）

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

如果管道请求审批，使用令牌恢复：

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

AI 触发工作流；Lobster 执行步骤。审批关卡使副作用明确且可审计。示例：将输入项映射到工具调用：

```
gog.gmail.search --query 'newer_than:1d' \
  | openclaw.invoke --tool message --action send --each --item-key message --args-json '{"provider":"telegram","to":"..."}'
```

## 纯 JSON 的 LLM 步骤 (llm-task)

对于需要**结构化 LLM 步骤**的工作流，启用可选的 `llm-task` 插件工具并从 Lobster 调用它。这可以在保持工作流确定性的同时，仍然让您使用模型进行分类/总结/起草。启用该工具：

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

在管道中使用它：

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

有关详细信息和配置选项，请参阅 [LLM 任务](./llm-task.md)。

## 工作流文件 (.lobster)

Lobster 可以运行包含 `name`、`args`、`steps`、`env`、`condition` 和 `approval` 字段的 YAML/JSON 工作流文件。在 OpenClaw 工具调用中，将 `pipeline` 设置为文件路径。

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

注意：

-   `stdin: $step.stdout` 和 `stdin: $step.json` 传递前一个步骤的输出。
-   `condition`（或 `when`）可以根据 `$step.approved` 来条件化执行步骤。

## 安装 Lobster

在运行 OpenClaw Gateway 的**同一主机**上安装 Lobster CLI（参见 [Lobster 仓库](https://github.com/openclaw/lobster)），并确保 `lobster` 在 `PATH` 中。

## 启用该工具

Lobster 是一个**可选**的插件工具（默认未启用）。推荐配置（附加式，安全）：

```json
{
  "tools": {
    "alsoAllow": ["lobster"]
  }
}
```

或按代理配置：

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

除非您打算在限制性允许列表模式下运行，否则请避免使用 `tools.allow: ["lobster"]`。注意：对于可选插件，允许列表是选择加入的。如果您的允许列表只列出了插件工具（如 `lobster`），OpenClaw 会保持核心工具启用。要限制核心工具，请在允许列表中也包含您想要的核心工具或组。

## 示例：邮件分类处理

没有 Lobster 时：

```
用户："检查我的邮件并起草回复"
→ openclaw 调用 gmail.list
→ LLM 总结
→ 用户："为 #2 和 #5 起草回复"
→ LLM 起草
→ 用户："发送 #2"
→ openclaw 调用 gmail.send
（每天重复，不记得处理了哪些邮件）
```

使用 Lobster：

```json
{
  "action": "run",
  "pipeline": "email.triage --limit 20",
  "timeoutMs": 30000
}
```

返回一个 JSON 信封（已截断）：

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

用户批准 → 恢复：

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

一个工作流。确定性的。安全的。

## 工具参数

### run

在工具模式下运行管道。

```json
{
  "action": "run",
  "pipeline": "gog.gmail.search --query 'newer_than:1d' | email.triage",
  "cwd": "workspace",
  "timeoutMs": 30000,
  "maxStdoutBytes": 512000
}
```

使用参数运行工作流文件：

```json
{
  "action": "run",
  "pipeline": "/path/to/inbox-triage.lobster",
  "argsJson": "{\"tag\":\"family\"}"
}
```

### resume

在审批后继续暂停的工作流。

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

### 可选输入

-   `cwd`：管道的相对工作目录（必须保持在当前进程工作目录内）。
-   `timeoutMs`：如果子进程超过此持续时间则终止（默认：20000）。
-   `maxStdoutBytes`：如果 stdout 超过此大小则终止子进程（默认：512000）。
-   `argsJson`：传递给 `lobster run --args-json` 的 JSON 字符串（仅适用于工作流文件）。

## 输出信封

Lobster 返回一个 JSON 信封，包含以下三种状态之一：

-   `ok` → 成功完成
-   `needs_approval` → 已暂停；需要 `requiresApproval.resumeToken` 来恢复
-   `cancelled` → 明确拒绝或取消

该工具在 `content`（美化 JSON）和 `details`（原始对象）中都展示此信封。

## 审批

如果存在 `requiresApproval`，检查提示并决定：

-   `approve: true` → 恢复并继续执行副作用
-   `approve: false` → 取消并最终化工作流

使用 `approve --preview-from-stdin --limit N` 将 JSON 预览附加到审批请求，而无需自定义 jq/heredoc 胶水代码。恢复令牌现在是紧凑的：Lobster 在其状态目录下存储工作流恢复状态，并返回一个小的令牌密钥。

## OpenProse

OpenProse 与 Lobster 配合良好：使用 `/prose` 编排多代理准备，然后运行 Lobster 管道进行确定性审批。如果 Prose 程序需要 Lobster，可以通过 `tools.subagents.tools` 为子代理允许 `lobster` 工具。参见 [OpenProse](../prose.md)。

## 安全性

-   **仅限本地子进程** — 插件本身不进行网络调用。
-   **无密钥管理** — Lobster 不管理 OAuth；它调用执行此操作的 OpenClaw 工具。
-   **沙箱感知** — 当工具上下文处于沙箱中时被禁用。
-   **强化** — 固定的可执行文件名 (`lobster`) 位于 `PATH` 中；强制执行超时和输出限制。

## 故障排除

-   **`lobster 子进程超时`** → 增加 `timeoutMs`，或拆分长管道。
-   **`lobster 输出超过 maxStdoutBytes`** → 提高 `maxStdoutBytes` 或减少输出大小。
-   **`lobster 返回无效 JSON`** → 确保管道在工具模式下运行且仅打印 JSON。
-   **`lobster 失败（代码 …）`** → 在终端中运行相同的管道以检查 stderr。

## 了解更多

-   [插件](./plugin.md)
-   [插件工具开发](../plugins/agent-tools.md)

## 案例研究：社区工作流

一个公开示例：一个“第二大脑”CLI + Lobster 管道，用于管理三个 Markdown 知识库（个人、伴侣、共享）。该 CLI 输出 JSON 用于统计、收件箱列表和过时扫描；Lobster 将这些命令链接到工作流中，如 `weekly-review`、`inbox-triage`、`memory-consolidation` 和 `shared-task-sync`，每个都带有审批关卡。AI 在可用时处理判断（分类），在不可用时回退到确定性规则。

-   讨论串：[https://x.com/plattenschieber/status/2014508656335770033](https://x.com/plattenschieber/status/2014508656335770033)
-   仓库：[https://github.com/bloomedai/brain-cli](https://github.com/bloomedai/brain-cli)

[LLM 任务](./llm-task.md)[工具循环检测](./loop-detection.md)