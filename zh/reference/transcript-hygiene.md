

  技术参考

  
# 转录卫生

本文档描述了在运行前（构建模型上下文时）应用于转录的**供应商特定修复**。这些是用于满足严格供应商要求的**内存中**调整。这些卫生步骤**不会**重写磁盘上存储的 JSONL 转录文件；但是，一个单独的会话文件修复过程可能会在会话加载之前，通过丢弃无效行来重写格式错误的 JSONL 文件。当发生修复时，原始文件会与会话文件一起备份。范围包括：

-   工具调用 ID 清理
-   工具调用输入验证
-   工具结果配对修复
-   轮次验证 / 排序
-   思考签名清理
-   图像负载清理
-   用户输入来源标记（用于会话间路由的提示）

如果您需要转录存储的详细信息，请参阅：

-   [/reference/session-management-compaction](./session-management-compaction.md)

* * *

## 运行位置

所有转录卫生处理都集中在嵌入式运行器中：

-   策略选择：`src/agents/transcript-policy.ts`
-   清理/修复应用：`src/agents/pi-embedded-runner/google.ts` 中的 `sanitizeSessionHistory`

该策略使用 `provider`、`modelApi` 和 `modelId` 来决定应用什么。与转录卫生处理分开，会话文件在加载前会被修复（如果需要）：

-   `src/agents/session-file-repair.ts` 中的 `repairSessionFileIfNeeded`
-   从 `run/attempt.ts` 和 `compact.ts`（嵌入式运行器）调用

* * *

## 全局规则：图像清理

图像负载总是被清理，以防止因大小限制（对过大的 base64 图像进行下采样/重新压缩）而导致供应商端拒绝。这也有助于控制支持视觉的模型的图像驱动令牌压力。较低的最大尺寸通常会减少令牌使用量；较高的尺寸会保留更多细节。实现：

-   `src/agents/pi-embedded-helpers/images.ts` 中的 `sanitizeSessionMessagesImages`
-   `src/agents/tool-images.ts` 中的 `sanitizeContentBlocksImages`
-   最大图像边长可通过 `agents.defaults.imageMaxDimensionPx` 配置（默认值：`1200`）。

* * *

## 全局规则：格式错误的工具调用

缺少 `input` 和 `arguments` 的助手工具调用块在构建模型上下文之前会被丢弃。这可以防止因部分持久化的工具调用（例如，在速率限制失败后）而导致供应商拒绝。实现：

-   `src/agents/session-transcript-repair.ts` 中的 `sanitizeToolCallInputs`
-   在 `src/agents/pi-embedded-runner/google.ts` 中的 `sanitizeSessionHistory` 中应用

* * *

## 全局规则：会话间输入来源

当代理通过 `sessions_send` 将提示发送到另一个会话时（包括代理到代理的回复/通知步骤），OpenClaw 会持久化创建的用户轮次，并附带：

-   `message.provenance.kind = "inter_session"`

此元数据在转录追加时写入，并且不改变角色（`role: "user"` 保持不变以确保供应商兼容性）。转录读取器可以使用此信息来避免将路由的内部提示视为最终用户编写的指令。在上下文重建期间，OpenClaw 还会在内存中为这些用户轮次预先添加一个简短的 `[Inter-session message]` 标记，以便模型能够将它们与外部最终用户指令区分开来。

* * *

## 供应商矩阵（当前行为）

**OpenAI / OpenAI Codex**

-   仅图像清理。
-   为 OpenAI Responses/Codex 转录丢弃孤立的推理签名（没有后续内容块的独立推理项）。
-   无工具调用 ID 清理。
-   无工具结果配对修复。
-   无轮次验证或重新排序。
-   无合成工具结果。
-   无思考签名剥离。

**Google (Generative AI / Gemini CLI / Antigravity)**

-   工具调用 ID 清理：严格的字母数字。
-   工具结果配对修复和合成工具结果。
-   轮次验证（Gemini 风格的轮次交替）。
-   Google 轮次顺序修复（如果历史记录以助手开始，则预先添加一个微小的用户引导）。
-   Antigravity Claude：规范化思考签名；丢弃未签名的思考块。

**Anthropic / Minimax (Anthropic 兼容)**

-   工具结果配对修复和合成工具结果。
-   轮次验证（合并连续的用户轮次以满足严格的交替要求）。

**Mistral (包括基于模型 ID 的检测)**

-   工具调用 ID 清理：strict9（长度为 9 的字母数字）。

**OpenRouter Gemini**

-   思考签名清理：剥离非 base64 的 `thought_signature` 值（保留 base64）。

**其他所有情况**

-   仅图像清理。

* * *

## 历史行为（2026.1.22 之前）

在 2026.1.22 版本发布之前，OpenClaw 应用了多层次的转录卫生处理：

-   一个**转录清理扩展**在每次上下文构建时运行，可以：
    -   修复工具使用/结果配对。
    -   清理工具调用 ID（包括保留 `_`/`-` 的非严格模式）。
-   运行器还执行供应商特定的清理，这造成了重复工作。
-   在供应商策略之外发生了额外的变更，包括：
    -   在持久化之前从助手文本中剥离 `` 标签。
    -   丢弃空的助手错误轮次。
    -   在工具调用后修剪助手内容。

这种复杂性导致了跨供应商的回归（特别是 `openai-responses` 的 `call_id|fc_id` 配对）。2026.1.22 的清理工作移除了该扩展，将逻辑集中在运行器中，并使 OpenAI 除了图像清理外**不做任何改动**。

[API 使用与成本](./api-usage-costs.md)[日期与时间](../date-time.md)