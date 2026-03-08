

  CLI コマンド

  
# sessions

保存された会話セッションを一覧表示します。

```bash
openclaw sessions
openclaw sessions --agent work
openclaw sessions --all-agents
openclaw sessions --active 120
openclaw sessions --json
```

スコープ選択:

-   default: 設定されたデフォルトエージェントストア
-   `--agent `: 設定された1つのエージェントストア
-   `--all-agents`: 設定されたすべてのエージェントストアを集約
-   `--store `: 明示的なストアパス (`--agent` または `--all-agents` と組み合わせ不可)

JSON の例: `openclaw sessions --all-agents --json`:

```json
{
  "path": null,
  "stores": [
    { "agentId": "main", "path": "/home/user/.openclaw/agents/main/sessions/sessions.json" },
    { "agentId": "work", "path": "/home/user/.openclaw/agents/work/sessions/sessions.json" }
  ],
  "allAgents": true,
  "count": 2,
  "activeMinutes": null,
  "sessions": [
    { "agentId": "main", "key": "agent:main:main", "model": "gpt-5" },
    { "agentId": "work", "key": "agent:work:main", "model": "claude-opus-4-5" }
  ]
}
```

## クリーンアップメンテナンス

メンテナンスを今すぐ実行します（次の書き込みサイクルを待たずに）:

```bash
openclaw sessions cleanup --dry-run
openclaw sessions cleanup --agent work --dry-run
openclaw sessions cleanup --all-agents --dry-run
openclaw sessions cleanup --enforce
openclaw sessions cleanup --enforce --active-key "agent:main:telegram:dm:123"
openclaw sessions cleanup --json
```

`openclaw sessions cleanup` は設定の `session.maintenance` 設定を使用します:

-   スコープ注意: `openclaw sessions cleanup` はセッションストア/トランスクリプトのみをメンテナンスします。cron実行ログ (`cron/runs/.jsonl`) の削除は行いません。これらは [Cron設定](../automation/cron-jobs.md#configuration) の `cron.runLog.maxBytes` および `cron.runLog.keepLines` で管理され、[Cronメンテナンス](../automation/cron-jobs.md#maintenance) で説明されています。
-   `--dry-run`: 書き込みを行わずに、何件のエントリが削除/制限されるかをプレビューします。
    -   テキストモードでは、dry-runはセッションごとのアクションテーブル (`Action`, `Key`, `Age`, `Model`, `Flags`) を出力し、保持されるものと削除されるものを確認できます。
-   `--enforce`: `session.maintenance.mode` が `warn` の場合でもメンテナンスを適用します。
-   `--active-key `: 特定のアクティブなキーをディスク予算による削除から保護します。
-   `--agent `: 設定された1つのエージェントストアに対してクリーンアップを実行します。
-   `--all-agents`: 設定されたすべてのエージェントストアに対してクリーンアップを実行します。
-   `--store `: 特定の `sessions.json` ファイルに対して実行します。
-   `--json`: JSONサマリーを出力します。`--all-agents` と組み合わせると、出力にはストアごとのサマリーが含まれます。

`openclaw sessions cleanup --all-agents --dry-run --json`:

```json
{
  "allAgents": true,
  "mode": "warn",
  "dryRun": true,
  "stores": [
    {
      "agentId": "main",
      "storePath": "/home/user/.openclaw/agents/main/sessions/sessions.json",
      "beforeCount": 120,
      "afterCount": 80,
      "pruned": 40,
      "capped": 0
    },
    {
      "agentId": "work",
      "storePath": "/home/user/.openclaw/agents/work/sessions/sessions.json",
      "beforeCount": 18,
      "afterCount": 18,
      "pruned": 0,
      "capped": 0
    }
  ]
}
```

関連項目:

-   セッション設定: [設定リファレンス](../gateway/configuration-reference.md#session)

[security](./security.md)[setup](./setup.md)