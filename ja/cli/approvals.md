

  CLIコマンド

  
# approvals

**ローカルホスト**、**ゲートウェイホスト**、または**ノードホスト**の実行承認を管理します。デフォルトでは、コマンドはディスク上のローカルの承認ファイルを対象とします。ゲートウェイを対象にするには `--gateway` を、特定のノードを対象にするには `--node` を使用します。関連項目:

-   実行承認: [実行承認](../tools/exec-approvals.md)
-   ノード: [ノード](../nodes.md)

## 一般的なコマンド

```bash
openclaw approvals get
openclaw approvals get --node <id|name|ip>
openclaw approvals get --gateway
```

## ファイルから承認を置き換える

```bash
openclaw approvals set --file ./exec-approvals.json
openclaw approvals set --node <id|name|ip> --file ./exec-approvals.json
openclaw approvals set --gateway --file ./exec-approvals.json
```

## 許可リストヘルパー

```bash
openclaw approvals allowlist add "~/Projects/**/bin/rg"
openclaw approvals allowlist add --agent main --node <id|name|ip> "/usr/bin/uptime"
openclaw approvals allowlist add --agent "*" "/usr/bin/uname"

openclaw approvals allowlist remove "~/Projects/**/bin/rg"
```

## 注意事項

-   `--node` は `openclaw nodes` と同じリゾルバーを使用します（ID、名前、IP、またはIDプレフィックス）。
-   `--agent` のデフォルトは `"*"` で、すべてのエージェントに適用されます。
-   ノードホストは `system.execApprovals.get/set` をアドバタイズしている必要があります（macOSアプリまたはヘッドレスノードホスト）。
-   承認ファイルはホストごとに `~/.openclaw/exec-approvals.json` に保存されます。

[agents](./agents.md)[browser](./browser.md)

---