

  組み込みツール

  
# PDFツール

`pdf` ツールは、1つまたは複数のPDF文書を分析し、テキストを返します。主な動作:

-   AnthropicおよびGoogleモデルプロバイダー向けのネイティブプロバイダーモード。
-   その他のプロバイダー向けの抽出フォールバックモード（まずテキストを抽出し、必要に応じてページ画像を処理）。
-   単一入力 (`pdf`) または複数入力 (`pdfs`) をサポート、呼び出しあたり最大10個のPDF。

## 利用可能性

このツールは、OpenClawがエージェント用にPDF対応モデル設定を解決できる場合にのみ登録されます:

1.  `agents.defaults.pdfModel`
2.  フォールバック: `agents.defaults.imageModel`
3.  フォールバック: 利用可能な認証情報に基づくベストエフォートのプロバイダーデフォルト

使用可能なモデルが解決できない場合、`pdf` ツールは公開されません。

## 入力リファレンス

-   `pdf` (`string`): 1つのPDFパスまたはURL
-   `pdfs` (`string[]`): 複数のPDFパスまたはURL、合計最大10個
-   `prompt` (`string`): 分析プロンプト、デフォルト `Analyze this PDF document.`
-   `pages` (`string`): ページフィルター（例: `1-5` または `1,3,7-9`）
-   `model` (`string`): オプションのモデル上書き (`provider/model`)
-   `maxBytesMb` (`number`): PDFあたりのサイズ上限（MB単位）

入力に関する注意:

-   `pdf` と `pdfs` はロード前にマージされ、重複が削除されます。
-   PDF入力が提供されない場合、ツールはエラーを返します。
-   `pages` は1から始まるページ番号として解析され、重複削除、ソート、設定された最大ページ数にクランプされます。
-   `maxBytesMb` のデフォルトは `agents.defaults.pdfMaxBytesMb` または `10` です。

## サポートされるPDF参照

-   ローカルファイルパス (`~` 展開を含む)
-   `file://` URL
-   `http://` および `https://` URL

参照に関する注意:

-   その他のURIスキーム（例: `ftp://`）は `unsupported_pdf_reference` で拒否されます。
-   サンドボックスモードでは、リモートの `http(s)` URLは拒否されます。
-   ワークスペース専用ファイルポリシーが有効な場合、許可されたルート外のローカルファイルパスは拒否されます。

## 実行モード

### ネイティブプロバイダーモード

ネイティブモードは、プロバイダー `anthropic` および `google` に対して使用されます。ツールは生のPDFバイトを直接プロバイダーAPIに送信します。ネイティブモードの制限:

-   `pages` はサポートされていません。設定されている場合、ツールはエラーを返します。

### 抽出フォールバックモード

フォールバックモードは、非ネイティブプロバイダーに対して使用されます。フロー:

1.  選択されたページからテキストを抽出します（最大 `agents.defaults.pdfMaxPages`、デフォルト `20`）。
2.  抽出されたテキストの長さが `200` 文字未満の場合、選択されたページをPNG画像にレンダリングして含めます。
3.  抽出されたコンテンツとプロンプトを選択されたモデルに送信します。

フォールバックの詳細:

-   ページ画像の抽出には `4,000,000` ピクセルの予算を使用します。
-   ターゲットモデルが画像入力をサポートしておらず、抽出可能なテキストもない場合、ツールはエラーを返します。
-   抽出フォールバックには `pdfjs-dist`（および画像レンダリング用の `@napi-rs/canvas`）が必要です。

## 設定

```json
{
  agents: {
    defaults: {
      pdfModel: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["openai/gpt-5-mini"],
      },
      pdfMaxBytesMb: 10,
      pdfMaxPages: 20,
    },
  },
}
```

全フィールドの詳細については、[設定リファレンス](../gateway/configuration-reference.md)を参照してください。

## 出力詳細

ツールはテキストを `content[0].text` で返し、構造化メタデータを `details` で返します。一般的な `details` フィールド:

-   `model`: 解決されたモデル参照 (`provider/model`)
-   `native`: ネイティブプロバイダーモードの場合は `true`、フォールバックの場合は `false`
-   `attempts`: 成功する前に失敗したフォールバック試行回数

パスフィールド:

-   単一PDF入力: `details.pdf`
-   複数PDF入力: `pdf` エントリを持つ `details.pdfs[]`
-   サンドボックスパス書き換えメタデータ（該当する場合）: `rewrittenFrom`

## エラー動作

-   PDF入力がない: `pdf required: provide a path or URL to a PDF document` をスロー
-   多すぎるPDF: `details.error = "too_many_pdfs"` の構造化エラーを返す
-   サポートされていない参照スキーム: `details.error = "unsupported_pdf_reference"` を返す
-   ネイティブモードで `pages` を指定: `pages is not supported with native PDF providers` エラーを明確にスロー

## 例

単一PDF:

```json
{
  "pdf": "/tmp/report.pdf",
  "prompt": "Summarize this report in 5 bullets"
}
```

複数PDF:

```json
{
  "pdfs": ["/tmp/q1.pdf", "/tmp/q2.pdf"],
  "prompt": "Compare risks and timeline changes across both documents"
}
```

ページフィルター付きフォールバックモデル:

```json
{
  "pdf": "https://example.com/report.pdf",
  "pages": "1-3,7",
  "model": "openai/gpt-5-mini",
  "prompt": "Extract only customer-impacting incidents"
}
```

[差分](./diffs.md)[昇格モード](./elevated.md)