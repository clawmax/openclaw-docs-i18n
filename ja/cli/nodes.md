

  CLIコマンド

  
# nodes

ペアリング済みノード（デバイス）を管理し、ノードの機能を呼び出します。関連情報:

-   ノード概要: [ノード](../nodes.md)
-   カメラ: [カメラノード](../nodes/camera.md)
-   画像: [画像ノード](../nodes/images.md)

共通オプション:

-   `--url`, `--token`, `--timeout`, `--json`

## よく使うコマンド

```bash
openclaw nodes list
openclaw nodes list --connected
openclaw nodes list --last-connected 24h
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes status
openclaw nodes status --connected
openclaw nodes status --last-connected 24h
```

`nodes list` は、保留中/ペアリング済みのテーブルを表示します。ペアリング済みの行には、直近の接続からの経過時間（最終接続）が含まれます。`--connected` を使用すると、現在接続中のノードのみを表示します。`--last-connected <期間>` を使用すると、指定期間内（例: `24h`, `7d`）に接続したノードに絞り込めます。

## 呼び出し / 実行

```bash
openclaw nodes invoke --node <id|name|ip> --command <command> --params <json>
openclaw nodes run --node <id|name|ip> <command...>
openclaw nodes run --raw "git status"
openclaw nodes run --agent main --node <id|name|ip> --raw "git status"
```

呼び出しフラグ:

-   `--params `: JSONオブジェクト文字列（デフォルト `{}`）。
-   `--invoke-timeout `: ノード呼び出しのタイムアウト（デフォルト `15000`）。
-   `--idempotency-key `: オプションの冪等性キー。

### 実行形式のデフォルト

`nodes run` は、モデルの実行動作（デフォルト + 承認）を模倣します:

-   `tools.exec.*`（および `agents.list[].tools.exec.*` のオーバーライド）を読み取ります。
-   `system.run` を呼び出す前に、実行承認（`exec.approval.request`）を使用します。
-   `tools.exec.node` が設定されている場合、`--node` は省略できます。
-   `system.run` をアドバタイズするノードが必要です（macOSコンパニオンアプリまたはヘッドレスノードホスト）。

フラグ:

-   `--cwd `: 作業ディレクトリ。
-   `--env <key=val>`: 環境変数のオーバーライド（繰り返し可能）。注: ノードホストは `PATH` のオーバーライドを無視します（また、`tools.exec.pathPrepend` はノードホストには適用されません）。
-   `--command-timeout `: コマンドのタイムアウト。
-   `--invoke-timeout `: ノード呼び出しのタイムアウト（デフォルト `30000`）。
-   `--needs-screen-recording`: 画面録画の許可を必須とします。
-   `--raw `: シェル文字列を実行します（`/bin/sh -lc` または `cmd.exe /c`）。Windowsノードホストの許可リストモードでは、`cmd.exe /c` シェルラッパーの実行には承認が必要です（許可リストのエントリだけでは、ラッパー形式は自動的に許可されません）。
-   `--agent `: エージェントスコープの承認/許可リスト（デフォルトは設定されたエージェント）。
-   `--ask <off|on-miss|always>`, `--security <deny|allowlist|full>`: オーバーライド。

[node](./node.md)[onboard](./onboard.md)

---