

  协议与 API

  
# CLI 后端

当 API 提供商宕机、达到速率限制或暂时出现异常时，OpenClaw 可以运行**本地 AI CLI** 作为**纯文本回退**方案。这是有意设计的保守方案：

-   **工具被禁用**（无工具调用）。
-   **文本输入 → 文本输出**（可靠）。
-   **支持会话**（因此后续轮次能保持连贯性）。
-   **图像可以透传**（如果 CLI 接受图像路径）。

这被设计为**安全网**，而非主要路径。当您需要“始终可用”的文本响应而不依赖外部 API 时使用它。

## 新手快速入门

您可以使用 Claude Code CLI **无需任何配置**（OpenClaw 内置了默认配置）：

```bash
openclaw agent --message "hi" --model claude-cli/opus-4.6
```

Codex CLI 也可以开箱即用：

```bash
openclaw agent --message "hi" --model codex-cli/gpt-5.4
```

如果您的网关在 launchd/systemd 下运行且 PATH 环境变量有限，只需添加命令路径：

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

就这样。除了 CLI 本身之外，无需密钥或额外的身份验证配置。

## 将其用作回退方案

将 CLI 后端添加到您的回退列表中，使其仅在主要模型失败时运行：

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

注意：

-   如果您使用 `agents.defaults.models`（允许列表），则必须包含 `claude-cli/...`。
-   如果主要提供商失败（身份验证、速率限制、超时），OpenClaw 将尝试 CLI 后端。

## 配置概述

所有 CLI 后端都位于：

```
agents.defaults.cliBackends
```

每个条目由一个**提供商 ID**（例如 `claude-cli`、`my-cli`）作为键。该提供商 ID 成为您模型引用的左侧部分：

```
<provider>/<model>
```

### 配置示例

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

## 工作原理

1.  **选择后端**：基于提供商前缀（`claude-cli/...`）。
2.  **构建系统提示**：使用相同的 OpenClaw 提示 + 工作区上下文。
3.  **执行 CLI**：附带会话 ID（如果支持），以保持历史记录的一致性。
4.  **解析输出**（JSON 或纯文本）并返回最终文本。
5.  **持久化会话 ID**：每个后端独立存储，以便后续对话重用相同的 CLI 会话。

## 会话

-   如果 CLI 支持会话，请设置 `sessionArg`（例如 `--session-id`）或 `sessionArgs`（占位符 `{sessionId}`），当 ID 需要插入到多个标志中时使用后者。
-   如果 CLI 使用**恢复子命令**且标志不同，请设置 `resumeArgs`（恢复时替换 `args`）和可选的 `resumeOutput`（用于非 JSON 恢复）。
-   `sessionMode`：
    -   `always`：始终发送会话 ID（如果未存储则使用新的 UUID）。
    -   `existing`：仅当之前存储过会话 ID 时才发送。
    -   `none`：从不发送会话 ID。

## 图像（透传）

如果您的 CLI 接受图像路径，请设置 `imageArg`：

```yaml
imageArg: "--image",
imageMode: "repeat"
```

OpenClaw 会将 base64 图像写入临时文件。如果设置了 `imageArg`，这些路径将作为 CLI 参数传递。如果缺少 `imageArg`，OpenClaw 会将文件路径附加到提示中（路径注入），这对于自动从纯路径加载本地文件的 CLI 来说已经足够（Claude Code CLI 的行为）。

## 输入 / 输出

-   `output: "json"`（默认）尝试解析 JSON 并提取文本 + 会话 ID。
-   `output: "jsonl"` 解析 JSONL 流（Codex CLI 的 `--json`）并提取最后一条代理消息以及存在的 `thread_id`。
-   `output: "text"` 将 stdout 视为最终响应。

输入模式：

-   `input: "arg"`（默认）将提示作为最后一个 CLI 参数传递。
-   `input: "stdin"` 通过 stdin 发送提示。
-   如果提示非常长且设置了 `maxPromptArgChars`，则使用 stdin。

## 默认配置（内置）

OpenClaw 为 `claude-cli` 提供了默认配置：

-   `command: "claude"`
-   `args: ["-p", "--output-format", "json", "--permission-mode", "bypassPermissions"]`
-   `resumeArgs: ["-p", "--output-format", "json", "--permission-mode", "bypassPermissions", "--resume", "{sessionId}"]`
-   `modelArg: "--model"`
-   `systemPromptArg: "--append-system-prompt"`
-   `sessionArg: "--session-id"`
-   `systemPromptWhen: "first"`
-   `sessionMode: "always"`

OpenClaw 也为 `codex-cli` 提供了默认配置：

-   `command: "codex"`
-   `args: ["exec","--json","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
-   `resumeArgs: ["exec","resume","{sessionId}","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
-   `output: "jsonl"`
-   `resumeOutput: "text"`
-   `modelArg: "--model"`
-   `imageArg: "--image"`
-   `sessionMode: "existing"`

仅在需要时覆盖（常见情况：绝对 `command` 路径）。

## 限制

-   **无 OpenClaw 工具**（CLI 后端永远不会接收工具调用）。某些 CLI 可能仍会运行其自身的代理工具。
-   **无流式传输**（CLI 输出被收集后返回）。
-   **结构化输出**取决于 CLI 的 JSON 格式。
-   **Codex CLI 会话**通过文本输出恢复（无 JSONL），这比初始的 `--json` 运行结构化程度低。OpenClaw 会话仍正常工作。

## 故障排除

-   **找不到 CLI**：将 `command` 设置为完整路径。
-   **错误的模型名称**：使用 `modelAliases` 将 `provider/model` 映射到 CLI 模型。
-   **无会话连续性**：确保设置了 `sessionArg` 且 `sessionMode` 不是 `none`（Codex CLI 目前无法使用 JSON 输出恢复）。
-   **图像被忽略**：设置 `imageArg`（并验证 CLI 支持文件路径）。

[工具调用 API](./tools-invoke-http-api.md)[本地模型](./local-models.md)