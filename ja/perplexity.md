

  組み込みツール

  
# Perplexity Sonar

OpenClawは`web_search`ツールにPerplexity Sonarを使用できます。Perplexityの直接APIまたはOpenRouter経由で接続できます。

## APIオプション

### Perplexity (直接)

-   ベースURL: [https://api.perplexity.ai](https://api.perplexity.ai)
-   環境変数: `PERPLEXITY_API_KEY`

### OpenRouter (代替)

-   ベースURL: [https://openrouter.ai/api/v1](https://openrouter.ai/api/v1)
-   環境変数: `OPENROUTER_API_KEY`
-   プリペイド/暗号通貨クレジットをサポート。

## 設定例

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

## Braveからの切り替え

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

`PERPLEXITY_API_KEY`と`OPENROUTER_API_KEY`の両方が設定されている場合、`tools.web.search.perplexity.baseUrl`（または`tools.web.search.perplexity.apiKey`）を設定して区別します。ベースURLが設定されていない場合、OpenClawはAPIキーのソースに基づいてデフォルトを選択します:

-   `PERPLEXITY_API_KEY` または `pplx-...` → 直接Perplexity (`https://api.perplexity.ai`)
-   `OPENROUTER_API_KEY` または `sk-or-...` → OpenRouter (`https://openrouter.ai/api/v1`)
-   不明なキー形式 → OpenRouter (安全なフォールバック)

## モデル

-   `perplexity/sonar` — Web検索付き高速Q&A
-   `perplexity/sonar-pro` (デフォルト) — 多段階推論 + Web検索
-   `perplexity/sonar-reasoning-pro` — 深い調査

完全なweb\_search設定については、[Webツール](./tools/web.md)を参照してください。

[Brave検索](./brave-search.md)[差分](./tools/diffs.md)