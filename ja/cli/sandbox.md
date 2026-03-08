

  CLI コマンド

  
# サンドボックス CLI

セキュリティのため、分離された Docker コンテナ内でエージェントを実行する OpenClaw のサンドボックスコンテナを管理します。

## 概要

OpenClaw はセキュリティのために、エージェントを分離された Docker コンテナ内で実行できます。`sandbox` コマンドは、特に更新後や設定変更後にこれらのコンテナを管理するのに役立ちます。

## コマンド

### openclaw sandbox explain

**実効的な** サンドボックスモード/スコープ/ワークスペースアクセス、サンドボックスツールポリシー、および昇格ゲート（修正用の設定キーパス付き）を検査します。

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

### openclaw sandbox list

すべてのサンドボックスコンテナとそのステータス、設定を一覧表示します。

```bash
openclaw sandbox list
openclaw sandbox list --browser  # ブラウザコンテナのみを一覧表示
openclaw sandbox list --json     # JSON 出力
```

**出力内容:**

-   コンテナ名とステータス (実行中/停止中)
-   Docker イメージと設定に一致するかどうか
-   経過時間 (作成からの時間)
-   アイドル時間 (最終使用からの時間)
-   関連するセッション/エージェント

### openclaw sandbox recreate

サンドボックスコンテナを削除し、更新されたイメージ/設定で強制的に再作成します。

```bash
openclaw sandbox recreate --all                # すべてのコンテナを再作成
openclaw sandbox recreate --session main       # 特定のセッション
openclaw sandbox recreate --agent mybot        # 特定のエージェント
openclaw sandbox recreate --browser            # ブラウザコンテナのみ
openclaw sandbox recreate --all --force        # 確認をスキップ
```

**オプション:**

-   `--all`: すべてのサンドボックスコンテナを再作成
-   `--session `: 特定のセッション用のコンテナを再作成
-   `--agent `: 特定のエージェント用のコンテナを再作成
-   `--browser`: ブラウザコンテナのみを再作成
-   `--force`: 確認プロンプトをスキップ

**重要:** コンテナは、次にエージェントが使用されるときに自動的に再作成されます。

## ユースケース

### Docker イメージを更新した後

```bash
# 新しいイメージをプル
docker pull openclaw-sandbox:latest
docker tag openclaw-sandbox:latest openclaw-sandbox:bookworm-slim

# 新しいイメージを使用するように設定を更新
# 設定を編集: agents.defaults.sandbox.docker.image (または agents.list[].sandbox.docker.image)

# コンテナを再作成
openclaw sandbox recreate --all
```

### サンドボックス設定を変更した後

```bash
# 設定を編集: agents.defaults.sandbox.* (または agents.list[].sandbox.*)

# 新しい設定を適用するために再作成
openclaw sandbox recreate --all
```

### setupCommand を変更した後

```bash
openclaw sandbox recreate --all
# または単一のエージェントのみ:
openclaw sandbox recreate --agent family
```

### 特定のエージェントのみの場合

```bash
# 単一のエージェントのコンテナのみを更新
openclaw sandbox recreate --agent alfred
```

## なぜこれが必要なのか？

**問題:** サンドボックスの Docker イメージや設定を更新すると:

-   既存のコンテナは古い設定で実行を継続します
-   コンテナは24時間の非アクティブ後にのみ削除されます
-   定期的に使用されるエージェントは古いコンテナを無期限に実行し続けます

**解決策:** `openclaw sandbox recreate` を使用して古いコンテナを強制的に削除します。次に必要になったときに現在の設定で自動的に再作成されます。ヒント: 手動での `docker rm` よりも `openclaw sandbox recreate` を優先してください。これは Gateway のコンテナ命名規則を使用し、スコープ/セッションキーが変更されたときの不一致を回避します。

## 設定

サンドボックス設定は、`~/.openclaw/openclaw.json` の `agents.defaults.sandbox` にあります（エージェントごとの上書きは `agents.list[].sandbox` に記述します）:

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "all", // off, non-main, all
        "scope": "agent", // session, agent, shared
        "docker": {
          "image": "openclaw-sandbox:bookworm-slim",
          "containerPrefix": "openclaw-sbx-",
          // ... その他の Docker オプション
        },
        "prune": {
          "idleHours": 24, // 24時間アイドル後に自動削除
          "maxAgeDays": 7, // 7日後に自動削除
        },
      },
    },
  },
}
```

## 関連項目

-   [サンドボックスドキュメント](../gateway/sandboxing.md)
-   [エージェント設定](../concepts/agent-workspace.md)
-   [Doctor コマンド](../gateway/doctor.md) - サンドボックス設定を確認

[reset](./reset.md)[secrets](./secrets.md)

---