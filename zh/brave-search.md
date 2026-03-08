

  内置工具

  
# Brave 搜索

OpenClaw 支持将 Brave Search 作为 `web_search` 的网页搜索提供商。

## 获取 API 密钥

1.  在 [https://brave.com/search/api/](https://brave.com/search/api/) 创建 Brave Search API 账户。
2.  在仪表板中，选择 **Data for Search** 计划并生成一个 API 密钥。
3.  将密钥存储在配置中（推荐）或在 Gateway 环境中设置 `BRAVE_API_KEY`。

## 配置示例

```json
{
  tools: {
    web: {
      search: {
        provider: "brave",
        apiKey: "BRAVE_API_KEY_HERE",
        maxResults: 5,
        timeoutSeconds: 30,
      },
    },
  },
}
```

## 工具参数

| 参数 | 描述 |
| --- | --- |
| `query` | 搜索查询（必需） |
| `count` | 要返回的结果数量（1-10，默认：5） |
| `country` | 2 字母 ISO 国家代码（例如，“US”、“DE”） |
| `language` | 搜索结果的 ISO 639-1 语言代码（例如，“en”、“de”、“fr”） |
| `ui_lang` | UI 元素的 ISO 语言代码 |
| `freshness` | 时间过滤器：`day`（24小时）、`week`、`month` 或 `year` |
| `date_after` | 仅在此日期之后发布的结果（YYYY-MM-DD） |
| `date_before` | 仅在此日期之前发布的结果（YYYY-MM-DD） |

**示例：**

```
// 特定国家和语言的搜索
await web_search({
  query: "renewable energy",
  country: "DE",
  language: "de",
});

// 近期结果（过去一周）
await web_search({
  query: "AI news",
  freshness: "week",
});

// 日期范围搜索
await web_search({
  query: "AI developments",
  date_after: "2024-01-01",
  date_before: "2024-06-30",
});
```

## 注意事项

-   Data for AI 计划**不**兼容 `web_search`。
-   Brave 提供付费计划；请在 Brave API 门户中查看当前限制。
-   Brave 条款包含对某些 AI 相关用途使用搜索结果的限制。请查阅 Brave 服务条款并确认您的预期用途符合规定。对于法律问题，请咨询您的法律顾问。
-   结果默认缓存 15 分钟（可通过 `cacheTtlMinutes` 配置）。

完整 web_search 配置请参见 [Web 工具](./tools/web.md)。

[apply\_patch 工具](./tools/apply-patch.md)[Perplexity Sonar](./perplexity.md)