

  エージェント連携

  
# Agent Send

`openclaw agent` は、受信チャットメッセージを必要とせずに、単一のエージェントターンを実行します。デフォルトでは **Gateway を経由** します。現在のマシン上の組み込みランタイムを強制するには `--local` を追加してください。

## 動作

-   必須: `--message `
-   セッション選択:
    -   `--to ` はセッションキーを導出します（グループ/チャネルターゲットは分離を維持し、ダイレクトチャットは `main` に統合されます）、**または**
    -   `--session-id ` は既存のセッションをIDで再利用します、**または**
    -   `--agent ` は設定済みのエージェントを直接ターゲットします（そのエージェントの `main` セッションキーを使用）
-   通常の受信返信と同じ組み込みエージェントランタイムを実行します。
-   思考/詳細フラグはセッションストアに保持されます。
-   出力:
    -   デフォルト: 返信テキスト（および `MEDIA:` 行）を表示
    -   `--json`: 構造化されたペイロードとメタデータを表示
-   `--deliver` + `--channel` を使用して、オプションでチャネルに配信（ターゲット形式は `openclaw message --target` と一致）。
-   セッションを変更せずに配信を上書きするには `--reply-channel`/`--reply-to`/`--reply-account` を使用します。

Gateway に到達できない場合、CLI は **組み込みのローカル実行にフォールバック** します。

## 例

```bash
openclaw agent --to +15555550123 --message "status update"
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --to +15555550123 --message "Trace logs" --verbose on --json
openclaw agent --to +15555550123 --message "Summon reply" --deliver
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```

## フラグ

-   `--local`: ローカルで実行（シェルにモデルプロバイダーのAPIキーが必要）
-   `--deliver`: 返信を選択したチャネルに送信
-   `--channel`: 配信チャネル (`whatsapp|telegram|discord|googlechat|slack|signal|imessage`, デフォルト: `whatsapp`)
-   `--reply-to`: 配信ターゲットの上書き
-   `--reply-channel`: 配信チャネルの上書き
-   `--reply-account`: 配信アカウントIDの上書き
-   `--thinking <off|minimal|low|medium|high|xhigh>`: 思考レベルを保持（GPT-5.2 + Codex モデルのみ）
-   `--verbose <on|full|off>`: 詳細レベルを保持
-   `--timeout `: エージェントタイムアウトの上書き
-   `--json`: 構造化されたJSONを出力

[ブラウザのトラブルシューティング](./browser-linux-troubleshooting.md)[サブエージェント](./subagents.md)

---