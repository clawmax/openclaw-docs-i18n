

  基础

  
# Agent 运行时

OpenClaw 运行一个源自 **pi-mono** 的嵌入式 agent 运行时。

## 工作空间 (必需)

OpenClaw 使用一个单一的 agent 工作空间目录 (`agents.defaults.workspace`) 作为 agent 工具和上下文的**唯一**工作目录 (`cwd`)。推荐：使用 `openclaw setup` 来创建 `~/.openclaw/openclaw.json`（如果缺失）并初始化工作空间文件。完整的工作空间布局 + 备份指南：[Agent 工作空间](./agent-workspace.md) 如果启用了 `agents.defaults.sandbox`，非主会话可以通过 `agents.defaults.sandbox.workspaceRoot` 下的每个会话工作空间覆盖此设置（参见 [Gateway 配置](../gateway/configuration.md)）。

## 引导文件 (注入)

在 `agents.defaults.workspace` 内，OpenClaw 期望存在以下用户可编辑的文件：

-   `AGENTS.md` — 操作说明 + “记忆”
-   `SOUL.md` — 角色设定、边界、语气
-   `TOOLS.md` — 用户维护的工具笔记（例如 `imsg`、`sag`、约定）
-   `BOOTSTRAP.md` — 一次性首次运行仪式（完成后删除）
-   `IDENTITY.md` — agent 名称/风格/表情符号
-   `USER.md` — 用户资料 + 首选称呼

在新会话的第一个回合，OpenClaw 会将这些文件的内容直接注入到 agent 上下文中。空白文件会被跳过。大文件会被修剪和截断，并带有标记，以保持提示的精简（阅读文件以获取完整内容）。如果文件缺失，OpenClaw 会注入一个单一的“文件缺失”标记行（并且 `openclaw setup` 将创建一个安全的默认模板）。`BOOTSTRAP.md` 仅针对**全新的工作空间**创建（没有其他引导文件存在）。如果您在完成仪式后删除它，它不应在后续重启时重新创建。要完全禁用引导文件创建（针对预置种子的工作空间），请设置：

```json
{ agent: { skipBootstrap: true } }
```

## 内置工具

核心工具（读/执行/编辑/写及相关系统工具）始终可用，但受工具策略约束。`apply_patch` 是可选的，由 `tools.exec.applyPatch` 控制。`TOOLS.md` **不**控制哪些工具存在；它是关于*您*希望如何使用这些工具的指导。

## 技能

OpenClaw 从三个位置加载技能（工作空间在名称冲突时优先）：

-   捆绑的（随安装包提供）
-   托管/本地的：`~/.openclaw/skills`
-   工作空间的：`/skills`

技能可以通过配置/环境进行门控（参见 [Gateway 配置](../gateway/configuration.md) 中的 `skills`）。

## pi-mono 集成

OpenClaw 重用了 pi-mono 代码库的部分内容（模型/工具），但**会话管理、发现和工具连接由 OpenClaw 自行管理**。

-   没有 pi-coding agent 运行时。
-   不参考 `~/.pi/agent` 或 `/.pi` 设置。

## 会话

会话记录以 JSONL 格式存储在：

-   `~/.openclaw/agents//sessions/.jsonl`

会话 ID 是稳定的，由 OpenClaw 选择。旧版 Pi/Tau 会话文件夹**不**被读取。

## 流式传输期间的转向

当队列模式为 `steer` 时，传入的消息会被注入到当前运行中。队列在**每次工具调用后**被检查；如果存在排队的消息，则跳过当前助手消息的剩余工具调用（工具结果报错“Skipped due to queued user message.”），然后在下一个助手响应之前注入排队的用户消息。当队列模式为 `followup` 或 `collect` 时，传入的消息会被保留，直到当前回合结束，然后 agent 开始新的回合，处理排队的负载。有关模式 + 防抖/上限行为，请参见 [队列](./queue.md)。块流式传输会在助手块完成后立即发送；它**默认关闭** (`agents.defaults.blockStreamingDefault: "off"`)。通过 `agents.defaults.blockStreamingBreak` 调整边界（`text_end` 与 `message_end`；默认为 text_end）。通过 `agents.defaults.blockStreamingChunk` 控制软块分块（默认为 800–1200 字符；优先段落分隔，然后是换行符；句子最后）。使用 `agents.defaults.blockStreamingCoalesce` 合并流式块以减少单行垃圾信息（发送前基于空闲时间的合并）。非 Telegram 频道需要显式设置 `*.blockStreaming: true` 才能启用块回复。详细的工具摘要会在工具开始时发出（无防抖）；控制 UI 在可用时通过 agent 事件流式传输工具输出。更多详情：[流式传输 + 分块](./streaming.md)。

## 模型引用

配置中的模型引用（例如 `agents.defaults.model` 和 `agents.defaults.models`）通过分割**第一个** `/` 来解析。

-   配置模型时使用 `provider/model`。
-   如果模型 ID 本身包含 `/`（OpenRouter 风格），请包含提供商前缀（例如：`openrouter/moonshotai/kimi-k2`）。
-   如果省略提供商，OpenClaw 会将输入视为**默认提供商**的别名或模型（仅当模型 ID 中没有 `/` 时有效）。

## 配置 (最小化)

至少设置：

-   `agents.defaults.workspace`
-   `channels.whatsapp.allowFrom`（强烈推荐）

* * *

*下一步：[群组聊天](../channels/group-messages.md)* 🦞

[Gateway 架构](./architecture.md)[Agent 循环](./agent-loop.md)

---