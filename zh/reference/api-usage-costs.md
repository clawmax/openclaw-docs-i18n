

  技术参考

  
# API 使用与成本

本文档列出了**可能调用 API 密钥的功能**及其成本显示位置。主要关注可能产生供应商使用量或付费 API 调用的 OpenClaw 功能。

## 成本显示位置（聊天 + CLI）

**每会话成本快照**

-   `/status` 显示当前会话的模型、上下文使用情况和上次响应的令牌数。
-   如果模型使用 **API 密钥认证**，`/status` 还会显示**上次回复的预估成本**。

**每条消息的成本页脚**

-   `/usage full` 会在每次回复后附加一个使用情况页脚，包括**预估成本**（仅限 API 密钥）。
-   `/usage tokens` 仅显示令牌数；OAuth 流程会隐藏美元成本。

**CLI 使用量窗口（供应商配额）**

-   `openclaw status --usage` 和 `openclaw channels list` 显示供应商**使用量窗口**（配额快照，非每条消息成本）。

详情和示例请参阅[令牌使用与成本](./token-use.md)。

## 密钥的发现方式

OpenClaw 可以从以下位置获取凭据：

-   **认证配置文件**（按代理，存储在 `auth-profiles.json` 中）。
-   **环境变量**（例如 `OPENAI_API_KEY`、`BRAVE_API_KEY`、`FIRECRAWL_API_KEY`）。
-   **配置**（`models.providers.*.apiKey`、`tools.web.search.*`、`tools.web.fetch.firecrawl.*`、`memorySearch.*`、`talk.apiKey`）。
-   **技能**（`skills.entries..apiKey`），这些密钥可能会导出到技能进程的环境变量中。

## 可能消耗密钥的功能

### 1) 核心模型响应（聊天 + 工具）

每次回复或工具调用都会使用**当前模型供应商**（OpenAI、Anthropic 等）。这是使用量和成本的主要来源。定价配置请参阅[模型](../providers/models.md)，显示方式请参阅[令牌使用与成本](./token-use.md)。

### 2) 媒体理解（音频/图像/视频）

传入的媒体可以在回复运行前进行摘要/转录。这会使用模型/供应商 API。

-   音频：OpenAI / Groq / Deepgram（现在当密钥存在时**自动启用**）。
-   图像：OpenAI / Anthropic / Google。
-   视频：Google。

请参阅[媒体理解](../nodes/media-understanding.md)。

### 3) 记忆嵌入 + 语义搜索

语义记忆搜索在配置为远程供应商时使用**嵌入 API**：

-   `memorySearch.provider = "openai"` → OpenAI 嵌入
-   `memorySearch.provider = "gemini"` → Gemini 嵌入
-   `memorySearch.provider = "voyage"` → Voyage 嵌入
-   `memorySearch.provider = "mistral"` → Mistral 嵌入
-   `memorySearch.provider = "ollama"` → Ollama 嵌入（本地/自托管；通常无托管 API 计费）
-   如果本地嵌入失败，可选择回退到远程供应商

您可以通过 `memorySearch.provider = "local"` 保持本地运行（无 API 使用）。请参阅[记忆](../concepts/memory.md)。

### 4) 网络搜索工具

`web_search` 使用 API 密钥，并可能根据您的供应商产生使用费：

-   **Perplexity Search API**: `PERPLEXITY_API_KEY`
-   **Brave Search API**: `BRAVE_API_KEY` 或 `tools.web.search.apiKey`
-   **Gemini (Google Search)**: `GEMINI_API_KEY`
-   **Grok (xAI)**: `XAI_API_KEY`
-   **Kimi (Moonshot)**: `KIMI_API_KEY` 或 `MOONSHOT_API_KEY`

请参阅[网络工具](../tools/web.md)。

### 5) 网页抓取工具（Firecrawl）

`web_fetch` 在 API 密钥存在时可以调用 **Firecrawl**：

-   `FIRECRAWL_API_KEY` 或 `tools.web.fetch.firecrawl.apiKey`

如果未配置 Firecrawl，该工具将回退到直接抓取 + readability（无付费 API）。请参阅[网络工具](../tools/web.md)。

### 6) 供应商使用量快照（状态/健康度）

某些状态命令会调用**供应商使用量端点**以显示配额窗口或认证健康度。这些通常是低流量调用，但仍会访问供应商 API：

-   `openclaw status --usage`
-   `openclaw models status --json`

请参阅[模型 CLI](../cli/models.md)。

### 7) 压缩保护摘要

压缩保护功能可以使用**当前模型**来总结会话历史记录，运行时将调用供应商 API。请参阅[会话管理与压缩](./session-management-compaction.md)。

### 8) 模型扫描 / 探测

`openclaw models scan` 可以探测 OpenRouter 模型，并在启用探测时使用 `OPENROUTER_API_KEY`。请参阅[模型 CLI](../cli/models.md)。

### 9) 对话（语音）

对话模式在配置时可以调用 **ElevenLabs**：

-   `ELEVENLABS_API_KEY` 或 `talk.apiKey`

请参阅[对话模式](../nodes/talk.md)。

### 10) 技能（第三方 API）

技能可以在 `skills.entries..apiKey` 中存储 `apiKey`。如果技能使用该密钥调用外部 API，则可能根据技能的供应商产生成本。请参阅[技能](../tools/skills.md)。

[提示缓存](./prompt-caching.md)[记录整理](./transcript-hygiene.md)