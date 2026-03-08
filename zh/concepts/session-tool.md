

  会话与记忆

  
# 会话工具

目标：提供一套小巧、不易误用的工具集，使代理能够列出会话、获取历史记录以及向另一个会话发送消息。

## 工具名称

-   `sessions_list`
-   `sessions_history`
-   `sessions_send`
-   `sessions_spawn`

## 关键模型

-   主要的直接聊天存储桶始终是字面键 `"main"`（解析为当前代理的主键）。
-   群聊使用 `agent:::group:` 或 `agent:::channel:`（传递完整键）。
-   定时任务使用 `cron:<job.id>`。
-   钩子使用 `hook:`，除非显式设置。
-   节点会话使用 `node-`，除非显式设置。

`global` 和 `unknown` 是保留值，永远不会被列出。如果 `session.scope = "global"`，我们将其在所有工具中别名为 `main`，因此调用者永远不会看到 `global`。

## sessions_list

将会话以行数组形式列出。参数：

-   `kinds?: string[]` 过滤器：可以是 `"main" | "group" | "cron" | "hook" | "node" | "other"` 中的任意值
-   `limit?: number` 最大行数（默认值：服务器默认值，例如限制为 200）
-   `activeMinutes?: number` 仅列出在 N 分钟内更新过的会话
-   `messageLimit?: number` 0 = 不包含消息（默认 0）；>0 = 包含最后 N 条消息

行为：

-   `messageLimit > 0` 会获取每个会话的 `chat.history` 并包含最后 N 条消息。
-   工具结果在列表输出中被过滤掉；使用 `sessions_history` 获取工具消息。
-   在**沙盒化**的代理会话中运行时，会话工具默认采用**仅限生成会话可见性**（见下文）。

行结构（JSON）：

-   `key`: 会话键（字符串）
-   `kind`: `main | group | cron | hook | node | other`
-   `channel`: `whatsapp | telegram | discord | signal | imessage | webchat | internal | unknown`
-   `displayName`（群组显示标签，如果可用）
-   `updatedAt`（毫秒）
-   `sessionId`
-   `model`, `contextTokens`, `totalTokens`
-   `thinkingLevel`, `verboseLevel`, `systemSent`, `abortedLastRun`
-   `sendPolicy`（如果设置了会话覆盖）
-   `lastChannel`, `lastTo`
-   `deliveryContext`（规范化后的 `{ channel, to, accountId }`，当可用时）
-   `transcriptPath`（从存储目录 + sessionId 派生的最佳路径）
-   `messages?`（仅当 `messageLimit > 0` 时）

## sessions_history

获取单个会话的转录记录。参数：

-   `sessionKey`（必需；接受会话键或来自 `sessions_list` 的 `sessionId`）
-   `limit?: number` 最大消息数（服务器限制）
-   `includeTools?: boolean`（默认 false）

行为：

-   `includeTools=false` 会过滤掉 `role: "toolResult"` 消息。
-   以原始转录格式返回消息数组。
-   当给定 `sessionId` 时，OpenClaw 会将其解析为相应的会话键（缺失的 id 会报错）。

## sessions_send

向另一个会话发送消息。参数：

-   `sessionKey`（必需；接受会话键或来自 `sessions_list` 的 `sessionId`）
-   `message`（必需）
-   `timeoutSeconds?: number`（默认 >0；0 = 即发即弃）

行为：

-   `timeoutSeconds = 0`：将消息加入队列并返回 `{ runId, status: "accepted" }`。
-   `timeoutSeconds > 0`：等待最多 N 秒以完成，然后返回 `{ runId, status: "ok", reply }`。
-   如果等待超时：`{ runId, status: "timeout", error }`。运行继续；稍后调用 `sessions_history`。
-   如果运行失败：`{ runId, status: "error", error }`。
-   通知传递在主运行完成后运行，并且是尽力而为的；`status: "ok"` 不保证通知已送达。
-   通过网关 `agent.wait`（服务器端）进行等待，因此重新连接不会中断等待。
-   代理到代理的消息上下文被注入到主运行中。
-   会话间消息以 `message.provenance.kind = "inter_session"` 持久化，以便转录阅读器能够区分路由的代理指令和外部用户输入。
-   主运行完成后，OpenClaw 运行一个**回复循环**：
    -   第 2 轮及以后在请求代理和目标代理之间交替进行。
    -   精确回复 `REPLY_SKIP` 以停止乒乓式来回。
    -   最大轮次为 `session.agentToAgent.maxPingPongTurns`（0–5，默认 5）。
-   循环结束后，OpenClaw 运行**代理到代理通知步骤**（仅限目标代理）：
    -   精确回复 `ANNOUNCE_SKIP` 以保持静默。
    -   任何其他回复都会发送到目标频道。
    -   通知步骤包含原始请求 + 第 1 轮回复 + 最新的乒乓式回复。

## 频道字段

-   对于群组，`channel` 是会话条目上记录的频道。
-   对于直接聊天，`channel` 从 `lastChannel` 映射而来。
-   对于定时任务/钩子/节点，`channel` 是 `internal`。
-   如果缺失，`channel` 为 `unknown`。

## 安全性 / 发送策略

基于策略的按频道/聊天类型（而非按会话 ID）进行阻止。

```json
{
  "session": {
    "sendPolicy": {
      "rules": [
        {
          "match": { "channel": "discord", "chatType": "group" },
          "action": "deny"
        }
      ],
      "default": "allow"
    }
  }
}
```

运行时覆盖（每个会话条目）：

-   `sendPolicy: "allow" | "deny"`（未设置 = 继承配置）
-   可通过 `sessions.patch` 或仅限所有者的 `/send on|off|inherit`（独立消息）设置。

执行点：

-   `chat.send` / `agent`（网关）
-   自动回复传递逻辑

## sessions_spawn

在隔离的会话中生成子代理运行，并将结果通知回请求者的聊天频道。参数：

-   `task`（必需）
-   `label?`（可选；用于日志/UI）
-   `agentId?`（可选；如果允许，则在另一个代理 ID 下生成）
-   `model?`（可选；覆盖子代理模型；无效值会报错）
-   `thinking?`（可选；覆盖子代理运行的思考级别）
-   `runTimeoutSeconds?`（当设置时，默认为 `agents.defaults.subagents.runTimeoutSeconds`，否则为 `0`；设置后，在 N 秒后中止子代理运行）
-   `thread?`（默认 false；当频道/插件支持时，为此生成请求线程绑定路由）
-   `mode?`（`run|session`；默认为 `run`，但当 `thread=true` 时默认为 `session`；`mode="session"` 要求 `thread=true`）
-   `cleanup?`（`delete|keep`，默认 `keep`）
-   `sandbox?`（`inherit|require`，默认 `inherit`；`require` 会拒绝生成，除非目标子运行时是沙盒化的）
-   `attachments?`（可选的内联文件数组；仅限子代理运行时，ACP 会拒绝）。每个条目：`{ name, content, encoding?: "utf8" | "base64", mimeType? }`。文件被具体化到子工作区的 `.openclaw/attachments//` 目录下。返回每个文件的 sha256 收据。
-   `attachAs?`（可选；`{ mountPath? }` 提示，为未来的挂载实现保留）

允许列表：

-   `agents.list[].subagents.allowAgents`：允许通过 `agentId` 使用的代理 ID 列表（`["*"]` 表示允许任何代理）。默认：仅允许请求者代理。
-   沙盒继承保护：如果请求者会话是沙盒化的，`sessions_spawn` 会拒绝将在非沙盒环境下运行的目标。

发现：

-   使用 `agents_list` 来发现哪些代理 ID 被允许用于 `sessions_spawn`。

行为：

-   启动一个新的 `agent::subagent:` 会话，并设置 `deliver: false`。
-   子代理默认拥有完整工具集**减去会话工具**（可通过 `tools.subagents.tools` 配置）。
-   子代理不允许调用 `sessions_spawn`（不允许子代理 → 子代理生成）。
-   始终非阻塞：立即返回 `{ status: "accepted", runId, childSessionKey }`。
-   当 `thread=true` 时，频道插件可以将传递/路由绑定到线程目标（Discord 支持由 `session.threadBindings.*` 和 `channels.discord.threadBindings.*` 控制）。
-   完成后，OpenClaw 运行子代理**通知步骤**，并将结果发布到请求者的聊天频道。
    -   如果助手最终回复为空，则从子代理历史记录中包含最新的 `toolResult` 作为 `Result`。
-   在通知步骤中精确回复 `ANNOUNCE_SKIP` 以保持静默。
-   通知回复被规范化为 `Status`/`Result`/`Notes`；`Status` 来自运行时结果（而非模型文本）。
-   子代理会话在 `agents.defaults.subagents.archiveAfterMinutes`（默认：60）后自动归档。
-   通知回复包含一行统计信息（运行时间、令牌数、sessionKey/sessionId、转录路径以及可选的成本）。

## 沙盒会话可见性

会话工具可以限定范围以减少跨会话访问。默认行为：

-   `tools.sessions.visibility` 默认为 `tree`（当前会话 + 生成的子代理会话）。
-   对于沙盒化会话，`agents.defaults.sandbox.sessionToolsVisibility` 可以硬性限制可见性。

配置：

```json
{
  tools: {
    sessions: {
      // "self" | "tree" | "agent" | "all"
      // 默认: "tree"
      visibility: "tree",
    },
  },
  agents: {
    defaults: {
      sandbox: {
        // 默认: "spawned"
        sessionToolsVisibility: "spawned", // 或 "all"
      },
    },
  },
}
```

说明：

-   `self`：仅当前会话键。
-   `tree`：当前会话 + 由当前会话生成的会话。
-   `agent`：属于当前代理 ID 的任何会话。
-   `all`：任何会话（跨代理访问仍需要 `tools.agentToAgent`）。
-   当会话是沙盒化的且 `sessionToolsVisibility="spawned"` 时，即使你设置了 `tools.sessions.visibility="all"`，OpenClaw 也会将可见性限制为 `tree`。

[会话清理](./session-pruning.md)[记忆](./memory.md)