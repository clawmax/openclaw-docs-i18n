

  CLIコマンド

  
# health

実行中のゲートウェイからヘルス情報を取得します。

```bash
openclaw health
openclaw health --json
openclaw health --verbose
```

注記:

-   `--verbose` はライブプローブを実行し、複数のアカウントが設定されている場合はアカウントごとのタイミングを表示します。
-   出力には、複数のエージェントが設定されている場合はエージェントごとのセッションストアが含まれます。

[gateway](./gateway.md)[hooks](./hooks.md)

---