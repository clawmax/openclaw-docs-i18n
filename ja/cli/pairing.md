

  CLIコマンド

  
# pairing

DMペアリングリクエストを承認または検査します（ペアリングをサポートするチャネル向け）。関連項目:

-   ペアリングフロー: [ペアリング](../channels/pairing.md)

## コマンド

```bash
openclaw pairing list telegram
openclaw pairing list --channel telegram --account work
openclaw pairing list telegram --json

openclaw pairing approve telegram <code>
openclaw pairing approve --channel telegram --account work <code> --notify
```

## 注記

-   チャネル入力: 位置引数として渡す (`pairing list telegram`) か、`--channel ` を使用します。
-   `pairing list` は、マルチアカウントチャネル向けに `--account ` をサポートします。
-   `pairing approve` は `--account ` と `--notify` をサポートします。
-   ペアリング可能なチャネルが1つだけ設定されている場合、`pairing approve ` が許可されます。

[onboard](./onboard.md)[plugins](./plugins.md)

---