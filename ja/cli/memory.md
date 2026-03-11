

  CLIコマンド

  
# memory

セマンティックメモリのインデックス作成と検索を管理します。アクティブなメモリプラグインによって提供されます（デフォルト: `memory-core`; 無効にするには `plugins.slots.memory = "none"` を設定）。関連項目:

-   メモリの概念: [Memory](../concepts/memory.md)
-   プラグイン: [Plugins](../tools/plugin.md)

## 例

```bash
openclaw memory status
openclaw memory status --deep
openclaw memory index --force
openclaw memory search "meeting notes"
openclaw memory search --query "deployment" --max-results 20
openclaw memory status --json
openclaw memory status --deep --index
openclaw memory status --deep --index --verbose
openclaw memory status --agent main
openclaw memory index --agent main --verbose
```

## オプション

`memory status` と `memory index`:

-   `--agent `: 単一のエージェントにスコープを限定します。これを指定しない場合、これらのコマンドは設定された各エージェントに対して実行されます。エージェントリストが設定されていない場合は、デフォルトエージェントにフォールバックします。
-   `--verbose`: プローブとインデックス作成中に詳細なログを出力します。

`memory status`:

-   `--deep`: ベクトル + 埋め込みの可用性をプローブします。
-   `--index`: ストアがダーティな場合に再インデックスを実行します（`--deep` を暗黙的に含みます）。
-   `--json`: JSON出力を表示します。

`memory index`:

-   `--force`: 完全な再インデックスを強制実行します。

`memory search`:

-   クエリ入力: 位置引数 `[query]` または `--query ` のいずれかを渡します。
-   両方が提供された場合、`--query` が優先されます。
-   どちらも提供されなかった場合、コマンドはエラーで終了します。
-   `--agent `: 単一のエージェントにスコープを限定します（デフォルト: デフォルトエージェント）。
-   `--max-results `: 返される結果の数を制限します。
-   `--min-score `: 低スコアの一致を除外します。
-   `--json`: JSON結果を表示します。

注意:

-   `memory index --verbose` はフェーズごとの詳細（プロバイダー、モデル、ソース、バッチアクティビティ）を出力します。
-   `memory status` には、`memorySearch.extraPaths` を介して設定された追加パスが含まれます。
-   実質的にアクティブなメモリリモートAPIキーフィールドがSecretRefとして設定されている場合、コマンドはアクティブなゲートウェイスナップショットからそれらの値を解決します。ゲートウェイが利用できない場合、コマンドは速やかに失敗します。
-   ゲートウェイバージョンスキューの注意: このコマンドパスは `secrets.resolve` をサポートするゲートウェイを必要とします。古いゲートウェイは不明なメソッドエラーを返します。

[logs](./logs.md)[message](./message.md)

---