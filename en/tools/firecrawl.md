

  Built-in tools

  
# Firecrawl

OpenClaw can use **Firecrawl** as a fallback extractor for `web_fetch`. It is a hosted content extraction service that supports bot circumvention and caching, which helps with JS-heavy sites or pages that block plain HTTP fetches.

## Get an API key

1.  Create a Firecrawl account and generate an API key.
2.  Store it in config or set `FIRECRAWL_API_KEY` in the gateway environment.

## Configure Firecrawl

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

Notes:

-   `firecrawl.enabled` defaults to true when an API key is present.
-   `maxAgeMs` controls how old cached results can be (ms). Default is 2 days.

## Stealth / bot circumvention

Firecrawl exposes a **proxy mode** parameter for bot circumvention (`basic`, `stealth`, or `auto`). OpenClaw always uses `proxy: "auto"` plus `storeInCache: true` for Firecrawl requests. If proxy is omitted, Firecrawl defaults to `auto`. `auto` retries with stealth proxies if a basic attempt fails, which may use more credits than basic-only scraping.

## How web\_fetch uses Firecrawl

`web_fetch` extraction order:

1.  Readability (local)
2.  Firecrawl (if configured)
3.  Basic HTML cleanup (last fallback)

See [Web tools](./web.md) for the full web tool setup.

[Exec Approvals](./exec-approvals.md)[LLM Task](./llm-task.md)
