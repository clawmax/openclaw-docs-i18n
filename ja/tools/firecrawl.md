

  組み込みツール

  
# Firecrawl

OpenClawは、`web_fetch`のフォールバック抽出ツールとして**Firecrawl**を使用できます。これは、ボット回避とキャッシュをサポートするホスト型コンテンツ抽出サービスで、JavaScriptが多用されるサイトやプレーンなHTTPフェッチをブロックするページの処理に役立ちます。

## APIキーを取得する

1.  Firecrawlアカウントを作成し、APIキーを生成します。
2.  設定ファイルに保存するか、ゲートウェイ環境で`FIRECRAWL_API_KEY`を設定します。

## Firecrawlを設定する

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

注記:

-   `firecrawl.enabled`は、APIキーが存在する場合、デフォルトでtrueになります。
-   `maxAgeMs`は、キャッシュされた結果の有効期限（ミリ秒）を制御します。デフォルトは2日間です。

## ステルス / ボット回避

Firecrawlは、ボット回避のための**プロキシモード**パラメータ（`basic`、`stealth`、`auto`）を公開しています。OpenClawはFirecrawlリクエストに対して常に`proxy: "auto"`と`storeInCache: true`を使用します。プロキシが省略された場合、Firecrawlはデフォルトで`auto`になります。`auto`は基本的な試行が失敗した場合にステルスプロキシで再試行するため、basicのみのスクレイピングよりも多くのクレジットを使用する可能性があります。

## web\_fetchがFirecrawlを使用する仕組み

`web_fetch`の抽出順序:

1.  Readability（ローカル）
2.  Firecrawl（設定されている場合）
3.  基本的なHTMLクリーンアップ（最終フォールバック）

完全なウェブツールのセットアップについては、[ウェブツール](./web.md)を参照してください。

[実行承認](./exec-approvals.md)[LLMタスク](./llm-task.md)