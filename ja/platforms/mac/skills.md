

  macOSコンパニオンアプリ

  
# スキル

macOSアプリは、OpenClawスキルをゲートウェイ経由で表示します。スキルをローカルで解析することはありません。

## データソース

-   `skills.status` (ゲートウェイ) は、すべてのスキルと、その適格性、不足要件（バンドルスキルに対する許可リストブロックを含む）を返します。
-   要件は、各 `SKILL.md` 内の `metadata.openclaw.requires` から導出されます。

## インストールアクション

-   `metadata.openclaw.install` はインストールオプション (brew/node/go/uv) を定義します。
-   アプリは `skills.install` を呼び出して、ゲートウェイホスト上でインストーラーを実行します。
-   ゲートウェイは、複数のインストーラーが提供されている場合、優先される1つのインストーラーのみを表示します（利用可能な場合はbrew、それ以外の場合は `skills.install` からのノードマネージャー、デフォルトはnpm）。

## 環境変数/APIキー

-   アプリはキーを `~/.openclaw/openclaw.json` の `skills.entries.` 以下に保存します。
-   `skills.update` は `enabled`、`apiKey`、`env` をパッチ適用します。

## リモートモード

-   インストールと設定の更新は、ゲートウェイホスト上で行われます（ローカルのMac上ではありません）。

[macOS IPC](./xpc.md)[Peekaboo Bridge](./peekaboo.md)

---