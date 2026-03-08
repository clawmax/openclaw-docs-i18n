

  CLI コマンド

  
# hooks

エージェントフック（`/new`、`/reset`、ゲートウェイ起動などのコマンドに対するイベント駆動型自動化）を管理します。関連項目:

-   フック: [フック](../automation/hooks.md)
-   プラグインフック: [プラグイン](../tools/plugin.md#plugin-hooks)

## すべてのフックを一覧表示

```bash
openclaw hooks list
```

ワークスペース、管理対象、バンドル済みディレクトリから発見されたすべてのフックを一覧表示します。**オプション:**

-   `--eligible`: 適格なフックのみを表示（要件を満たしているもの）
-   `--json`: JSON形式で出力
-   `-v, --verbose`: 不足している要件を含む詳細情報を表示

**出力例:**

```
フック (4/4 準備完了)

準備完了:
  🚀 boot-md ✓ - ゲートウェイ起動時に BOOT.md を実行
  📎 bootstrap-extra-files ✓ - エージェントブートストラップ中に追加のワークスペースブートストラップファイルを注入
  📝 command-logger ✓ - すべてのコマンドイベントを一元化された監査ファイルに記録
  💾 session-memory ✓ - /new コマンド発行時にセッションコンテキストをメモリに保存
```

**例 (詳細):**

```bash
openclaw hooks list --verbose
```

適格でないフックの不足要件を表示します。**例 (JSON):**

```bash
openclaw hooks list --json
```

プログラムで使用するための構造化されたJSONを返します。

## フック情報を取得

```bash
openclaw hooks info <name>
```

特定のフックに関する詳細情報を表示します。**引数:**

-   ``: フック名 (例: `session-memory`)

**オプション:**

-   `--json`: JSON形式で出力

**例:**

```bash
openclaw hooks info session-memory
```

**出力:**

```
💾 session-memory ✓ 準備完了

/new コマンド発行時にセッションコンテキストをメモリに保存

詳細:
  ソース: openclaw-bundled
  パス: /path/to/openclaw/hooks/bundled/session-memory/HOOK.md
  ハンドラー: /path/to/openclaw/hooks/bundled/session-memory/handler.ts
  ホームページ: https://docs.openclaw.ai/automation/hooks#session-memory
  イベント: command:new

要件:
  設定: ✓ workspace.dir
```

## フックの適格性を確認

```bash
openclaw hooks check
```

フックの適格性ステータス（準備完了数 vs 未完了数）の概要を表示します。**オプション:**

-   `--json`: JSON形式で出力

**出力例:**

```
フックステータス

総フック数: 4
準備完了: 4
未完了: 0
```

## フックを有効化

```bash
openclaw hooks enable <name>
```

特定のフックを有効化し、設定（`~/.openclaw/config.json`）に追加します。**注:** プラグインによって管理されるフックは `openclaw hooks list` で `plugin:` と表示され、ここでは有効化/無効化できません。代わりにプラグイン自体を有効化/無効化してください。**引数:**

-   ``: フック名 (例: `session-memory`)

**例:**

```bash
openclaw hooks enable session-memory
```

**出力:**

```
✓ フックを有効化しました: 💾 session-memory
```

**動作内容:**

-   フックが存在し、適格であるかを確認
-   設定内の `hooks.internal.entries..enabled = true` を更新
-   設定をディスクに保存

**有効化後:**

-   フックを再読み込みするためにゲートウェイを再起動してください（macOSではメニューバーアプリの再起動、開発中であればゲートウェイプロセスの再起動）。

## フックを無効化

```bash
openclaw hooks disable <name>
```

特定のフックを無効化し、設定を更新します。**引数:**

-   ``: フック名 (例: `command-logger`)

**例:**

```bash
openclaw hooks disable command-logger
```

**出力:**

```
⏸ フックを無効化しました: 📝 command-logger
```

**無効化後:**

-   フックを再読み込みするためにゲートウェイを再起動してください

## フックをインストール

```bash
openclaw hooks install <path-or-spec>
openclaw hooks install <npm-spec> --pin
```

ローカルフォルダー/アーカイブまたはnpmからフックパックをインストールします。Npm仕様は**レジストリのみ**（パッケージ名 + オプションの**正確なバージョン**または**ディストタグ**）です。Git/URL/ファイル仕様とセマバー範囲は拒否されます。依存関係のインストールは安全性のため `--ignore-scripts` で実行されます。ベア仕様と `@latest` は安定版トラックに留まります。npmがこれらのいずれかをプレリリースに解決した場合、OpenClawは停止し、`@beta`/`@rc` などのプレリリースタグまたは正確なプレリリースバージョンを明示的に指定してオプトインするよう求めます。**動作内容:**

-   フックパックを `~/.openclaw/hooks/` にコピー
-   インストールされたフックを `hooks.internal.entries.*` で有効化
-   インストールを `hooks.internal.installs` に記録

**オプション:**

-   `-l, --link`: コピーせずにローカルディレクトリをリンク（`hooks.internal.load.extraDirs` に追加）
-   `--pin`: npmインストールを正確な解決済み `name@version` として `hooks.internal.installs` に記録

**サポートされるアーカイブ:** `.zip`, `.tgz`, `.tar.gz`, `.tar` **例:**

```bash
# ローカルディレクトリ
openclaw hooks install ./my-hook-pack

# ローカルアーカイブ
openclaw hooks install ./my-hook-pack.zip

# NPMパッケージ
openclaw hooks install @openclaw/my-hook-pack

# コピーせずにローカルディレクトリをリンク
openclaw hooks install -l ./my-hook-pack
```

## フックを更新

```bash
openclaw hooks update <id>
openclaw hooks update --all
```

インストール済みのフックパック（npmインストールのみ）を更新します。**オプション:**

-   `--all`: 追跡中のすべてのフックパックを更新
-   `--dry-run`: 書き込みを行わずに変更内容を表示

保存された整合性ハッシュが存在し、取得したアーティファクトのハッシュが変更された場合、OpenClawは警告を表示し、続行前に確認を求めます。CI/非対話型実行でプロンプトをバイパスするには、グローバルオプション `--yes` を使用してください。

## バンドル済みフック

### session-memory

`/new` コマンドを発行したときにセッションコンテキストをメモリに保存します。**有効化:**

```bash
openclaw hooks enable session-memory
```

**出力:** `~/.openclaw/workspace/memory/YYYY-MM-DD-slug.md` **参照:** [session-memory ドキュメント](../automation/hooks.md#session-memory)

### bootstrap-extra-files

`agent:bootstrap` 中に追加のブートストラップファイル（例: モノレポローカルの `AGENTS.md` / `TOOLS.md`）を注入します。**有効化:**

```bash
openclaw hooks enable bootstrap-extra-files
```

**参照:** [bootstrap-extra-files ドキュメント](../automation/hooks.md#bootstrap-extra-files)

### command-logger

すべてのコマンドイベントを一元化された監査ファイルに記録します。**有効化:**

```bash
openclaw hooks enable command-logger
```

**出力:** `~/.openclaw/logs/commands.log` **ログを表示:**

```bash
# 最近のコマンド
tail -n 20 ~/.openclaw/logs/commands.log

# 整形表示
cat ~/.openclaw/logs/commands.log | jq .

# アクションでフィルタ
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**参照:** [command-logger ドキュメント](../automation/hooks.md#command-logger)

### boot-md

ゲートウェイ起動時（チャネル起動後）に `BOOT.md` を実行します。**イベント**: `gateway:startup` **有効化**:

```bash
openclaw hooks enable boot-md
```

**参照:** [boot-md ドキュメント](../automation/hooks.md#boot-md)

[health](./health.md)[logs](./logs.md)