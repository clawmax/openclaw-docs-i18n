

  实验

  
# OpenResponses 网关计划

## 背景

OpenClaw Gateway 目前通过 `/v1/chat/completions` 端点暴露了一个最小化的 OpenAI 兼容 Chat Completions 接口（参见 [OpenAI Chat Completions](../../gateway/openai-http-api.md)）。Open Responses 是一个基于 OpenAI Responses API 的开放推理标准。它专为智能体工作流设计，使用基于项目的输入和语义流式事件。OpenResponses 规范定义的是 `/v1/responses`，而非 `/v1/chat/completions`。

## 目标

-   添加一个符合 OpenResponses 语义的 `/v1/responses` 端点。
-   将 Chat Completions 保留为一个易于禁用并最终移除的兼容层。
-   使用隔离、可复用的模式来标准化验证和解析。

## 非目标

-   在首次实现中实现完整的 OpenResponses 功能对等（图像、文件、托管工具）。
-   替换内部智能体执行逻辑或工具编排。
-   在第一阶段改变现有的 `/v1/chat/completions` 行为。

## 研究总结

信息来源：OpenResponses OpenAPI、OpenResponses 规范网站以及 Hugging Face 博客文章。提取的关键点：

-   `POST /v1/responses` 接受 `CreateResponseBody` 字段，如 `model`、`input`（字符串或 `ItemParam[]`）、`instructions`、`tools`、`tool_choice`、`stream`、`max_output_tokens` 和 `max_tool_calls`。
-   `ItemParam` 是一个可区分的联合类型，包括：
    -   角色为 `system`、`developer`、`user`、`assistant` 的 `message` 项目
    -   `function_call` 和 `function_call_output`
    -   `reasoning`
    -   `item_reference`
-   成功的响应返回一个 `ResponseResource`，包含 `object: "response"`、`status` 和 `output` 项目。
-   流式传输使用语义事件，例如：
    -   `response.created`、`response.in_progress`、`response.completed`、`response.failed`
    -   `response.output_item.added`、`response.output_item.done`
    -   `response.content_part.added`、`response.content_part.done`
    -   `response.output_text.delta`、`response.output_text.done`
-   规范要求：
    -   `Content-Type: text/event-stream`
    -   `event:` 必须与 JSON 的 `type` 字段匹配
    -   终止事件必须是字面量 `[DONE]`
-   推理项目可能暴露 `content`、`encrypted_content` 和 `summary`。
-   HF 示例在请求中包含 `OpenResponses-Version: latest`（可选头部）。

## 提议的架构

-   添加 `src/gateway/open-responses.schema.ts`，仅包含 Zod 模式（无网关导入）。
-   添加 `src/gateway/openresponses-http.ts`（或 `open-responses-http.ts`）用于 `/v1/responses`。
-   保持 `src/gateway/openai-http.ts` 不变，作为遗留兼容性适配器。
-   添加配置 `gateway.http.endpoints.responses.enabled`（默认 `false`）。
-   保持 `gateway.http.endpoints.chatCompletions.enabled` 独立；允许两个端点分别切换。
-   当 Chat Completions 启用时，发出启动警告以表明其遗留状态。

## Chat Completions 的弃用路径

-   保持严格的模块边界：responses 和 chat completions 之间不共享模式类型。
-   通过配置使 Chat Completions 成为可选项，以便无需代码更改即可禁用它。
-   一旦 `/v1/responses` 稳定，更新文档将 Chat Completions 标记为遗留功能。
-   可选的未来步骤：将 Chat Completions 请求映射到 Responses 处理器，以简化移除路径。

## 第一阶段支持子集

-   接受 `input` 作为字符串或 `ItemParam[]`，包含消息角色和 `function_call_output`。
-   将系统和开发者消息提取到 `extraSystemPrompt` 中。
-   使用最近的 `user` 或 `function_call_output` 作为智能体运行的当前消息。
-   拒绝不支持的内容部分（图像/文件），并返回 `invalid_request_error`。
-   返回一个包含 `output_text` 内容的助手消息。
-   在令牌计数功能接入之前，返回 `usage` 字段并设为零值。

## 验证策略（无 SDK）

-   为以下支持子集实现 Zod 模式：
    -   `CreateResponseBody`
    -   `ItemParam` + 消息内容部分联合类型
    -   `ResponseResource`
    -   网关使用的流式事件形状
-   将模式保存在一个独立的模块中，以避免漂移并允许未来的代码生成。

## 流式传输实现（第一阶段）

-   SSE 行同时包含 `event:` 和 `data:`。
-   必需的事件序列（最小可行）：
    -   `response.created`
    -   `response.output_item.added`
    -   `response.content_part.added`
    -   `response.output_text.delta`（根据需要重复）
    -   `response.output_text.done`
    -   `response.content_part.done`
    -   `response.completed`
    -   `[DONE]`

## 测试与验证计划

-   为 `/v1/responses` 添加端到端测试覆盖：
    -   需要身份验证
    -   非流式响应形状
    -   流式事件顺序和 `[DONE]`
    -   使用头部和 `user` 字段的会话路由
-   保持 `src/gateway/openai-http.test.ts` 不变。
-   手动测试：使用 `stream: true` 向 `/v1/responses` 发送 curl 请求，并验证事件顺序和终止的 `[DONE]`。

## 文档更新（后续）

-   为 `/v1/responses` 的使用和示例添加新的文档页面。
-   更新 `/gateway/openai-http-api`，添加遗留说明并指向 `/v1/responses`。

[浏览器评估 CDP 重构](./browser-evaluate-cdp-refactor.md)[PTY 与进程监督计划](./pty-process-supervision.md)

---