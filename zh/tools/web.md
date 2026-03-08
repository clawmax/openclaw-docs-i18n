

  内置工具

  
# 网页工具

OpenClaw 内置了两个轻量级网页工具：

-   `web_search` — 使用 Perplexity Search API、Brave Search API、Gemini（带 Google 搜索联网）、Grok 或 Kimi 搜索网页。
-   `web_fetch` — HTTP 抓取 + 可读内容提取（HTML → Markdown/文本）。

这些**不是**浏览器自动化工具。对于重度依赖 JavaScript 的网站或需要登录的情况，请使用[浏览器工具](./browser.md)。

## 工作原理

-   `web_search` 调用您配置的提供商并返回结果。
-   结果按查询缓存 15 分钟（可配置）。
-   `web_fetch` 执行普通的 HTTP GET 请求并提取可读内容（HTML → Markdown/文本）。它**不**执行 JavaScript。
-   `web_fetch` 默认启用（除非显式禁用）。

有关特定提供商的详细信息，请参阅 [Perplexity 搜索设置](../perplexity.md) 和 [Brave 搜索设置](../brave-search.md)。

## 选择搜索提供商

| 提供商 | 优点 | 缺点 | API 密钥 |
| --- | --- | --- | --- |
| **Perplexity Search API** | 快速，结构化结果；支持域名、语言、地区和新鲜度过滤；内容提取 | — | `PERPLEXITY_API_KEY` |
| **Brave Search API** | 快速，结构化结果 | 过滤选项较少；适用 AI 使用条款 | `BRAVE_API_KEY` |
| **Gemini** | Google 搜索联网，AI 综合回答 | 需要 Gemini API 密钥 | `GEMINI_API_KEY` |
| **Grok** | xAI 联网回答 | 需要 xAI API 密钥 | `XAI_API_KEY` |
| **Kimi** | Moonshot 网页搜索能力 | 需要 Moonshot API 密钥 | `KIMI_API_KEY` / `MOONSHOT_API_KEY` |

### 自动检测

如果未显式设置 `provider`，OpenClaw 会根据可用的 API 密钥自动检测使用哪个提供商，按此顺序检查：

1.  **Brave** — `BRAVE_API_KEY` 环境变量或 `tools.web.search.apiKey` 配置
2.  **Gemini** — `GEMINI_API_KEY` 环境变量或 `tools.web.search.gemini.apiKey` 配置
3.  **Kimi** — `KIMI_API_KEY` / `MOONSHOT_API_KEY` 环境变量或 `tools.web.search.kimi.apiKey` 配置
4.  **Perplexity** — `PERPLEXITY_API_KEY` 环境变量或 `tools.web.search.perplexity.apiKey` 配置
5.  **Grok** — `XAI_API_KEY` 环境变量或 `tools.web.search.grok.apiKey` 配置

如果未找到任何密钥，则回退到 Brave（您将收到缺少密钥的错误提示，引导您配置一个）。

## 设置网页搜索

使用 `openclaw configure --section web` 来设置您的 API 密钥并选择提供商。

### Perplexity 搜索

1.  在 [perplexity.ai/settings/api](https://www.perplexity.ai/settings/api) 创建 Perplexity 账户
2.  在仪表板中生成 API 密钥
3.  运行 `openclaw configure --section web` 将密钥存储在配置中，或在环境中设置 `PERPLEXITY_API_KEY`。

更多详情请参阅 [Perplexity Search API 文档](https://docs.perplexity.ai/guides/search-quickstart)。

### Brave 搜索

1.  在 [brave.com/search/api](https://brave.com/search/api/) 创建 Brave Search API 账户
2.  在仪表板中，选择 **Data for Search** 计划（非“Data for AI”）并生成 API 密钥。
3.  运行 `openclaw configure --section web` 将密钥存储在配置中（推荐），或在环境中设置 `BRAVE_API_KEY`。

Brave 提供付费计划；请查看 Brave API 门户了解当前的限制和定价。

### 密钥存储位置

**通过配置（推荐）：** 运行 `openclaw configure --section web`。它将密钥存储在 `tools.web.search.perplexity.apiKey` 或 `tools.web.search.apiKey` 下。
**通过环境变量：** 在 Gateway 进程环境中设置 `PERPLEXITY_API_KEY` 或 `BRAVE_API_KEY`。对于网关安装，将其放在 `~/.openclaw/.env`（或您的服务环境）中。参见[环境变量](../help/faq.md#how-does-openclaw-load-environment-variables)。

### 配置示例

**Perplexity 搜索：**

```json
{
  tools: {
    web: {
      search: {
        enabled: true,
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...", // 如果设置了 PERPLEXITY_API_KEY 则为可选
        },
      },
    },
  },
}
```

**Brave 搜索：**

```json
{
  tools: {
    web: {
      search: {
        enabled: true,
        provider: "brave",
        apiKey: "YOUR_BRAVE_API_KEY", // 如果设置了 BRAVE_API_KEY 则为可选 // pragma: allowlist secret
      },
    },
  },
}
```

## 使用 Gemini（Google 搜索联网）

Gemini 模型支持内置的 [Google 搜索联网](https://ai.google.dev/gemini-api/docs/grounding)，该功能返回由实时 Google 搜索结果支持并带有引用的 AI 综合答案。

### 获取 Gemini API 密钥

1.  访问 [Google AI Studio](https://aistudio.google.com/apikey)
2.  创建 API 密钥
3.  在 Gateway 环境中设置 `GEMINI_API_KEY`，或配置 `tools.web.search.gemini.apiKey`

### 设置 Gemini 搜索

```json
{
  tools: {
    web: {
      search: {
        provider: "gemini",
        gemini: {
          // API 密钥（如果设置了 GEMINI_API_KEY 则为可选）
          apiKey: "AIza...",
          // 模型（默认为 "gemini-2.5-flash"）
          model: "gemini-2.5-flash",
        },
      },
    },
  },
}
```

**环境变量替代方案：** 在 Gateway 环境中设置 `GEMINI_API_KEY`。对于网关安装，将其放在 `~/.openclaw/.env` 中。

### 注意事项

-   Gemini 联网返回的引用 URL 会自动从 Google 的重定向 URL 解析为直接 URL。
-   重定向解析使用 SSRF 防护路径（HEAD + 重定向检查 + http/https 验证）后才返回最终的引用 URL。
-   重定向解析使用严格的 SSRF 默认设置，因此重定向到私有/内部目标会被阻止。
-   默认模型（`gemini-2.5-flash`）速度快且性价比高。任何支持联网的 Gemini 模型都可以使用。

## web\_search

使用您配置的提供商搜索网页。

### 要求

-   `tools.web.search.enabled` 不能为 `false`（默认：启用）
-   所选提供商的 API 密钥：
    -   **Brave**: `BRAVE_API_KEY` 或 `tools.web.search.apiKey`
    -   **Perplexity**: `PERPLEXITY_API_KEY` 或 `tools.web.search.perplexity.apiKey`
    -   **Gemini**: `GEMINI_API_KEY` 或 `tools.web.search.gemini.apiKey`
    -   **Grok**: `XAI_API_KEY` 或 `tools.web.search.grok.apiKey`
    -   **Kimi**: `KIMI_API_KEY`、`MOONSHOT_API_KEY` 或 `tools.web.search.kimi.apiKey`

### 配置

```json
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "BRAVE_API_KEY_HERE", // 如果设置了 BRAVE_API_KEY 则为可选
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
      },
    },
  },
}
```

### 工具参数

除非特别说明，所有参数对 Brave 和 Perplexity 都适用。

| 参数 | 描述 |
| --- | --- |
| `query` | 搜索查询（必需） |
| `count` | 返回的结果数（1-10，默认：5） |
| `country` | 2 字母 ISO 国家代码（例如，“US”、“DE”） |
| `language` | ISO 639-1 语言代码（例如，“en”、“de”） |
| `freshness` | 时间过滤器：`day`、`week`、`month` 或 `year` |
| `date_after` | 此日期之后的结果（YYYY-MM-DD） |
| `date_before` | 此日期之前的结果（YYYY-MM-DD） |
| `ui_lang` | UI 语言代码（仅限 Brave） |
| `domain_filter` | 域名允许列表/拒绝列表数组（仅限 Perplexity） |
| `max_tokens` | 总内容预算，默认 25000（仅限 Perplexity） |
| `max_tokens_per_page` | 每页令牌限制，默认 2048（仅限 Perplexity） |

**示例：**

```
// 德语特定搜索
await web_search({
  query: "TV online schauen",
  country: "DE",
  language: "de",
});

// 近期结果（过去一周）
await web_search({
  query: "TMBG interview",
  freshness: "week",
});

// 日期范围搜索
await web_search({
  query: "AI developments",
  date_after: "2024-01-01",
  date_before: "2024-06-30",
});

// 域名过滤（仅限 Perplexity）
await web_search({
  query: "climate research",
  domain_filter: ["nature.com", "science.org", ".edu"],
});

// 排除域名（仅限 Perplexity）
await web_search({
  query: "product reviews",
  domain_filter: ["-reddit.com", "-pinterest.com"],
});

// 更多内容提取（仅限 Perplexity）
await web_search({
  query: "detailed AI research",
  max_tokens: 50000,
  max_tokens_per_page: 4096,
});
```

## web\_fetch

抓取 URL 并提取可读内容。

### web\_fetch 要求

-   `tools.web.fetch.enabled` 不能为 `false`（默认：启用）
-   可选的 Firecrawl 后备：设置 `tools.web.fetch.firecrawl.apiKey` 或 `FIRECRAWL_API_KEY`。

### web\_fetch 配置

```json
{
  tools: {
    web: {
      fetch: {
        enabled: true,
        maxChars: 50000,
        maxCharsCap: 50000,
        maxResponseBytes: 2000000,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
        maxRedirects: 3,
        userAgent: "Mozilla/5.0 (Macintosh; Intel Mac OS X 14_7_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36",
        readability: true,
        firecrawl: {
          enabled: true,
          apiKey: "FIRECRAWL_API_KEY_HERE", // 如果设置了 FIRECRAWL_API_KEY 则为可选
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 86400000, // 毫秒（1 天）
          timeoutSeconds: 60,
        },
      },
    },
  },
}
```

### web\_fetch 工具参数

-   `url`（必需，仅限 http/https）
-   `extractMode` (`markdown` | `text`)
-   `maxChars`（截断长页面）

注意事项：

-   `web_fetch` 首先使用 Readability（主内容提取），然后使用 Firecrawl（如果已配置）。如果两者都失败，工具将返回错误。
-   Firecrawl 请求默认使用绕过机器人检测模式并缓存结果。
-   `web_fetch` 默认发送类似 Chrome 的 User-Agent 和 `Accept-Language`；如果需要，可以覆盖 `userAgent`。
-   `web_fetch` 会阻止私有/内部主机名并重新检查重定向（通过 `maxRedirects` 限制）。
-   `maxChars` 会被限制在 `tools.web.fetch.maxCharsCap` 以内。
-   `web_fetch` 在解析前将下载的响应体大小限制在 `tools.web.fetch.maxResponseBytes` 以内；过大的响应会被截断并包含警告。
-   `web_fetch` 是尽力而为的提取；某些网站需要使用浏览器工具。
-   有关密钥设置和服务详情，请参阅 [Firecrawl](./firecrawl.md)。
-   响应会被缓存（默认 15 分钟）以减少重复抓取。
-   如果您使用工具配置文件/允许列表，请添加 `web_search`/`web_fetch` 或 `group:web`。
-   如果缺少 API 密钥，`web_search` 会返回一个简短的设置提示和文档链接。

[思考层级](./thinking.md)[浏览器（OpenClaw 托管）](./browser.md)