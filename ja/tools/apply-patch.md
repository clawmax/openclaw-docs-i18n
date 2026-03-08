title: "OpenClaw AIにおけるマルチファイル編集のためのApply Patchツール"
description: "apply_patchツールを使用して、構造化されたマルチファイルのコード変更を安全に行う方法を学びます。パッチ形式を使用した複雑な編集に最適です。"
keywords: ["apply_patch", "パッチツール", "マルチファイル編集", "構造化パッチ", "openclawツール", "ファイル操作", "コード編集", "ワークスペース管理"]
---

  組み込みツール

  
# apply_patch ツール

構造化されたパッチ形式を使用してファイル変更を適用します。これは、単一の`edit`呼び出しが脆弱になる可能性のある、マルチファイルまたはマルチハンクの編集に最適です。このツールは、1つ以上のファイル操作をラップする単一の`input`文字列を受け入れます：

```markdown
*** Begin Patch
*** Add File: path/to/file.txt
+line 1
+line 2
*** Update File: src/app.ts
@@
-old line
+new line
*** Delete File: obsolete.txt
*** End Patch
```

## パラメータ

-   `input` (必須): `*** Begin Patch`と`*** End Patch`を含む完全なパッチ内容。

## 注意事項

-   パッチのパスは、相対パス（ワークスペースディレクトリからの）と絶対パスをサポートしています。
-   `tools.exec.applyPatch.workspaceOnly`はデフォルトで`true`（ワークスペース内のみ）です。`apply_patch`にワークスペースディレクトリ外への書き込み/削除を意図的に行わせたい場合にのみ`false`に設定してください。
-   ファイルの名前を変更するには、`*** Update File:`ハンク内で`*** Move to:`を使用します。
-   `*** End of File`は、必要な場合にEOFのみへの挿入を示します。
-   実験的機能であり、デフォルトでは無効です。`tools.exec.applyPatch.enabled`で有効にしてください。
-   OpenAI専用（OpenAI Codexを含む）。オプションで`tools.exec.applyPatch.allowModels`を介してモデルごとに制限できます。
-   設定は`tools.exec`の下にのみあります。

## 例

```json
{
  "tool": "apply_patch",
  "input": "*** Begin Patch\n*** Update File: src/index.ts\n@@\n-const foo = 1\n+const foo = 2\n*** End Patch"
}
```

[ツール](../tools.md)[Brave検索](../brave-search.md)