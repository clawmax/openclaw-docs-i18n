

  内置工具

  
# Perplexity Sonar

OpenClaw 可以使用 Perplexity Sonar 作为 `web_search` 工具。您可以通过 Perplexity 的直接 API 或通过 OpenRouter 进行连接。

## API 选项

### Perplexity (直接)

-   基础 URL: [https://api.perplexity.ai](https://api.perplexity.ai)
-   环境变量: `PERPLEXITY_API_KEY`

### OpenRouter (替代方案)

-   基础 URL: [https://openrouter.ai/api/v1](https://openrouter.ai/api/v1)
-   环境变量: `OPENROUTER_API_KEY`
-   支持预付费/加密货币积分。

## 配置示例

```json
{
  tools: {
    web: {
      search: {
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...",
          baseUrl: "https://api.perplexity.ai",
          model: "perplexity/sonar-pro",
        },
      },
    },
  },
}
```

## 从 Brave 切换

```json
{
  tools: {
    web: {
      search: {
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...",
          baseUrl: "https://api.perplexity.ai",
        },
      },
    },
  },
}
```

如果同时设置了 `PERPLEXITY_API_KEY` 和 `OPENROUTER_API_KEY`，请设置 `tools.web.search.perplexity.baseUrl`（或 `tools.web.search.perplexity.apiKey`）以消除歧义。如果未设置基础 URL，OpenClaw 将根据 API 密钥来源选择默认值：

-   `PERPLEXITY_API_KEY` 或 `pplx-...` → 直接 Perplexity (`https://api.perplexity.ai`)
-   `OPENROUTER_API_KEY` 或 `sk-or-...` → OpenRouter (`https://openrouter.ai/api/v1`)
-   未知密钥格式 → OpenRouter (安全回退)

## 模型

-   `perplexity/sonar` — 快速问答与网络搜索
-   `perplexity/sonar-pro` (默认) — 多步推理 + 网络搜索
-   `perplexity/sonar-reasoning-pro` — 深度研究

有关完整的 web\_search 配置，请参阅 [网络工具](./tools/web.md)。

[Brave 搜索](./brave-search.md)[差异](./tools/diffs.md)

---