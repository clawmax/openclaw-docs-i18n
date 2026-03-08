

  基础概念

  
# 上下文

“上下文”是 **OpenClaw 为一次运行发送给模型的所有内容**。它受模型的 **上下文窗口**（令牌限制）约束。初学者心智模型：

-   **系统提示词**（OpenClaw 构建）：规则、工具、技能列表、时间/运行时信息，以及注入的工作区文件。
-   **对话历史记录**：当前会话中你的消息和助手消息。
-   **工具调用/结果 + 附件**：命令输出、文件读取、图像/音频等。

上下文*不同于*“记忆”：记忆可以存储在磁盘上并在以后重新加载；上下文是模型当前窗口内的内容。

## 快速开始（检查上下文）

-   `/status` → 快速查看“我的窗口有多满？” + 会话设置。
-   `/context list` → 查看注入的内容及其大致大小（每个文件 + 总计）。
-   `/context detail` → 更详细的分析：每个文件、每个工具模式大小、每个技能条目大小以及系统提示词大小。
-   `/usage tokens` → 在正常回复后附加每次回复的用量页脚。
-   `/compact` → 将较早的历史记录总结为一个压缩条目以释放窗口空间。

另请参阅：[斜杠命令](../tools/slash-commands.md)、[令牌使用与成本](../reference/token-use.md)、[压缩](./compaction.md)。

## 示例输出

具体值因模型、提供商、工具策略以及工作区内容而异。

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

## 哪些内容计入上下文窗口

模型接收到的所有内容都计入，包括：

-   系统提示词（所有部分）。
-   对话历史记录。
-   工具调用 + 工具结果。
-   附件/转录文本（图像/音频/文件）。
-   压缩摘要和修剪产物。
-   提供商的“包装器”或隐藏头部（不可见，但仍被计入）。

## OpenClaw 如何构建系统提示词

系统提示词是 **OpenClaw 所有**的，并在每次运行时重建。它包括：

-   工具列表 + 简短描述。
-   技能列表（仅元数据；见下文）。
-   工作区位置。
-   时间（UTC + 如果配置了用户时间则转换）。
-   运行时元数据（主机/操作系统/模型/思考模式）。
-   在 **项目上下文** 下注入的工作区引导文件。

完整分析：[系统提示词](./system-prompt.md)。

## 注入的工作区文件（项目上下文）

默认情况下，OpenClaw 会注入一组固定的工作区文件（如果存在）：

-   `AGENTS.md`
-   `SOUL.md`
-   `TOOLS.md`
-   `IDENTITY.md`
-   `USER.md`
-   `HEARTBEAT.md`
-   `BOOTSTRAP.md`（仅首次运行）

大文件会根据 `agents.defaults.bootstrapMaxChars`（默认 `20000` 字符）按文件进行截断。OpenClaw 还通过 `agents.defaults.bootstrapTotalMaxChars`（默认 `150000` 字符）对所有文件的总引导注入量设置了上限。`/context` 显示 **原始大小与注入大小** 以及是否发生了截断。当发生截断时，运行时可以在项目上下文下注入一个提示词内的警告块。通过 `agents.defaults.bootstrapPromptTruncationWarning`（`off`、`once`、`always`；默认 `once`）配置此行为。

## 技能：哪些被注入，哪些按需加载

系统提示词包含一个紧凑的 **技能列表**（名称 + 描述 + 位置）。此列表有实际开销。默认情况下，*不包含*技能指令。模型被期望在**仅当需要时**才去 `read` 技能的 `SKILL.md` 文件。

## 工具：存在两种成本

工具以两种方式影响上下文：

1.  **工具列表文本** 在系统提示词中（你看到的“工具”部分）。
2.  **工具模式**（JSON）。这些被发送给模型以便它可以调用工具。即使你无法将它们视为纯文本，它们也会计入上下文。

`/context detail` 会分解最大的工具模式，以便你了解哪些占主导地位。

## 命令、指令和“内联快捷方式”

斜杠命令由网关处理。有几种不同的行为：

-   **独立命令**：仅包含 `/...` 的消息将作为命令运行。
-   **指令**：`/think`、`/verbose`、`/reasoning`、`/elevated`、`/model`、`/queue` 在模型看到消息之前被剥离。
    -   仅包含指令的消息会持久化会话设置。
    -   正常消息中的内联指令充当每条消息的提示。
-   **内联快捷方式**（仅限允许列表中的发送者）：正常消息中的某些 `/...` 标记可以立即运行（例如：“hey /status”），并且在模型看到剩余文本之前被剥离。

详情：[斜杠命令](../tools/slash-commands.md)。

## 会话、压缩和修剪（哪些内容会持久化）

哪些内容在消息之间持久化取决于机制：

-   **正常历史记录** 会持久化在会话记录中，直到被策略压缩/修剪。
-   **压缩** 将摘要持久化到记录中，并保持最近的消息完整。
-   **修剪** 从运行的*内存中*提示词中移除旧的工具结果，但不会重写记录。

文档：[会话](./session.md)、[压缩](./compaction.md)、[会话修剪](./session-pruning.md)。默认情况下，OpenClaw 使用内置的 `legacy` 上下文引擎进行组装和压缩。如果你安装了一个提供 `kind: "context-engine"` 的插件，并通过 `plugins.slots.contextEngine` 选择它，OpenClaw 会将上下文组装、`/compact` 以及相关的子代理上下文生命周期钩子委托给该引擎。

## /context 实际报告什么

`/context` 在可用时优先使用最新的 **运行构建的** 系统提示词报告：

-   `System prompt (run)` = 从上次嵌入式（支持工具的）运行中捕获并持久化在会话存储中。
-   `System prompt (estimate)` = 当不存在运行报告时（或通过不生成报告的 CLI 后端运行时）动态计算。

无论哪种方式，它都会报告大小和主要贡献者；它**不会**转储完整的系统提示词或工具模式。

[系统提示词](./system-prompt.md)[代理工作区](./agent-workspace.md)