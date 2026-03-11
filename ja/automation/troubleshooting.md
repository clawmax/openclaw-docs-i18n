

  自動化

  
# 自動化トラブルシューティング

このページは、スケジューラと配信の問題 (`cron` + `heartbeat`) に使用します。

## コマンドラダー

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

次に、自動化チェックを実行します:

```bash
openclaw cron status
openclaw cron list
openclaw system heartbeat last
```

## Cron が起動しない

```bash
openclaw cron status
openclaw cron list
openclaw cron runs --id <jobId> --limit 20
openclaw logs --follow
```

正常な出力は以下のようになります:

-   `cron status` が有効と将来の `nextWakeAtMs` を報告する。
-   ジョブが有効で、有効なスケジュール/タイムゾーンを持っている。
-   `cron runs` が `ok` または明示的なスキップ理由を表示する。

一般的な兆候:

-   `cron: scheduler disabled; jobs will not run automatically` → 設定/環境で cron が無効化されている。
-   `cron: timer tick failed` → スケジューラのティックがクラッシュした; 周囲のスタック/ログコンテキストを調査する。
-   実行出力内の `reason: not-due` → `--force` なしで手動実行が呼び出され、ジョブはまだ実行時刻でない。

## Cron は起動したが配信されない

```bash
openclaw cron runs --id <jobId> --limit 20
openclaw cron list
openclaw channels status --probe
openclaw logs --follow
```

正常な出力は以下のようになります:

-   実行ステータスが `ok` である。
-   分離ジョブに対して配信モード/ターゲットが設定されている。
-   チャネルプローブがターゲットチャネルの接続を報告する。

一般的な兆候:

-   実行は成功したが配信モードが `none` → 外部メッセージは期待されない。
-   配信ターゲットが欠落/無効 (`channel`/`to`) → 内部的には成功しても外部送信をスキップする可能性がある。
-   チャネル認証エラー (`unauthorized`, `missing_scope`, `Forbidden`) → チャネルの認証情報/権限により配信がブロックされる。

## Heartbeat が抑制またはスキップされた

```bash
openclaw system heartbeat last
openclaw logs --follow
openclaw config get agents.defaults.heartbeat
openclaw channels status --probe
```

正常な出力は以下のようになります:

-   ハートビートがゼロ以外の間隔で有効化されている。
-   最後のハートビート結果が `ran` (またはスキップ理由が理解できる)。

一般的な兆候:

-   `heartbeat skipped` で `reason=quiet-hours` → `activeHours` の範囲外。
-   `requests-in-flight` → メインレーンがビジー; ハートビートが延期された。
-   `empty-heartbeat-file` → `HEARTBEAT.md` に実行可能なコンテンツがなく、タグ付けされた cron イベントもキューされていないため、間隔ハートビートがスキップされた。
-   `alerts-disabled` → 可視性設定により外部ハートビートメッセージが抑制される。

## タイムゾーンと activeHours の落とし穴

```bash
openclaw config get agents.defaults.heartbeat.activeHours
openclaw config get agents.defaults.heartbeat.activeHours.timezone
openclaw config get agents.defaults.userTimezone || echo "agents.defaults.userTimezone not set"
openclaw cron list
openclaw logs --follow
```

簡単なルール:

-   `Config path not found: agents.defaults.userTimezone` はキーが設定されていないことを意味する; ハートビートはホストのタイムゾーン (または設定されていれば `activeHours.timezone`) にフォールバックする。
-   `--tz` なしの Cron はゲートウェイホストのタイムゾーンを使用する。
-   ハートビートの `activeHours` は設定されたタイムゾーン解決 (`user`, `local`, または明示的な IANA tz) を使用する。
-   タイムゾーンなしの ISO タイムスタンプは、cron の `at` スケジュールでは UTC として扱われる。

一般的な兆候:

-   ホストのタイムゾーン変更後、ジョブが間違った現地時刻に実行される。
-   `activeHours.timezone` が間違っているため、日中に常にハートビートがスキップされる。

関連項目:

-   [/automation/cron-jobs](./cron-jobs.md)
-   [/gateway/heartbeat](../gateway/heartbeat.md)
-   [/automation/cron-vs-heartbeat](./cron-vs-heartbeat.md)
-   [/concepts/timezone](../concepts/timezone.md)

[Cron vs Heartbeat](./cron-vs-heartbeat.md)[Webhooks](./webhook.md)