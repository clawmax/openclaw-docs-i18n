

  組み込みツール

  
# Diffs

`diffs`は、短い組み込みシステムガイダンスと、変更内容をエージェント用の読み取り専用差分アーティファクトに変換するコンパニオンスキルを備えたオプショナルなプラグインツールです。以下のいずれかを受け入れます:

-   `before` と `after` のテキスト
-   ユニファイド `patch`

以下のものを返すことができます:

-   キャンバス表示用のゲートウェイビューアURL
-   メッセージ配信用のレンダリング済みファイルパス（PNGまたはPDF）
-   両方の出力を1回の呼び出しで

有効にすると、プラグインは簡潔な使用ガイダンスをシステムプロンプト領域に前置し、また、エージェントがより完全な指示を必要とする場合に詳細なスキルを公開します。

## クイックスタート

1.  プラグインを有効化します。
2.  キャンバス優先フローの場合は `mode: "view"` で `diffs` を呼び出します。
3.  チャットファイル配信フローの場合は `mode: "file"` で `diffs` を呼び出します。
4.  両方のアーティファクトが必要な場合は `mode: "both"` で `diffs` を呼び出します。

## プラグインの有効化

```json
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
      },
    },
  },
}
```

## 組み込みシステムガイダンスの無効化

`diffs`ツールを有効にしたまま、組み込みのシステムプロンプトガイダンスを無効にしたい場合は、`plugins.entries.diffs.hooks.allowPromptInjection` を `false` に設定します:

```json
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
        hooks: {
          allowPromptInjection: false,
        },
      },
    },
  },
}
```

これにより、diffsプラグインの `before_prompt_build` フックがブロックされながら、プラグイン、ツール、コンパニオンスキルは利用可能なままになります。ガイダンスとツールの両方を無効にしたい場合は、代わりにプラグインを無効化してください。

## 典型的なエージェントワークフロー

1.  エージェントが `diffs` を呼び出します。
2.  エージェントが `details` フィールドを読み取ります。
3.  エージェントは以下のいずれかを行います:
    -   `details.viewerUrl` を `canvas present` で開く
    -   `details.filePath` を `path` または `filePath` を使用して `message` で送信する
    -   両方を行う

## 入力例

Before と after:

```json
{
  "before": "# Hello\n\nOne",
  "after": "# Hello\n\nTwo",
  "path": "docs/example.md",
  "mode": "view"
}
```

パッチ:

```json
{
  "patch": "diff --git a/src/example.ts b/src/example.ts\n--- a/src/example.ts\n+++ b/src/example.ts\n@@ -1 +1 @@\n-const x = 1;\n+const x = 2;\n",
  "mode": "both"
}
```

## ツール入力リファレンス

特に明記されていない限り、すべてのフィールドはオプションです:

-   `before` (`string`): 元のテキスト。`patch` が省略された場合、`after` とともに必須。
-   `after` (`string`): 更新後のテキスト。`patch` が省略された場合、`before` とともに必須。
-   `patch` (`string`): ユニファイド差分テキスト。`before` および `after` と排他的。
-   `path` (`string`): before/afterモードでの表示用ファイル名。
-   `lang` (`string`): before/afterモード用の言語オーバーライドヒント。
-   `title` (`string`): ビューアタイトルオーバーライド。
-   `mode` (`"view" | "file" | "both"`): 出力モード。デフォルトはプラグインのデフォルト `defaults.mode`。
-   `theme` (`"light" | "dark"`): ビューアテーマ。デフォルトはプラグインのデフォルト `defaults.theme`。
-   `layout` (`"unified" | "split"`): 差分レイアウト。デフォルトはプラグインのデフォルト `defaults.layout`。
-   `expandUnchanged` (`boolean`): 完全なコンテキストが利用可能な場合、変更されていないセクションを展開します。呼び出しごとのオプションのみ（プラグインデフォルトキーではありません）。
-   `fileFormat` (`"png" | "pdf"`): レンダリングファイル形式。デフォルトはプラグインのデフォルト `defaults.fileFormat`。
-   `fileQuality` (`"standard" | "hq" | "print"`): PNGまたはPDFレンダリング用の品質プリセット。
-   `fileScale` (`number`): デバイススケールオーバーライド (`1`\-`4`)。
-   `fileMaxWidth` (`number`): CSSピクセル単位の最大レンダリング幅 (`640`\-`2400`)。
-   `ttlSeconds` (`number`): ビューアアーティファクトのTTL（秒）。デフォルト1800、最大21600。
-   `baseUrl` (`string`): ビューアURLオリジンオーバーライド。`http` または `https` である必要があり、クエリ/ハッシュは不可。

検証と制限:

-   `before` と `after` はそれぞれ最大512 KiB。
-   `patch` は最大2 MiB。
-   `path` は最大2048バイト。
-   `lang` は最大128バイト。
-   `title` は最大1024バイト。
-   パッチ複雑度上限: 最大128ファイル、合計120000行。
-   `patch` と `before` または `after` の同時指定は拒否されます。
-   レンダリングファイルの安全制限（PNGおよびPDFに適用）:
    -   `fileQuality: "standard"`: 最大8 MP (8,000,000 レンダリングピクセル)。
    -   `fileQuality: "hq"`: 最大14 MP (14,000,000 レンダリングピクセル)。
    -   `fileQuality: "print"`: 最大24 MP (24,000,000 レンダリングピクセル)。
    -   PDFは最大50ページ。

## 出力詳細契約

ツールは `details` の下に構造化メタデータを返します。ビューアを作成するモードの共有フィールド:

-   `artifactId`
-   `viewerUrl`
-   `viewerPath`
-   `title`
-   `expiresAt`
-   `inputKind`
-   `fileCount`
-   `mode`

PNGまたはPDFがレンダリングされたときのファイルフィールド:

-   `filePath`
-   `path` (`filePath` と同じ値、メッセージツール互換性用)
-   `fileBytes`
-   `fileFormat`
-   `fileQuality`
-   `fileScale`
-   `fileMaxWidth`

モード動作の概要:

-   `mode: "view"`: ビューアフィールドのみ。
-   `mode: "file"`: ファイルフィールドのみ、ビューアアーティファクトなし。
-   `mode: "both"`: ビューアフィールドに加えてファイルフィールド。ファイルレンダリングが失敗した場合、ビューアは `fileError` とともに返されます。

## 折りたたまれた未変更セクション

-   ビューアは `N unmodified lines` のような行を表示できます。
-   それらの行の展開コントロールは条件付きであり、すべての入力種別で保証されるものではありません。
-   展開コントロールは、レンダリングされた差分に展開可能なコンテキストデータがある場合に表示されます。これはbefore/after入力では典型的です。
-   多くのユニファイドパッチ入力では、省略されたコンテキスト本文は解析されたパッチハンク内で利用できないため、行は展開コントロールなしで表示されることがあります。これは期待される動作です。
-   `expandUnchanged` は、展開可能なコンテキストが存在する場合にのみ適用されます。

## プラグインデフォルト

`~/.openclaw/openclaw.json` でプラグイン全体のデフォルトを設定します:

```json
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
        config: {
          defaults: {
            fontFamily: "Fira Code",
            fontSize: 15,
            lineSpacing: 1.6,
            layout: "unified",
            showLineNumbers: true,
            diffIndicators: "bars",
            wordWrap: true,
            background: true,
            theme: "dark",
            fileFormat: "png",
            fileQuality: "standard",
            fileScale: 2,
            fileMaxWidth: 960,
            mode: "both",
          },
        },
      },
    },
  },
}
```

サポートされるデフォルト:

-   `fontFamily`
-   `fontSize`
-   `lineSpacing`
-   `layout`
-   `showLineNumbers`
-   `diffIndicators`
-   `wordWrap`
-   `background`
-   `theme`
-   `fileFormat`
-   `fileQuality`
-   `fileScale`
-   `fileMaxWidth`
-   `mode`

明示的なツールパラメータはこれらのデフォルトを上書きします。

## セキュリティ設定

-   `security.allowRemoteViewer` (`boolean`, デフォルト `false`)
    -   `false`: ビューアルートへの非ループバックリクエストは拒否されます。
    -   `true`: トークン化されたパスが有効な場合、リモートビューアが許可されます。

例:

```json
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
        config: {
          security: {
            allowRemoteViewer: false,
          },
        },
      },
    },
  },
}
```

## アーティファクトのライフサイクルとストレージ

-   アーティファクトは一時サブフォルダ `$TMPDIR/openclaw-diffs` の下に保存されます。
-   ビューアアーティファクトメタデータには以下が含まれます:
    -   ランダムアーティファクトID (20桁の16進文字)
    -   ランダムトークン (48桁の16進文字)
    -   `createdAt` と `expiresAt`
    -   保存された `viewer.html` パス
-   指定がない場合のデフォルトのビューアTTLは30分です。
-   受け入れられる最大ビューアTTLは6時間です。
-   クリーンアップはアーティファクト作成後に機会的に実行されます。
-   期限切れのアーティファクトは削除されます。
-   フォールバッククリーンアップは、メタデータが欠落している場合、24時間以上経過した古いフォルダを削除します。

## ビューアURLとネットワーク動作

ビューアルート:

-   `/plugins/diffs/view/{artifactId}/{token}`

ビューアアセット:

-   `/plugins/diffs/assets/viewer.js`
-   `/plugins/diffs/assets/viewer-runtime.js`

URL構築動作:

-   `baseUrl` が提供された場合、厳密な検証後にそれが使用されます。
-   `baseUrl` がない場合、ビューアURLはデフォルトでループバック `127.0.0.1` になります。
-   ゲートウェイバインモードが `custom` で `gateway.customBindHost` が設定されている場合、そのホストが使用されます。

`baseUrl` ルール:

-   `http://` または `https://` である必要があります。
-   クエリとハッシュは拒否されます。
-   オリジンとオプションのベースパスは許可されます。

## セキュリティモデル

ビューアの強化:

-   デフォルトではループバックのみ。
-   厳密なIDとトークン検証を伴うトークン化ビューアパス。
-   ビューアレスポンスCSP:
    -   `default-src 'none'`
    -   スクリプトとアセットは自己からのみ
    -   アウトバウンド `connect-src` なし
-   リモートアクセスが有効な場合のリモートミススロットリング:
    -   60秒あたり40回の失敗
    -   60秒のロックアウト (`429 Too Many Requests`)

ファイルレンダリングの強化:

-   スクリーンショットブラウザリクエストルーティングはデフォルトで拒否。
-   `http://127.0.0.1/plugins/diffs/assets/*` からのローカルビューアアセットのみ許可。
-   外部ネットワークリクエストはブロックされます。

## ファイルモードのブラウザ要件

`mode: "file"` および `mode: "both"` にはChromium互換ブラウザが必要です。解決順序:

1.  OpenClaw設定内の `browser.executablePath`。
2.  環境変数:
    -   `OPENCLAW_BROWSER_EXECUTABLE_PATH`
    -   `BROWSER_EXECUTABLE_PATH`
    -   `PLAYWRIGHT_CHROMIUM_EXECUTABLE_PATH`
3.  プラットフォームコマンド/パス検出フォールバック。

一般的な失敗テキスト:

-   `Diff PNG/PDF rendering requires a Chromium-compatible browser...`

Chrome、Chromium、Edge、またはBraveをインストールするか、上記の実行可能パスオプションのいずれかを設定することで修正してください。

## トラブルシューティング

入力検証エラー:

-   `Provide patch or both before and after text.`
    -   `before` と `after` の両方を指定するか、`patch` を提供してください。
-   `Provide either patch or before/after input, not both.`
    -   入力モードを混在させないでください。
-   `Invalid baseUrl: ...`
    -   クエリ/ハッシュなしで、オプションのパスを持つ `http(s)` オリジンを使用してください。
-   `{field} exceeds maximum size (...)`
    -   ペイロードサイズを減らしてください。
-   大きなパッチの拒否
    -   パッチファイル数または総行数を減らしてください。

ビューアアクセシビリティの問題:

-   ビューアURLはデフォルトで `127.0.0.1` に解決されます。
-   リモートアクセスシナリオでは、以下のいずれかを行います:
    -   ツール呼び出しごとに `baseUrl` を渡す、または
    -   `gateway.bind=custom` と `gateway.customBindHost` を使用する
-   外部ビューアアクセスを意図する場合にのみ `security.allowRemoteViewer` を有効にしてください。

未変更行の行に展開ボタンがない:

-   これは、パッチが展開可能なコンテキストを持たないパッチ入力で発生する可能性があります。
-   これは期待される動作であり、ビューアの失敗を示すものではありません。

アーティファクトが見つからない:

-   アーティファクトがTTLにより期限切れ。
-   トークンまたはパスが変更された。
-   クリーンアップが古いデータを削除した。

## 運用ガイダンス

-   ローカルのインタラクティブレビューには `mode: "view"` を優先してください。
-   添付ファイルが必要なアウトバウンドチャットチャネルには `mode: "file"` を優先してください。
-   デプロイメントがリモートビューアURLを必要としない限り、`allowRemoteViewer` は無効のままにしてください。
-   機密性の高い差分には明示的に短い `ttlSeconds` を設定してください。
-   必要な場合を除き、差分入力にシークレットを送信しないでください。
-   チャネルが画像を積極的に圧縮する場合（例: TelegramやWhatsApp）、PDF出力 (`fileFormat: "pdf"`) を優先してください。

差分レンダリングエンジン:

## 関連ドキュメント

-   [ツール概要](../tools.md)
-   [プラグイン](./plugin.md)
-   [ブラウザ](./browser.md)

[Perplexity Sonar](../perplexity.md)[PDF Tool](./pdf.md)