

  設定と操作

  
# バックグラウンドExecとプロセスツール

OpenClawはシェルコマンドを`exec`ツールを通じて実行し、長時間実行されるタスクをメモリ内に保持します。`process`ツールはそれらのバックグラウンドセッションを管理します。

## execツール

主なパラメータ:

-   `command` (必須)
-   `yieldMs` (デフォルト 10000): この遅延後に自動的にバックグラウンド化
-   `background` (bool): 即時バックグラウンド化
-   `timeout` (秒, デフォルト 1800): このタイムアウト後にプロセスを強制終了
-   `elevated` (bool): 昇格モードが有効/許可されている場合、ホスト上で実行
-   本物のTTYが必要ですか？ `pty: true`を設定します。
-   `workdir`, `env`

動作:

-   フォアグラウンド実行は出力を直接返します。
-   バックグラウンド化された場合（明示的またはタイムアウト）、ツールは`status: "running"` + `sessionId`と短い末尾出力を返します。
-   出力は、セッションがポーリングされるかクリアされるまでメモリ内に保持されます。
-   `process`ツールが許可されていない場合、`exec`は同期的に実行され、`yieldMs`/`background`は無視されます。
-   生成されたexecコマンドは、コンテキストを認識するシェル/プロファイルルールのために`OPENCLAW_SHELL=exec`を受け取ります。

## 子プロセスブリッジング

exec/processツールの外部で長時間実行される子プロセスを生成する場合（例えば、CLIの再起動やゲートウェイヘルパー）、子プロセスブリッジヘルパーをアタッチして、終了シグナルが転送され、終了/エラー時にリスナーがデタッチされるようにします。これにより、systemd上の孤立プロセスを回避し、プラットフォーム間で一貫したシャットダウン動作を維持できます。環境変数オーバーライド:

-   `PI_BASH_YIELD_MS`: デフォルトのyield (ms)
-   `PI_BASH_MAX_OUTPUT_CHARS`: メモリ内出力上限 (文字数)
-   `OPENCLAW_BASH_PENDING_MAX_OUTPUT_CHARS`: ストリームごとの保留中stdout/stderr上限 (文字数)
-   `PI_BASH_JOB_TTL_MS`: 完了したセッションのTTL (ms, 1分〜3時間に制限)

設定 (推奨):

-   `tools.exec.backgroundMs` (デフォルト 10000)
-   `tools.exec.timeoutSec` (デフォルト 1800)
-   `tools.exec.cleanupMs` (デフォルト 1800000)
-   `tools.exec.notifyOnExit` (デフォルト true): バックグラウンド化されたexecが終了したときにシステムイベントをエンキューし、ハートビートを要求します。
-   `tools.exec.notifyOnExitEmptySuccess` (デフォルト false): trueの場合、出力を生成しなかった成功したバックグラウンド実行に対しても完了イベントをエンキューします。

## processツール

アクション:

-   `list`: 実行中 + 完了したセッションの一覧
-   `poll`: セッションの新しい出力を排出 (終了ステータスも報告)
-   `log`: 集約された出力を読み取り (`offset` + `limit`をサポート)
-   `write`: stdinを送信 (`data`, オプションで`eof`)
-   `kill`: バックグラウンドセッションを終了
-   `clear`: 完了したセッションをメモリから削除
-   `remove`: 実行中の場合は終了、それ以外の場合は完了済みならクリア

注意点:

-   バックグラウンド化されたセッションのみが一覧表示され、メモリ内に保持されます。
-   セッションはプロセス再起動時に失われます（ディスクへの永続化はありません）。
-   セッションログは、`process poll/log`を実行し、ツールの結果が記録された場合にのみチャット履歴に保存されます。
-   `process`はエージェントごとにスコープされます。そのエージェントによって開始されたセッションのみを認識します。
-   `process list`には、クイックスキャンのための派生`name`（コマンド動詞 + ターゲット）が含まれます。
-   `process log`は行ベースの`offset`/`limit`を使用します。
-   `offset`と`limit`の両方が省略された場合、最後の200行を返し、ページングのヒントを含みます。
-   `offset`が提供され、`limit`が省略された場合、`offset`から末尾までを返します（200行に制限されません）。

## 例

長時間タスクを実行し、後でポーリング:

```json
{ "tool": "exec", "command": "sleep 5 && echo done", "yieldMs": 1000 }
```

```json
{ "tool": "process", "action": "poll", "sessionId": "<id>" }
```

即時にバックグラウンドで開始:

```json
{ "tool": "exec", "command": "npm run build", "background": true }
```

stdinを送信:

```json
{ "tool": "process", "action": "write", "sessionId": "<id>", "data": "y\n" }
```

[ゲートウェイロック](./gateway-lock.md)[複数ゲートウェイ](./multiple-gateways.md)