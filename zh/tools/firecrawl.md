

  内置工具

  
# Firecrawl

OpenClaw 可以将 **Firecrawl** 用作 `web_fetch` 的备用提取器。它是一个托管的内容提取服务，支持绕过机器人检测和缓存，有助于处理 JavaScript 密集型网站或阻止普通 HTTP 抓取的页面。

## 获取 API 密钥

1.  创建一个 Firecrawl 账户并生成一个 API 密钥。
2.  将其存储在配置中或在网关环境中设置 `FIRECRAWL_API_KEY`。

## 配置 Firecrawl

```json
{
  tools: {
    web: {
      fetch: {
        firecrawl: {
          apiKey: "FIRECRAWL_API_KEY_HERE",
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 172800000,
          timeoutSeconds: 60,
        },
      },
    },
  },
}
```

注意事项：

-   当存在 API 密钥时，`firecrawl.enabled` 默认为 true。
-   `maxAgeMs` 控制缓存结果的有效时长（毫秒）。默认为 2 天。

## 隐身 / 机器人规避

Firecrawl 为机器人规避提供了一个 **代理模式** 参数 (`basic`、`stealth` 或 `auto`)。OpenClaw 始终为 Firecrawl 请求使用 `proxy: "auto"` 加上 `storeInCache: true`。如果省略代理参数，Firecrawl 默认为 `auto`。`auto` 模式会在基本尝试失败时使用隐身代理重试，这可能比仅使用基本模式抓取消耗更多积分。

## web\_fetch 如何使用 Firecrawl

`web_fetch` 提取顺序：

1.  Readability（本地）
2.  Firecrawl（如果已配置）
3.  基本 HTML 清理（最后备选）

完整网页工具设置请参阅[网页工具](./web.md)。

[执行审批](./exec-approvals.md)[LLM 任务](./llm-task.md)

---