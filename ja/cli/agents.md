

  CLI コマンド

  
# agents

分離エージェント（ワークスペース + 認証 + ルーティング）を管理します。関連項目:

-   マルチエージェントルーティング: [マルチエージェントルーティング](../concepts/multi-agent.md)
-   エージェントワークスペース: [エージェントワークスペース](../concepts/agent-workspace.md)

## 例

```bash
openclaw agents list
openclaw agents add work --workspace ~/.openclaw/workspace-work
openclaw agents bindings
openclaw agents bind --agent work --bind telegram:ops
openclaw agents unbind --agent work --bind telegram:ops
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
openclaw agents set-identity --agent main --avatar avatars/openclaw.png
openclaw agents delete work
```

## ルーティングバインディング

ルーティングバインディングを使用して、インバウンドチャネルトラフィックを特定のエージェントに固定します。バインディングを一覧表示:

```bash
openclaw agents bindings
openclaw agents bindings --agent work
openclaw agents bindings --json
```

バインディングを追加:

```bash
openclaw agents bind --agent work --bind telegram:ops --bind discord:guild-a
```

`accountId` を省略した場合 (`--bind `)、OpenClaw はチャネルのデフォルトとプラグインのセットアップフックから可能な限り解決します。

### バインディングスコープの動作

-   `accountId` のないバインディングは、チャネルのデフォルトアカウントのみに一致します。
-   `accountId: "*"` はチャネル全体のフォールバック（すべてのアカウント）であり、明示的なアカウントバインディングよりも優先度が低くなります。
-   同じエージェントにすでに `accountId` のない一致するチャネルバインディングがあり、後で明示的または解決された `accountId` でバインドした場合、OpenClaw は重複を追加する代わりに、既存のバインディングをその場でアップグレードします。

例:

```bash
# 初期のチャネルのみのバインディング
openclaw agents bind --agent work --bind telegram

# 後でアカウントスコープのバインディングにアップグレード
openclaw agents bind --agent work --bind telegram:ops
```

アップグレード後、そのバインディングのルーティングは `telegram:ops` にスコープされます。デフォルトアカウントのルーティングも必要な場合は、明示的に追加してください（例: `--bind telegram:default`）。バインディングを削除:

```bash
openclaw agents unbind --agent work --bind telegram:ops
openclaw agents unbind --agent work --all
```

## IDファイル

各エージェントワークスペースは、ワークスペースルートに `IDENTITY.md` を含むことができます:

-   例のパス: `~/.openclaw/workspace/IDENTITY.md`
-   `set-identity --from-identity` はワークスペースルート（または明示的な `--identity-file`）から読み取ります

アバターのパスはワークスペースルートからの相対パスで解決されます。

## IDの設定

`set-identity` はフィールドを `agents.list[].identity` に書き込みます:

-   `name`
-   `theme`
-   `emoji`
-   `avatar` (ワークスペース相対パス、http(s) URL、またはデータURI)

`IDENTITY.md` から読み込み:

```bash
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
```

フィールドを明示的に上書き:

```bash
openclaw agents set-identity --agent main --name "OpenClaw" --emoji "🦞" --avatar avatars/openclaw.png
```

設定例:

```json
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "OpenClaw",
          theme: "space lobster",
          emoji: "🦞",
          avatar: "avatars/openclaw.png",
        },
      },
    ],
  },
}
```

[agent](./agent.md)[approvals](./approvals.md)