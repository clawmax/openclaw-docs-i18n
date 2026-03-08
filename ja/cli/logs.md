

  CLI コマンド

  
# logs

RPC経由でゲートウェイのファイルログを tail します（リモートモードで動作）。関連項目:

-   ロギングの概要: [ロギング](../logging.md)

## 例

```bash
openclaw logs
openclaw logs --follow
openclaw logs --json
openclaw logs --limit 500
openclaw logs --local-time
openclaw logs --follow --local-time
```

`--local-time` を使用すると、タイムスタンプがローカルタイムゾーンで表示されます。

[hooks](./hooks.md)[memory](./memory.md)

---