

  CLI コマンド

  
# system

ゲートウェイのシステムレベルヘルパー: システムイベントのエンキュー、ハートビートの制御、プレゼンスの表示。

## 一般的なコマンド

```bash
openclaw system event --text "Check for urgent follow-ups" --mode now
openclaw system heartbeat enable
openclaw system heartbeat last
openclaw system presence
```

## system event

**メイン**セッションにシステムイベントをエンキューします。次のハートビートで、プロンプト内の`System:`行として注入されます。`--mode now`を使用してハートビートを即時トリガーします；`next-heartbeat`は次のスケジュールされたタイクルを待機します。フラグ:

-   `--text `: 必須のシステムイベントテキスト。
-   `--mode `: `now` または `next-heartbeat` (デフォルト)。
-   `--json`: 機械可読な出力。

## system heartbeat last|enable|disable

ハートビート制御:

-   `last`: 最後のハートビートイベントを表示します。
-   `enable`: ハートビートを再有効化します (無効化されていた場合に使用)。
-   `disable`: ハートビートを一時停止します。

フラグ:

-   `--json`: 機械可読な出力。

## system presence

ゲートウェイが認識している現在のシステムプレゼンスエントリ (ノード、インスタンス、および同様のステータス行) を一覧表示します。フラグ:

-   `--json`: 機械可読な出力。

## 注意事項

-   現在の設定 (ローカルまたはリモート) で到達可能な実行中のゲートウェイが必要です。
-   システムイベントは一時的なものであり、再起動後も保持されません。

[status](./status.md)[tui](./tui.md)

---