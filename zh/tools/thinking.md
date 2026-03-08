

  内置工具

  
# 思维层级

## 功能说明

-   在任何入站消息正文中使用内联指令：`/t `、`/think:` 或 `/thinking `。
-   层级（别名）：`off | minimal | low | medium | high | xhigh | adaptive`
    -   minimal → “think”
    -   low → “think hard”
    -   medium → “think harder”
    -   high → “ultrathink”（最大预算）
    -   xhigh → “ultrathink+”（仅限 GPT-5.2 + Codex 模型）
    -   adaptive → 提供商管理的自适应推理预算（支持 Anthropic Claude 4.6 模型系列）
    -   `x-high`、`x_high`、`extra-high`、`extra high` 和 `extra_high` 映射到 `xhigh`。
    -   `highest`、`max` 映射到 `high`。
-   提供商说明：
    -   Anthropic Claude 4.6 模型在未设置显式思维层级时默认为 `adaptive`。
    -   Z.AI (`zai/*`) 仅支持二进制思维（`on`/`off`）。任何非 `off` 的层级都被视为 `on`（映射到 `low`）。
    -   Moonshot (`moonshot/*`) 将 `/think off` 映射到 `thinking: { type: "disabled" }`，并将任何非 `off` 的层级映射到 `thinking: { type: "enabled" }`。当思维启用时，Moonshot 只接受 `tool_choice` 为 `auto|none`；OpenClaw 会将不兼容的值规范化为 `auto`。

## 解析顺序

1.  消息上的内联指令（仅适用于该条消息）。
2.  会话覆盖（通过发送仅包含指令的消息设置）。
3.  全局默认值（配置中的 `agents.defaults.thinkingDefault`）。
4.  后备方案：Anthropic Claude 4.6 模型为 `adaptive`，其他支持推理的模型为 `low`，否则为 `off`。

## 设置会话默认值

-   发送一条**仅包含**指令的消息（允许有空白字符），例如 `/think:medium` 或 `/t high`。
-   该设置将在当前会话中保持有效（默认按发送者区分）；可通过 `/think:off` 或会话空闲重置来清除。
-   系统会发送确认回复（`Thinking level set to high.` / `Thinking disabled.`）。如果层级无效（例如 `/thinking big`），命令将被拒绝并给出提示，且会话状态保持不变。
-   发送不带参数的 `/think`（或 `/think:`）以查看当前思维层级。

## 按代理应用

-   **嵌入式 Pi**：解析后的层级会传递给进程内的 Pi 代理运行时。

## 详细指令 (/verbose 或 /v)

-   层级：`on`（最小）| `full` | `off`（默认）。
-   仅包含指令的消息会切换会话的详细日志记录状态，并回复 `Verbose logging enabled.` / `Verbose logging disabled.`；无效层级会返回提示而不改变状态。
-   `/verbose off` 会存储一个显式的会话覆盖；可通过会话 UI 选择 `inherit` 来清除它。
-   内联指令仅影响该条消息；否则应用会话/全局默认值。
-   发送不带参数的 `/verbose`（或 `/verbose:`）以查看当前的详细日志级别。
-   当详细日志开启时，发出结构化工具结果的代理（Pi、其他 JSON 代理）会将每个工具调用作为其自己的仅元数据消息发送回来，并在可用时（路径/命令）以 ` <tool-name>: ` 为前缀。这些工具摘要会在每个工具启动时立即发送（单独的对话气泡），而不是作为流式增量发送。
-   工具失败摘要在正常模式下仍然可见，但原始错误详细信息后缀会被隐藏，除非详细日志级别为 `on` 或 `full`。
-   当详细日志级别为 `full` 时，工具输出也会在完成后转发（单独的对话气泡，截断到安全长度）。如果在运行过程中切换 `/verbose on|full|off`，后续的工具气泡将遵循新设置。

## 推理可见性 (/reasoning)

-   层级：`on|off|stream`。
-   仅包含指令的消息会切换是否在回复中显示思维区块。
-   启用后，推理会作为**单独的消息**发送，前缀为 `Reasoning:`。
-   `stream`（仅限 Telegram）：在生成回复时将推理流式传输到 Telegram 的草稿气泡中，然后发送最终答案而不包含推理。
-   别名：`/reason`。
-   发送不带参数的 `/reasoning`（或 `/reasoning:`）以查看当前的推理级别。

## 相关链接

-   提升模式文档位于[提升模式](./elevated.md)。

## 心跳检测

-   心跳探测正文是配置的心跳提示（默认：`如果存在 HEARTBEAT.md 文件（工作区上下文），请阅读它。严格遵守其指示。不要推断或重复先前聊天中的旧任务。如果无需关注任何事项，请回复 HEARTBEAT_OK。`）。心跳消息中的内联指令照常应用（但应避免通过心跳消息更改会话默认值）。
-   心跳传递默认仅发送最终有效载荷。若要同时发送单独的 `Reasoning:` 消息（当可用时），请设置 `agents.defaults.heartbeat.includeReasoning: true` 或按代理设置 `agents.list[].heartbeat.includeReasoning: true`。

## 网页聊天界面

-   网页聊天思维选择器在页面加载时反映入站会话存储/配置中存储的会话层级。
-   选择另一个层级仅适用于下一条消息（`thinkingOnce`）；发送后，选择器会跳回存储的会话层级。
-   要更改会话默认值，请发送 `/think:` 指令（如前所述）；选择器将在下次重新加载后反映该更改。

[反应](./reactions.md)[网页工具](./web.md)