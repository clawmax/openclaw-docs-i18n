

  CLIコマンド

  
# cron

Gatewayスケジューラのcronジョブを管理します。関連項目:

-   Cronジョブ: [Cronジョブ](../automation/cron-jobs.md)

ヒント: 完全なコマンド体系は `openclaw cron --help` を実行してください。注: 分離型 `cron add` ジョブはデフォルトで `--announce` 配信になります。出力を内部に留めるには `--no-deliver` を使用してください。`--deliver` は `--announce` の非推奨エイリアスとして残っています。注: ワンショット (`--at`) ジョブは、デフォルトで成功後に削除されます。保持するには `--keep-after-run` を使用してください。注: 定期ジョブは、連続エラー後に指数関数的なリトライバックオフ (30s → 1m → 5m → 15m → 60m) を使用するようになり、次回の成功実行後に通常のスケジュールに戻ります。注: 保持/削除は設定で制御されます:

-   `cron.sessionRetention` (デフォルト `24h`) は完了した分離型実行セッションを削除します。
-   `cron.runLog.maxBytes` + `cron.runLog.keepLines` は `~/.openclaw/cron/runs/.jsonl` を削除します。

## 一般的な編集

メッセージを変更せずに配信設定を更新する:

```bash
openclaw cron edit <job-id> --announce --channel telegram --to "123456789"
```

分離ジョブの配信を無効にする:

```bash
openclaw cron edit <job-id> --no-deliver
```

分離ジョブに軽量ブートストラップコンテキストを有効にする:

```bash
openclaw cron edit <job-id> --light-context
```

特定のチャンネルに通知する:

```bash
openclaw cron edit <job-id> --announce --channel slack --to "channel:C1234567890"
```

軽量ブートストラップコンテキストを持つ分離ジョブを作成する:

```bash
openclaw cron add \
  --name "軽量朝のブリーフ" \
  --cron "0 7 * * *" \
  --session isolated \
  --message "夜間の更新を要約してください。" \
  --light-context \
  --no-deliver
```

`--light-context` は分離型エージェントターンジョブにのみ適用されます。cron実行の場合、軽量モードは完全なワークスペースブートストラップセットを注入する代わりに、ブートストラップコンテキストを空のまま保持します。

[configure](./configure.md)[daemon](./daemon.md)