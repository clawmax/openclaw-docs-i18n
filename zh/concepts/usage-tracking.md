

  概念内部

  
# 使用量追踪

## 是什么

-   直接从提供商的使用量端点拉取使用量/配额。
-   无估算成本；仅显示提供商报告的时间窗口数据。

## 显示位置

-   聊天中的 `/status`：包含会话令牌和估算成本（仅限 API 密钥）的丰富表情状态卡。当可用时，会显示**当前模型提供商**的使用量。
-   聊天中的 `/usage off|tokens|full`：每条响应的使用量页脚（OAuth 仅显示令牌）。
-   聊天中的 `/usage cost`：从 OpenClaw 会话日志聚合的本地成本摘要。
-   CLI：`openclaw status --usage` 打印完整的按提供商细分数据。
-   CLI：`openclaw channels list` 在提供商配置旁打印相同的使用量快照（使用 `--no-usage` 跳过）。
-   macOS 菜单栏：上下文下的“使用量”部分（仅在可用时显示）。

## 提供商 + 凭证

-   **Anthropic (Claude)**：认证配置文件中的 OAuth 令牌。
-   **GitHub Copilot**：认证配置文件中的 OAuth 令牌。
-   **Gemini CLI**：认证配置文件中的 OAuth 令牌。
-   **Antigravity**：认证配置文件中的 OAuth 令牌。
-   **OpenAI Codex**：认证配置文件中的 OAuth 令牌（存在时使用 accountId）。
-   **MiniMax**：API 密钥（编码计划密钥；`MINIMAX_CODE_PLAN_KEY` 或 `MINIMAX_API_KEY`）；使用 5 小时编码计划窗口。
-   **z.ai**：通过环境变量/配置/认证存储的 API 密钥。

如果不存在匹配的 OAuth/API 凭证，则隐藏使用量信息。

[输入指示器](./typing-indicators.md)[时区](./timezone.md)