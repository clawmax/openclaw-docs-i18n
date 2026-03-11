

  環境とデバッグ

  
# 診断フラグ

診断フラグを使用すると、どこでも冗長なログ記録を有効にすることなく、ターゲットを絞ったデバッグログを有効にできます。フラグはオプトイン方式であり、サブシステムがチェックしない限り効果はありません。

## 仕組み

-   フラグは文字列です（大文字小文字を区別しません）。
-   設定ファイルまたは環境変数による上書きでフラグを有効にできます。
-   ワイルドカードがサポートされています：
    -   `telegram.*` は `telegram.http` に一致します
    -   `*` はすべてのフラグを有効にします

## 設定ファイルで有効にする

```json
{
  "diagnostics": {
    "flags": ["telegram.http"]
  }
}
```

複数のフラグ：

```json
{
  "diagnostics": {
    "flags": ["telegram.http", "gateway.*"]
  }
}
```

フラグを変更した後はゲートウェイを再起動してください。

## 環境変数による上書き（ワンオフ）

```
OPENCLAW_DIAGNOSTICS=telegram.http,telegram.payload
```

すべてのフラグを無効にする：

```
OPENCLAW_DIAGNOSTICS=0
```

## ログの出力先

フラグはログを標準の診断ログファイルに出力します。デフォルト：

```
/tmp/openclaw/openclaw-YYYY-MM-DD.log
```

`logging.file` を設定した場合は、そのパスが使用されます。ログは JSONL 形式（1行に1つの JSON オブジェクト）です。`logging.redactSensitive` に基づく秘匿化は引き続き適用されます。

## ログを抽出する

最新のログファイルを選択：

```bash
ls -t /tmp/openclaw/openclaw-*.log | head -n 1
```

Telegram HTTP 診断ログをフィルタリング：

```bash
rg "telegram http error" /tmp/openclaw/openclaw-*.log
```

または、問題を再現しながら tail する：

```bash
tail -f /tmp/openclaw/openclaw-$(date +%F).log | rg "telegram http error"
```

リモートゲートウェイの場合は、`openclaw logs --follow` も使用できます（[/cli/logs](../cli/logs.md) を参照）。

## 注意事項

-   `logging.level` が `warn` より高い値に設定されている場合、これらのログは抑制される可能性があります。デフォルトの `info` で問題ありません。
-   フラグは有効にしたままにしても安全です。特定のサブシステムのログ量にのみ影響します。
-   ログの出力先、レベル、秘匿化を変更するには、[/logging](../logging.md) を使用してください。

[Node + tsx クラッシュ](../debug/node-issue.md)[Node.js](../install/node.md)