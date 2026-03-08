title: "拡張機能管理のためのOpenClaw CLIプラグインコマンドガイド"
description: "OpenClaw CLIプラグインの一覧表示、インストール、有効化、更新、アンインストール方法を学びます。プラグインのセキュリティ、固定バージョン、バンドル済み拡張機能を管理します。"
keywords: ["openclaw プラグイン", "cli プラグイン管理", "プラグインのインストール", "プラグインコマンド", "ゲートウェイ拡張機能", "プラグインセキュリティ", "プラグインの更新", "プラグインのアンインストール"]
---

  CLIコマンド

  
# plugins

ゲートウェイプラグイン/拡張機能（プロセス内ロード）を管理します。関連項目:

-   プラグインシステム: [プラグイン](../tools/plugin.md)
-   プラグインマニフェスト + スキーマ: [プラグインマニフェスト](../plugins/manifest.md)
-   セキュリティ強化: [セキュリティ](../gateway/security.md)

## コマンド

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins uninstall <id>
openclaw plugins doctor
openclaw plugins update <id>
openclaw plugins update --all
```

バンドル済みプラグインはOpenClawに同梱されていますが、初期状態では無効です。`plugins enable` を使用して有効化します。すべてのプラグインは、インラインJSONスキーマ (`configSchema`、空でも可) を含む `openclaw.plugin.json` ファイルを提供する必要があります。マニフェストやスキーマが欠落している、または無効な場合、プラグインはロードされず、設定検証が失敗します。

### インストール

```bash
openclaw plugins install <path-or-spec>
openclaw plugins install <npm-spec> --pin
```

セキュリティ注意: プラグインのインストールはコードを実行するのと同様に扱ってください。固定バージョンを優先してください。Npm仕様は **レジストリのみ** を対象とします（パッケージ名 + オプションの **正確なバージョン** または **ディストタグ**）。Git/URL/ファイル仕様やセマンティックバージョニングの範囲は拒否されます。依存関係のインストールは安全性のため `--ignore-scripts` を付けて実行されます。ベア仕様と `@latest` は安定版トラックに留まります。npmがこれらをプレリリースに解決した場合、OpenClawは停止し、`@beta`/`@rc` などのプレリリースタグや `@1.2.3-beta.4` などの正確なプレリリースバージョンを明示的に指定するよう求めます。ベアインストール仕様がバンドル済みプラグインID（例: `diffs`）と一致する場合、OpenClawはバンドル済みプラグインを直接インストールします。同じ名前のnpmパッケージをインストールするには、明示的なスコープ付き仕様（例: `@scope/diffs`）を使用してください。サポートされるアーカイブ形式: `.zip`, `.tgz`, `.tar.gz`, `.tar`。ローカルディレクトリのコピーを避けるには `--link` を使用します（`plugins.load.paths` に追加されます）:

```bash
openclaw plugins install -l ./my-plugin
```

npmインストール時に `--pin` を使用すると、解決された正確な仕様 (`name@version`) を `plugins.installs` に保存しつつ、デフォルトの動作（ピン留めなし）を維持します。

### アンインストール

```bash
openclaw plugins uninstall <id>
openclaw plugins uninstall <id> --dry-run
openclaw plugins uninstall <id> --keep-files
```

`uninstall` は、`plugins.entries`、`plugins.installs`、プラグイン許可リスト、および該当する場合はリンクされた `plugins.load.paths` エントリからプラグインレコードを削除します。アクティブなメモリプラグインの場合、メモリスロットは `memory-core` にリセットされます。デフォルトでは、アンインストールはアクティブな状態ディレクトリの拡張機能ルート (`$OPENCLAW_STATE_DIR/extensions/`) 下のプラグインインストールディレクトリも削除します。ファイルをディスク上に保持するには `--keep-files` を使用します。`--keep-config` は `--keep-files` の非推奨エイリアスとしてサポートされています。

### 更新

```bash
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins update <id> --dry-run
```

更新はnpmからインストールされたプラグイン（`plugins.installs` で追跡）にのみ適用されます。保存された整合性ハッシュが存在し、取得されたアーティファクトのハッシュが変更された場合、OpenClawは警告を表示し、続行する前に確認を求めます。CI/非対話実行でプロンプトをバイパスするには、グローバルオプション `--yes` を使用します。

[pairing](./pairing.md)[qr](./qr.md)

---