

  組み込みツール

  
# Webツール

OpenClawには2つの軽量なWebツールが同梱されています:

-   `web_search` — Perplexity Search API、Brave Search API、Google Searchグラウンディングを備えたGemini、Grok、またはKimiを使用してウェブを検索します。
-   `web_fetch` — HTTPフェッチ + 読み取り可能な抽出 (HTML → マークダウン/テキスト)。

これらは**ブラウザ自動化ではありません**。JavaScriptが多用されるサイトやログインが必要なサイトには、[Browserツール](./browser.md)を使用してください。

## 仕組み

-   `web_search`は設定されたプロバイダーを呼び出し、結果を返します。
-   結果はクエリごとに15分間キャッシュされます（設定可能）。
-   `web_fetch`はプレーンなHTTP GETを実行し、読み取り可能なコンテンツを抽出します（HTML → マークダウン/テキスト）。JavaScriptは実行**しません**。
-   `web_fetch`はデフォルトで有効です（明示的に無効にしない限り）。

プロバイダー固有の詳細については、[Perplexity Searchの設定](../perplexity.md)および[Brave Searchの設定](../brave-search.md)を参照してください。

## 検索プロバイダーの選択

| プロバイダー | 長所 | 短所 | APIキー |
| --- | --- | --- | --- |
| **Perplexity Search API** | 高速、構造化された結果；ドメイン、言語、地域、鮮度フィルター；コンテンツ抽出 | — | `PERPLEXITY_API_KEY` |
| **Brave Search API** | 高速、構造化された結果 | フィルタリングオプションが少ない；AI利用規約が適用される | `BRAVE_API_KEY` |
| **Gemini** | Google Searchグラウンディング、AIによる統合 | Gemini APIキーが必要 | `GEMINI_API_KEY` |
| **Grok** | xAIのウェブグラウンディング応答 | xAI APIキーが必要 | `XAI_API_KEY` |
| **Kimi** | Moonshotのウェブ検索機能 | Moonshot APIキーが必要 | `KIMI_API_KEY` / `MOONSHOT_API_KEY` |

### 自動検出

`provider`が明示的に設定されていない場合、OpenClawは利用可能なAPIキーに基づいて使用するプロバイダーを自動検出します。以下の順序でチェックします:

1.  **Brave** — `BRAVE_API_KEY` 環境変数または `tools.web.search.apiKey` 設定
2.  **Gemini** — `GEMINI_API_KEY` 環境変数または `tools.web.search.gemini.apiKey` 設定
3.  **Kimi** — `KIMI_API_KEY` / `MOONSHOT_API_KEY` 環境変数または `tools.web.search.kimi.apiKey` 設定
4.  **Perplexity** — `PERPLEXITY_API_KEY` 環境変数または `tools.web.search.perplexity.apiKey` 設定
5.  **Grok** — `XAI_API_KEY` 環境変数または `tools.web.search.grok.apiKey` 設定

キーが見つからない場合、Braveにフォールバックします（設定を促すキー不足エラーが表示されます）。

## ウェブ検索の設定

`openclaw configure --section web`を使用して、APIキーを設定し、プロバイダーを選択します。

### Perplexity Search

1.  [perplexity.ai/settings/api](https://www.perplexity.ai/settings/api) でPerplexityアカウントを作成します
2.  ダッシュボードでAPIキーを生成します
3.  `openclaw configure --section web`を実行してキーを設定に保存するか、環境変数に`PERPLEXITY_API_KEY`を設定します。

詳細は[Perplexity Search APIドキュメント](https://docs.perplexity.ai/guides/search-quickstart)を参照してください。

### Brave Search

1.  [brave.com/search/api](https://brave.com/search/api/) でBrave Search APIアカウントを作成します
2.  ダッシュボードで、**Data for Search**プラン（「Data for AI」ではない）を選択し、APIキーを生成します。
3.  `openclaw configure --section web`を実行してキーを設定に保存する（推奨）か、環境変数に`BRAVE_API_KEY`を設定します。

Braveは有料プランを提供しています。現在の制限と価格についてはBrave APIポータルを確認してください。

### キーの保存場所

**設定経由（推奨）:** `openclaw configure --section web`を実行します。キーは`tools.web.search.perplexity.apiKey`または`tools.web.search.apiKey`の下に保存されます。
**環境変数経由:** Gatewayプロセスの環境に`PERPLEXITY_API_KEY`または`BRAVE_API_KEY`を設定します。ゲートウェイインストールの場合、`~/.openclaw/.env`（またはサービスの環境）に配置します。[環境変数](../help/faq.md#how-does-openclaw-load-environment-variables)を参照してください。

### 設定例

**Perplexity Search:**

```json
{
  tools: {
    web: {
      search: {
        enabled: true,
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...", // PERPLEXITY_API_KEYが設定されている場合はオプション
        },
      },
    },
  },
}
```

**Brave Search:**

```json
{
  tools: {
    web: {
      search: {
        enabled: true,
        provider: "brave",
        apiKey: "YOUR_BRAVE_API_KEY", // BRAVE_API_KEYが設定されている場合はオプション // pragma: allowlist secret
      },
    },
  },
}
```

## Geminiの使用（Google Searchグラウンディング）

Geminiモデルは組み込みの[Google Searchグラウンディング](https://ai.google.dev/gemini-api/docs/grounding)をサポートしており、引用付きのライブGoogle検索結果に裏付けられたAI統合回答を返します。

### Gemini APIキーの取得

1.  [Google AI Studio](https://aistudio.google.com/apikey)にアクセスします
2.  APIキーを作成します
3.  Gateway環境に`GEMINI_API_KEY`を設定するか、`tools.web.search.gemini.apiKey`を設定します

### Gemini検索の設定

```json
{
  tools: {
    web: {
      search: {
        provider: "gemini",
        gemini: {
          // APIキー (GEMINI_API_KEYが設定されている場合はオプション)
          apiKey: "AIza...",
          // モデル (デフォルトは "gemini-2.5-flash")
          model: "gemini-2.5-flash",
        },
      },
    },
  },
}
```

**環境変数の代替方法:** Gateway環境に`GEMINI_API_KEY`を設定します。ゲートウェイインストールの場合、`~/.openclaw/.env`に配置します。

### 注意点

-   Geminiグラウンディングからの引用URLは、GoogleのリダイレクトURLから直接URLに自動的に解決されます。
-   リダイレクト解決は、最終的な引用URLを返す前に、SSRFガードパス（HEAD + リダイレクトチェック + http/https検証）を使用します。
-   リダイレクト解決は厳格なSSRFデフォルトを使用するため、プライベート/内部ターゲットへのリダイレクトはブロックされます。
-   デフォルトモデル（`gemini-2.5-flash`）は高速でコスト効率が良いです。グラウンディングをサポートする任意のGeminiモデルを使用できます。

## web\_search

設定されたプロバイダーを使用してウェブを検索します。

### 必要条件

-   `tools.web.search.enabled`が`false`であってはならない（デフォルト: 有効）
-   選択したプロバイダーのAPIキー:
    -   **Brave**: `BRAVE_API_KEY` または `tools.web.search.apiKey`
    -   **Perplexity**: `PERPLEXITY_API_KEY` または `tools.web.search.perplexity.apiKey`
    -   **Gemini**: `GEMINI_API_KEY` または `tools.web.search.gemini.apiKey`
    -   **Grok**: `XAI_API_KEY` または `tools.web.search.grok.apiKey`
    -   **Kimi**: `KIMI_API_KEY`、`MOONSHOT_API_KEY`、または `tools.web.search.kimi.apiKey`

### 設定

```json
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "BRAVE_API_KEY_HERE", // BRAVE_API_KEYが設定されている場合はオプション
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
      },
    },
  },
}
```

### ツールパラメータ

特に明記されていない限り、すべてのパラメータはBraveとPerplexityの両方で機能します。

| パラメータ | 説明 |
| --- | --- |
| `query` | 検索クエリ（必須） |
| `count` | 返す結果数（1-10、デフォルト: 5） |
| `country` | 2文字のISO国コード（例: "US"、"DE"） |
| `language` | ISO 639-1言語コード（例: "en"、"de"） |
| `freshness` | 時間フィルター: `day`、`week`、`month`、`year` |
| `date_after` | この日付以降の結果（YYYY-MM-DD） |
| `date_before` | この日付以前の結果（YYYY-MM-DD） |
| `ui_lang` | UI言語コード（Braveのみ） |
| `domain_filter` | ドメイン許可リスト/拒否リスト配列（Perplexityのみ） |
| `max_tokens` | 合計コンテンツ予算、デフォルト25000（Perplexityのみ） |
| `max_tokens_per_page` | ページごとのトークン制限、デフォルト2048（Perplexityのみ） |

**例:**

```
// ドイツ語固有の検索
await web_search({
  query: "TV online schauen",
  country: "DE",
  language: "de",
});

// 最近の結果（過去1週間）
await web_search({
  query: "TMBG interview",
  freshness: "week",
});

// 日付範囲検索
await web_search({
  query: "AI developments",
  date_after: "2024-01-01",
  date_before: "2024-06-30",
});

// ドメインフィルタリング（Perplexityのみ）
await web_search({
  query: "climate research",
  domain_filter: ["nature.com", "science.org", ".edu"],
});

// ドメイン除外（Perplexityのみ）
await web_search({
  query: "product reviews",
  domain_filter: ["-reddit.com", "-pinterest.com"],
});

// より多くのコンテンツ抽出（Perplexityのみ）
await web_search({
  query: "detailed AI research",
  max_tokens: 50000,
  max_tokens_per_page: 4096,
});
```

## web\_fetch

URLをフェッチし、読み取り可能なコンテンツを抽出します。

### web\_fetchの必要条件

-   `tools.web.fetch.enabled`が`false`であってはならない（デフォルト: 有効）
-   オプションのFirecrawlフォールバック: `tools.web.fetch.firecrawl.apiKey`または`FIRECRAWL_API_KEY`を設定します。

### web\_fetch設定

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
          apiKey: "FIRECRAWL_API_KEY_HERE", // FIRECRAWL_API_KEYが設定されている場合はオプション
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 86400000, // ミリ秒 (1日)
          timeoutSeconds: 60,
        },
      },
    },
  },
}
```

### web\_fetchツールパラメータ

-   `url`（必須、http/httpsのみ）
-   `extractMode`（`markdown` | `text`）
-   `maxChars`（長いページを切り詰める）

注意点:

-   `web_fetch`はまずReadability（メインコンテンツ抽出）を使用し、次にFirecrawl（設定されている場合）を使用します。両方とも失敗した場合、ツールはエラーを返します。
-   Firecrawlリクエストはデフォルトでボット回避モードを使用し、結果をキャッシュします。
-   `web_fetch`はデフォルトでChromeライクなUser-Agentと`Accept-Language`を送信します。必要に応じて`userAgent`をオーバーライドしてください。
-   `web_fetch`はプライベート/内部ホスト名をブロックし、リダイレクトを再チェックします（`maxRedirects`で制限）。
-   `maxChars`は`tools.web.fetch.maxCharsCap`にクランプされます。
-   `web_fetch`は解析前にダウンロードしたレスポンスボディサイズを`tools.web.fetch.maxResponseBytes`に制限します。サイズ超過のレスポンスは切り詰められ、警告が含まれます。
-   `web_fetch`はベストエフォートの抽出です。一部のサイトではブラウザツールが必要になります。
-   キーの設定とサービスの詳細については[Firecrawl](./firecrawl.md)を参照してください。
-   レスポンスはキャッシュされ（デフォルト15分）、繰り返しフェッチを減らします。
-   ツールプロファイル/許可リストを使用する場合は、`web_search`/`web_fetch`または`group:web`を追加してください。
-   APIキーがない場合、`web_search`はドキュメントリンク付きの短い設定ヒントを返します。

[思考レベル](./thinking.md)[ブラウザ（OpenClaw管理）](./browser.md)