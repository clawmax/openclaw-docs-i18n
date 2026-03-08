title: "OpenClaw Canvas macOS アプリ ドキュメントと使用ガイド"
description: "macOS 上の OpenClaw Canvas パネルを HTML/CSS/JS ワークスペースや A2UI で使用する方法を学びます。ファイルストレージ、エージェント API、CLI コマンド、セキュリティについて説明します。"
keywords: ["openclaw canvas", "macos canvas パネル", "a2ui 統合", "エージェント api", "wkwebview", "canvas ワークスペース", "openclaw ドキュメント", "canvas cli コマンド"]
---

  macOS コンパニオンアプリ

  
# Canvas

macOS アプリは、`WKWebView` を使用したエージェント制御の **Canvas パネル** を埋め込みます。これは HTML/CSS/JS、A2UI、および小さなインタラクティブ UI サーフェスのための軽量なビジュアルワークスペースです。

## Canvas の保存場所

Canvas の状態は Application Support の下に保存されます:

-   `~/Library/Application Support/OpenClaw/canvas//...`

Canvas パネルはこれらのファイルを **カスタム URL スキーム** で提供します:

-   `openclaw-canvas:///`

例:

-   `openclaw-canvas://main/` → `/main/index.html`
-   `openclaw-canvas://main/assets/app.css` → `/main/assets/app.css`
-   `openclaw-canvas://main/widgets/todo/` → `/main/widgets/todo/index.html`

ルートに `index.html` が存在しない場合、アプリは **組み込みの足場ページ** を表示します。

## パネルの動作

-   境界線なし、サイズ変更可能なパネルで、メニューバー（またはマウスカーソル）の近くに固定されます。
-   セッションごとにサイズ/位置を記憶します。
-   ローカルの Canvas ファイルが変更されると自動的にリロードします。
-   一度に表示される Canvas パネルは 1 つだけです（必要に応じてセッションが切り替わります）。

Canvas は設定 → **Canvas を許可** から無効にできます。無効にすると、canvas ノードコマンドは `CANVAS_DISABLED` を返します。

## エージェント API サーフェス

Canvas は **Gateway WebSocket** を介して公開されるため、エージェントは以下を行うことができます:

-   パネルの表示/非表示
-   パスまたは URL へのナビゲート
-   JavaScript の評価
-   スナップショット画像のキャプチャ

CLI の例:

```bash
openclaw nodes canvas present --node <id>
openclaw nodes canvas navigate --node <id> --url "/"
openclaw nodes canvas eval --node <id> --js "document.title"
openclaw nodes canvas snapshot --node <id>
```

注意:

-   `canvas.navigate` は **ローカル Canvas パス**、`http(s)` URL、および `file://` URL を受け入れます。
-   `"/"` を渡すと、Canvas はローカルの足場ページまたは `index.html` を表示します。

## Canvas 内の A2UI

A2UI は Gateway の canvas ホストによってホストされ、Canvas パネル内でレンダリングされます。Gateway が Canvas ホストをアドバタイズすると、macOS アプリは初回起動時に A2UI ホストページに自動的にナビゲートします。デフォルトの A2UI ホスト URL:

```
http://<gateway-host>:18789/__openclaw__/a2ui/
```

### A2UI コマンド (v0.8)

Canvas は現在、**A2UI v0.8** のサーバー→クライアントメッセージを受け付けます:

-   `beginRendering`
-   `surfaceUpdate`
-   `dataModelUpdate`
-   `deleteSurface`

`createSurface` (v0.9) はサポートされていません。CLI の例:

```bash
cat > /tmp/a2ui-v0.8.jsonl <<'EOFA2'
{"surfaceUpdate":{"surfaceId":"main","components":[{"id":"root","component":{"Column":{"children":{"explicitList":["title","content"]}}}},{"id":"title","component":{"Text":{"text":{"literalString":"Canvas (A2UI v0.8)"},"usageHint":"h1"}}},{"id":"content","component":{"Text":{"text":{"literalString":"If you can read this, A2UI push works."},"usageHint":"body"}}}]}}
{"beginRendering":{"surfaceId":"main","root":"root"}}
EOFA2

openclaw nodes canvas a2ui push --jsonl /tmp/a2ui-v0.8.jsonl --node <id>
```

簡単な動作確認:

```bash
openclaw nodes canvas a2ui push --node <id> --text "Hello from A2UI"
```

## Canvas からのエージェント実行のトリガー

Canvas はディープリンクを介して新しいエージェント実行をトリガーできます:

-   `openclaw://agent?...`

例 (JS 内):

```bash
window.location.href = "openclaw://agent?message=Review%20this%20design";
```

有効なキーが提供されない限り、アプリは確認を求めます。

## セキュリティに関する注意

-   Canvas スキームはディレクトリトラバーサルをブロックします。ファイルはセッションルートの下に存在する必要があります。
-   ローカル Canvas コンテンツはカスタムスキームを使用します（ループバックサーバーは不要です）。
-   外部の `http(s)` URL は、明示的にナビゲートされた場合にのみ許可されます。

[WebChat](./webchat.md)[Gateway ライフサイクル](./child-process.md)

---