

  基础概念

  
# 智能体循环

智能体循环是智能体一次完整的“真实”运行过程：输入 → 上下文组装 → 模型推理 → 工具执行 → 流式回复 → 持久化。这是将消息转化为动作和最终回复的权威路径，同时保持会话状态的一致性。在 OpenClaw 中，一个循环是每个会话的一次串行化运行，在模型思考、调用工具和流式输出时发出生命周期和流事件。本文档解释了这一真实循环是如何端到端连接的。

## 入口点

-   网关 RPC: `agent` 和 `agent.wait`。
-   CLI: `agent` 命令。

## 工作原理（高层概述）

1.  `agent` RPC 验证参数，解析会话（sessionKey/sessionId），持久化会话元数据，立即返回 `{ runId, acceptedAt }`。
2.  `agentCommand` 运行智能体：
    -   解析模型 + 思考/详细模式默认值
    -   加载技能快照
    -   调用 `runEmbeddedPiAgent`（pi-agent-core 运行时）
    -   如果嵌入式循环未发出**生命周期结束/错误**事件，则发出该事件
3.  `runEmbeddedPiAgent`：
    -   通过每个会话 + 全局队列串行化运行
    -   解析模型 + 认证配置文件并构建 pi 会话
    -   订阅 pi 事件并流式传输助手/工具增量
    -   强制执行超时 -> 如果超时则中止运行
    -   返回有效载荷 + 使用量元数据
4.  `subscribeEmbeddedPiSession` 将 pi-agent-core 事件桥接到 OpenClaw `agent` 流：
    -   工具事件 => `stream: "tool"`
    -   助手增量 => `stream: "assistant"`
    -   生命周期事件 => `stream: "lifecycle"` (`phase: "start" | "end" | "error"`)
5.  `agent.wait` 使用 `waitForAgentJob`：
    -   等待 `runId` 的**生命周期结束/错误**事件
    -   返回 `{ status: ok|error|timeout, startedAt, endedAt, error? }`

## 队列 + 并发

-   运行按会话键（会话通道）串行化，并可选择通过全局通道串行化。
-   这可以防止工具/会话竞争并保持会话历史的一致性。
-   消息传递通道可以选择馈送到此通道系统的队列模式（collect/steer/followup）。请参阅[命令队列](./queue.md)。

## 会话 + 工作区准备

-   工作区被解析和创建；沙盒运行可能会重定向到沙盒工作区根目录。
-   技能被加载（或从快照重用）并注入到环境和提示中。
-   引导/上下文文件被解析并注入到系统提示报告中。
-   获取会话写锁；`SessionManager` 在流式传输前被打开并准备就绪。

## 提示组装 + 系统提示

-   系统提示由 OpenClaw 的基础提示、技能提示、引导上下文和每次运行的覆盖项构建而成。
-   强制执行模型特定的限制和压缩保留令牌。
-   关于模型看到的内容，请参阅[系统提示](./system-prompt.md)。

## 钩子点（可以拦截的位置）

OpenClaw 有两个钩子系统：

-   **内部钩子**（网关钩子）：用于命令和生命周期事件的事件驱动脚本。
-   **插件钩子**：智能体/工具生命周期和网关管道内部的扩展点。

### 内部钩子（网关钩子）

-   **`agent:bootstrap`**：在构建引导文件时运行，在系统提示最终确定之前。使用此钩子添加/移除引导上下文文件。
-   **命令钩子**：`/new`、`/reset`、`/stop` 和其他命令事件（参见钩子文档）。

有关设置和示例，请参阅[钩子](../automation/hooks.md)。

### 插件钩子（智能体 + 网关生命周期）

这些钩子在智能体循环或网关管道内部运行：

-   **`before_model_resolve`**：在会话之前运行（无 `messages`），用于在模型解析之前确定性地覆盖提供者/模型。
-   **`before_prompt_build`**：在会话加载后运行（带有 `messages`），用于在提示提交前注入 `prependContext`、`systemPrompt`、`prependSystemContext` 或 `appendSystemContext`。使用 `prependContext` 用于每轮动态文本，使用系统上下文字段用于应位于系统提示空间的稳定指导。
-   **`before_agent_start`**：旧版兼容性钩子，可能在任一阶段运行；建议使用上述明确的钩子。
-   **`agent_end`**：在完成后检查最终消息列表和运行元数据。
-   **`before_compaction` / `after_compaction`**：观察或注释压缩周期。
-   **`before_tool_call` / `after_tool_call`**：拦截工具参数/结果。
-   **`tool_result_persist`**：在工具结果写入会话记录之前同步转换它们。
-   **`message_received` / `message_sending` / `message_sent`**：入站 + 出站消息钩子。
-   **`session_start` / `session_end`**：会话生命周期边界。
-   **`gateway_start` / `gateway_stop`**：网关生命周期事件。

有关钩子 API 和注册详细信息，请参阅[插件](../tools/plugin.md#plugin-hooks)。

## 流式传输 + 部分回复

-   助手增量从 pi-agent-core 流式传输并作为 `assistant` 事件发出。
-   块流式传输可以在 `text_end` 或 `message_end` 时发出部分回复。
-   推理流式传输可以作为单独的流或作为块回复发出。
-   有关分块和块回复行为，请参阅[流式传输](./streaming.md)。

## 工具执行 + 消息传递工具

-   工具开始/更新/结束事件在 `tool` 流上发出。
-   工具结果在记录/发出前会进行大小和图像有效载荷的清理。
-   消息传递工具的发送被跟踪，以抑制重复的助手确认。

## 回复整形 + 抑制

-   最终有效载荷由以下部分组装：
    -   助手文本（以及可选的推理）
    -   内联工具摘要（当启用详细模式且允许时）
    -   模型出错时的助手错误文本
-   `NO_REPLY` 被视为静默令牌并从传出有效载荷中过滤掉。
-   消息传递工具的重复项从最终有效载荷列表中移除。
-   如果没有剩余的可渲染有效载荷且工具出错，则会发出一个后备工具错误回复（除非消息传递工具已经发送了用户可见的回复）。

## 压缩 + 重试

-   自动压缩会发出 `compaction` 流事件，并可能触发重试。
-   重试时，内存缓冲区和工具摘要会被重置，以避免重复输出。
-   有关压缩管道，请参阅[压缩](./compaction.md)。

## 事件流（当前）

-   `lifecycle`：由 `subscribeEmbeddedPiSession` 发出（以及作为 `agentCommand` 的后备）
-   `assistant`：来自 pi-agent-core 的流式增量
-   `tool`：来自 pi-agent-core 的流式工具事件

## 聊天通道处理

-   助手增量被缓冲到聊天 `delta` 消息中。
-   在**生命周期结束/错误**时发出聊天 `final` 消息。

## 超时

-   `agent.wait` 默认值：30秒（仅等待）。`timeoutMs` 参数可覆盖。
-   智能体运行时：`agents.defaults.timeoutSeconds` 默认 600秒；在 `runEmbeddedPiAgent` 中止计时器中强制执行。

## 可能提前结束的情况

-   智能体超时（中止）
-   AbortSignal（取消）
-   网关断开连接或 RPC 超时
-   `agent.wait` 超时（仅等待，不会停止智能体）

[智能体运行时](./agent.md)[系统提示](./system-prompt.md)