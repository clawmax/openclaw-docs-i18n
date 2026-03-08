

  組み込みツール

  
# Brave Search

OpenClawは、`web_search`のためのWeb検索プロバイダーとしてBrave Searchをサポートしています。

## APIキーを取得する

1.  [https://brave.com/search/api/](https://brave.com/search/api/) でBrave Search APIアカウントを作成します。
2.  ダッシュボードで、**Data for Search**プランを選択し、APIキーを生成します。
3.  キーを設定ファイルに保存する（推奨）か、Gateway環境で`BRAVE_API_KEY`を設定します。

## 設定例

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

## ツールパラメータ

| パラメータ | 説明 |
| --- | --- |
| `query` | 検索クエリ（必須） |
| `count` | 返す結果の数（1-10、デフォルト: 5） |
| `country` | 2文字のISO国コード（例: "US"、"DE"） |
| `language` | 検索結果のISO 639-1言語コード（例: "en"、"de"、"fr"） |
| `ui_lang` | UI要素のためのISO言語コード |
| `freshness` | 時間フィルター: `day`（24時間）、`week`、`month`、`year` |
| `date_after` | この日付以降に公開された結果のみ（YYYY-MM-DD） |
| `date_before` | この日付以前に公開された結果のみ（YYYY-MM-DD） |

**例:**

```javascript
// 国と言語を指定した検索
await web_search({
  query: "再生可能エネルギー",
  country: "DE",
  language: "de",
});

// 最近の結果（過去1週間）
await web_search({
  query: "AIニュース",
  freshness: "week",
});

// 日付範囲を指定した検索
await web_search({
  query: "AIの進展",
  date_after: "2024-01-01",
  date_before: "2024-06-30",
});
```

## 注意事項

-   Data for AIプランは`web_search`と**互換性がありません**。
-   Braveは有料プランを提供しています。現在の制限についてはBrave APIポータルで確認してください。
-   Braveの利用規約には、検索結果の一部のAI関連利用に関する制限が含まれています。Braveの利用規約を確認し、ご自身の使用目的が準拠していることを確認してください。法的な質問については、弁護士に相談してください。
-   結果はデフォルトで15分間キャッシュされます（`cacheTtlMinutes`で設定可能）。

web_searchの完全な設定については、[Webツール](./tools/web.md)を参照してください。

[apply\_patch ツール](./tools/apply-patch.md)[Perplexity Sonar](./perplexity.md)

---