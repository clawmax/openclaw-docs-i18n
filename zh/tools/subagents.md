

  代理协调

  
# 子代理

子代理是从现有代理运行中派生的后台代理运行。它们在各自的会话（`agent::subagent:`）中运行，并在完成后将结果**通告**回请求者的聊天频道。

## 斜杠命令

使用 `/subagents` 来检查或控制**当前会话**的子代理运行：

-   `/subagents list`
-   `/subagents kill <id|#|all>`
-   `/subagents log <id|#> [limit] [tools]`
-   `/subagents info <id|#>`
-   `/subagents send <id|#> `
-   `/subagents steer <id|#> `
-   `/subagents spawn   [--model ] [--thinking ]`

线程绑定控制：这些命令适用于支持持久线程绑定的频道。参见下面的**支持线程的频道**。

-   `/focus <subagent-label|session-key|session-id|session-label>`
-   `/unfocus`
-   `/agents`
-   `/session idle <duration|off>`
-   `/session max-age <duration|off>`

`/subagents info` 显示运行元数据（状态、时间戳、会话ID、转录路径、清理）。

### 派生行为

`/subagents spawn` 以用户命令（而非内部中继）的方式启动一个后台子代理，并在运行完成时向请求者聊天频道发送一条最终完成更新。

-   派生命令是非阻塞的；它会立即返回一个运行ID。
-   完成后，子代理会向请求者聊天频道通告一条摘要/结果消息。
-   对于手动派生，交付具有弹性：
    -   OpenClaw 首先尝试使用稳定的幂等键进行直接 `agent` 交付。
    -   如果直接交付失败，则回退到队列路由。
    -   如果队列路由仍然不可用，则在最终放弃前会以短指数退避重试通告。
-   到请求者会话的完成交接是运行时生成的内部上下文（非用户编写的文本），包括：
    -   `Result`（`assistant` 回复文本，如果助手回复为空，则为最新的 `toolResult`）
    -   `Status`（`completed successfully` / `failed` / `timed out` / `unknown`）
    -   紧凑的运行时/令牌统计信息
    -   一条交付指令，告诉请求者代理以正常的助手语气重写（而非转发原始内部元数据）
-   `--model` 和 `--thinking` 会覆盖该特定运行的默认值。
-   使用 `info`/`log` 在完成后检查详细信息和输出。
-   `/subagents spawn` 是单次运行模式（`mode: "run"`）。对于持久线程绑定的会话，请使用带有 `thread: true` 和 `mode: "session"` 的 `sessions_spawn`。
-   对于 ACP 工具会话（Codex, Claude Code, Gemini CLI），请使用带有 `runtime: "acp"` 的 `sessions_spawn`，并参见 [ACP 代理](./acp-agents.md)。

主要目标：

-   并行化“研究/长任务/慢工具”工作，而不阻塞主运行。
-   默认保持子代理隔离（会话分离 + 可选沙箱化）。
-   保持工具表面难以滥用：子代理默认**不**获取会话工具。
-   支持可配置的嵌套深度，用于编排器模式。

成本说明：每个子代理有其**自己的**上下文和令牌使用量。对于繁重或重复性任务，请为子代理设置更便宜的模型，并将您的主代理保持在更高质量的模型上。您可以通过 `agents.defaults.subagents.model` 或每个代理的覆盖项来配置此设置。

## 工具

使用 `sessions_spawn`：

-   启动一个子代理运行（`deliver: false`，全局通道：`subagent`）
-   然后运行一个通告步骤，并将通告回复发布到请求者聊天频道
-   默认模型：继承调用者，除非您设置了 `agents.defaults.subagents.model`（或每个代理的 `agents.list[].subagents.model`）；显式的 `sessions_spawn.model` 仍然优先。
-   默认思考级别：继承调用者，除非您设置了 `agents.defaults.subagents.thinking`（或每个代理的 `agents.list[].subagents.thinking`）；显式的 `sessions_spawn.thinking` 仍然优先。
-   默认运行超时：如果省略 `sessions_spawn.runTimeoutSeconds`，OpenClaw 在设置时使用 `agents.defaults.subagents.runTimeoutSeconds`；否则回退到 `0`（无超时）。

工具参数：

-   `task`（必需）
-   `label?`（可选）
-   `agentId?`（可选；如果允许，则在另一个代理ID下派生）
-   `model?`（可选；覆盖子代理模型；无效值将被跳过，子代理将在默认模型上运行，并在工具结果中给出警告）
-   `thinking?`（可选；覆盖子代理运行的思考级别）
-   `runTimeoutSeconds?`（默认为设置时的 `agents.defaults.subagents.runTimeoutSeconds`，否则为 `0`；设置时，子代理运行在 N 秒后中止）
-   `thread?`（默认 `false`；为 `true` 时，为此子代理会话请求频道线程绑定）
-   `mode?`（`run|session`）
    -   默认为 `run`
    -   如果 `thread: true` 且省略 `mode`，则默认变为 `session`
    -   `mode: "session"` 需要 `thread: true`
-   `cleanup?`（`delete|keep`，默认 `keep`）
-   `sandbox?`（`inherit|require`，默认 `inherit`；`require` 会拒绝派生，除非目标子运行时是沙箱化的）
-   `sessions_spawn` **不**接受频道交付参数（`target`, `channel`, `to`, `threadId`, `replyTo`, `transport`）。对于交付，请从派生的运行中使用 `message`/`sessions_send`。

## 线程绑定会话

当为频道启用线程绑定时，子代理可以保持绑定到一个线程，以便该线程中的后续用户消息继续路由到同一个子代理会话。

### 支持线程的频道

-   Discord（目前唯一支持的频道）：支持持久的线程绑定子代理会话（带有 `thread: true` 的 `sessions_spawn`）、手动线程控制（`/focus`, `/unfocus`, `/agents`, `/session idle`, `/session max-age`）以及适配器键 `channels.discord.threadBindings.enabled`, `channels.discord.threadBindings.idleHours`, `channels.discord.threadBindings.maxAgeHours` 和 `channels.discord.threadBindings.spawnSubagentSessions`。

快速流程：

1.  使用带有 `thread: true`（以及可选的 `mode: "session"`）的 `sessions_spawn` 进行派生。
2.  OpenClaw 在活动频道中创建或绑定一个线程到该会话目标。
3.  该线程中的回复和后续消息路由到绑定的会话。
4.  使用 `/session idle` 检查/更新不活动自动取消聚焦，使用 `/session max-age` 控制硬性上限。
5.  使用 `/unfocus` 手动分离。

手动控制：

-   `/focus ` 将当前线程（或创建一个）绑定到一个子代理/会话目标。
-   `/unfocus` 移除当前绑定线程的绑定。
-   `/agents` 列出活动运行和绑定状态（`thread:` 或 `unbound`）。
-   `/session idle` 和 `/session max-age` 仅对已聚焦的绑定线程有效。

配置开关：

-   全局默认值：`session.threadBindings.enabled`, `session.threadBindings.idleHours`, `session.threadBindings.maxAgeHours`
-   频道覆盖和派生自动绑定键是适配器特定的。参见上面的**支持线程的频道**。

有关当前适配器详情，请参见[配置参考](../gateway/configuration-reference.md)和[斜杠命令](./slash-commands.md)。允许列表：

-   `agents.list[].subagents.allowAgents`：可以通过 `agentId` 定位的代理ID列表（`["*"]` 表示允许任何）。默认：仅请求者代理。
-   沙箱继承保护：如果请求者会话是沙箱化的，`sessions_spawn` 会拒绝将运行在非沙箱化环境的目标。

发现：

-   使用 `agents_list` 查看当前哪些代理ID被允许用于 `sessions_spawn`。

自动归档：

-   子代理会话在 `agents.defaults.subagents.archiveAfterMinutes`（默认：60）后自动归档。
-   归档使用 `sessions.delete` 并将转录重命名为 `*.deleted.`（同一文件夹）。
-   `cleanup: "delete"` 在通告后立即归档（仍通过重命名保留转录）。
-   自动归档是尽力而为的；如果网关重启，待处理的计时器将丢失。
-   `runTimeoutSeconds` **不会**自动归档；它只停止运行。会话保持到自动归档。
-   自动归档同样适用于深度1和深度2的会话。

## 嵌套子代理

默认情况下，子代理无法派生自己的子代理（`maxSpawnDepth: 1`）。您可以通过设置 `maxSpawnDepth: 2` 来启用一级嵌套，这允许**编排器模式**：主代理 → 编排器子代理 → 工作器子子代理。

### 如何启用

```json
{
  agents: {
    defaults: {
      subagents: {
        maxSpawnDepth: 2, // 允许子代理派生子代理（默认：1）
        maxChildrenPerAgent: 5, // 每个代理会话的最大活动子代理数（默认：5）
        maxConcurrent: 8, // 全局并发通道上限（默认：8）
        runTimeoutSeconds: 900, // 省略时 sessions_spawn 的默认超时（0 = 无超时）
      },
    },
  },
}
```

### 深度级别

| 深度 | 会话键格式 | 角色 | 可以派生？ |
| --- | --- | --- | --- |
| 0 | `agent::main` | 主代理 | 总是可以 |
| 1 | `agent::subagent:` | 子代理（当允许深度2时，可作为编排器） | 仅当 `maxSpawnDepth >= 2` 时 |
| 2 | `agent::subagent::subagent:` | 子子代理（叶工作器） | 从不 |

### 通告链

结果沿链向上流动：

1.  深度2工作器完成 → 向其父级（深度1编排器）通告
2.  深度1编排器接收通告，综合结果，完成 → 向主代理通告
3.  主代理接收通告并交付给用户

每个级别只看到其直接子级的通告。

### 按深度的工具策略

-   **深度 1（编排器，当 `maxSpawnDepth >= 2` 时）**：获取 `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history`，以便管理其子级。其他会话/系统工具仍然被拒绝。
-   **深度 1（叶节点，当 `maxSpawnDepth == 1` 时）**：无会话工具（当前默认行为）。
-   **深度 2（叶工作器）**：无会话工具 — `sessions_spawn` 在深度2总是被拒绝。无法派生更多子级。

### 每个代理的派生限制

每个代理会话（在任何深度）最多可以同时拥有 `maxChildrenPerAgent`（默认：5）个活动子级。这可以防止单个编排器失控地扇出。

### 级联停止

停止一个深度1编排器会自动停止其所有深度2子级：

-   主聊天中的 `/stop` 会停止所有深度1代理，并级联到它们的深度2子级。
-   `/subagents kill ` 停止特定的子代理，并级联到其子级。
-   `/subagents kill all` 停止请求者的所有子代理，并级联。

## 认证

子代理认证由**代理ID**解析，而非会话类型：

-   子代理会话键是 `agent::subagent:`。
-   认证存储从该代理的 `agentDir` 加载。
-   主代理的认证配置文件作为**后备**合并进来；代理配置文件在冲突时覆盖主配置文件。

注意：合并是附加的，因此主配置文件始终可用作后备。目前尚不支持每个代理完全隔离的认证。

## 通告

子代理通过一个通告步骤报告：

-   通告步骤在子代理会话内部运行（而非请求者会话）。
-   如果子代理的回复恰好是 `ANNOUNCE_SKIP`，则不会发布任何内容。
-   否则，交付取决于请求者深度：
    -   顶级请求者会话使用带有外部交付（`deliver=true`）的后续 `agent` 调用
    -   嵌套的请求者子代理会话接收内部后续注入（`deliver=false`），以便编排器可以在会话内综合子级结果
    -   如果嵌套的请求者子代理会话已不存在，OpenClaw 在可用时回退到该会话的请求者
-   在构建嵌套完成结果时，子级完成聚合被限定在当前请求者运行内，防止过时的先前运行子级输出泄漏到当前通告中。
-   当频道适配器上可用时，通告回复保留线程/主题路由。
-   通告上下文被规范化为稳定的内部事件块：
    -   来源（`subagent` 或 `cron`）
    -   子会话键/ID
    -   通告类型 + 任务标签
    -   从运行时结果派生的状态行（`success`, `error`, `timeout`, 或 `unknown`）
    -   来自通告步骤的结果内容（如果缺失则为 `(no output)`）
    -   一条后续指令，描述何时回复与保持静默
-   `Status` 不是从模型输出推断的；它来自运行时结果信号。

通告负载在末尾包含一行统计信息（即使被包装）：

-   运行时（例如，`runtime 5m12s`）
-   令牌使用量（输入/输出/总计）
-   配置模型定价时的估计成本（`models.providers.*.models[].cost`）
-   `sessionKey`, `sessionId` 和转录路径（以便主代理可以通过 `sessions_history` 获取历史记录或在磁盘上检查文件）
-   内部元数据仅用于编排；面向用户的回复应以正常的助手语气重写。

## 工具策略（子代理工具）

默认情况下，子代理获取**除会话工具和系统工具外的所有工具**：

-   `sessions_list`
-   `sessions_history`
-   `sessions_send`
-   `sessions_spawn`

当 `maxSpawnDepth >= 2` 时，深度1编排器子代理额外接收 `sessions_spawn`, `subagents`, `sessions_list` 和 `sessions_history`，以便它们可以管理其子级。通过配置覆盖：

```json
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 1,
      },
    },
  },
  tools: {
    subagents: {
      tools: {
        // deny 优先
        deny: ["gateway", "cron"],
        // 如果设置了 allow，则变为仅允许（deny 仍然优先）
        // allow: ["read", "exec", "process"]
      },
    },
  },
}
```

## 并发

子代理使用专用的进程内队列通道：

-   通道名称：`subagent`
-   并发数：`agents.defaults.subagents.maxConcurrent`（默认 `8`）

## 停止

-   在请求者聊天中发送 `/stop` 会中止请求者会话，并停止由其派生的任何活动子代理运行，级联到嵌套子级。
-   `/subagents kill ` 停止特定的子代理，并级联到其子级。

## 限制

-   子代理通告是**尽力而为的**。如果网关重启，待处理的“通告返回”工作将丢失。
-   子代理仍然共享相同的网关进程资源；请将 `maxConcurrent` 视为安全阀。
-   `sessions_spawn` 总是非阻塞的：它立即返回 `{ status: "accepted", runId, childSessionKey }`。
-   子代理上下文仅注入 `AGENTS.md` + `TOOLS.md`（无 `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md` 或 `BOOTSTRAP.md`）。
-   最大嵌套深度为 5（`maxSpawnDepth` 范围：1–5）。深度2推荐用于大多数用例。
-   `maxChildrenPerAgent` 限制每个会话的活动子级数（默认：5，范围：1–20）。

[代理发送](./agent-send.md)[ACP 代理](./acp-agents.md)