

  環境とデバッグ

  
# 環境変数

OpenClawは複数のソースから環境変数を取得します。ルールは **既存の値を決して上書きしない** です。

## 優先順位 (高い → 低い)

1.  **プロセス環境** (親シェル/デーモンからGatewayプロセスが既に持っているもの)。
2.  **カレントワーキングディレクトリの `.env`** (dotenvデフォルト; 上書きしない)。
3.  **グローバル `.env`** (`~/.openclaw/.env` にあるもの、別名 `$OPENCLAW_STATE_DIR/.env`; 上書きしない)。
4.  **設定ファイルの `env` ブロック** (`~/.openclaw/openclaw.json` 内; 欠落している場合のみ適用)。
5.  **オプションのログインシェルインポート** (`env.shellEnv.enabled` または `OPENCLAW_LOAD_SHELL_ENV=1`)、期待されるキーが欠落している場合のみ適用。

設定ファイルが完全に存在しない場合、ステップ4はスキップされます。有効になっていればシェルインポートは実行されます。

## 設定ファイルの env ブロック

インライン環境変数を設定する2つの同等の方法 (どちらも非上書き):

```json
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
  },
}
```

## シェル環境インポート

`env.shellEnv` はログインシェルを実行し、**欠落している** 期待されるキーのみをインポートします:

```json
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

環境変数による同等の設定:

-   `OPENCLAW_LOAD_SHELL_ENV=1`
-   `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

## ランタイムで注入される環境変数

OpenClawはまた、生成された子プロセスにコンテキストマーカーを注入します:

-   `OPENCLAW_SHELL=exec`: `exec` ツールを通じて実行されるコマンドに設定。
-   `OPENCLAW_SHELL=acp`: ACPランタイムバックエンドプロセスの生成時に設定 (例: `acpx`)。
-   `OPENCLAW_SHELL=acp-client`: `openclaw acp client` がACPブリッジプロセスを生成するときに設定。
-   `OPENCLAW_SHELL=tui-local`: ローカルTUI `!` シェルコマンドに設定。

これらはランタイムマーカーです (ユーザー設定は不要)。シェル/プロファイルのロジックでコンテキスト固有のルールを適用するために使用できます。

## 設定内での環境変数置換

`${VAR_NAME}` 構文を使用して、設定文字列値内で直接環境変数を参照できます:

```json
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}",
      },
    },
  },
}
```

詳細は [設定: 環境変数置換](../gateway/configuration.md#env-var-substitution-in-config) を参照してください。

## シークレット参照 vs `\${ENV}` 文字列

OpenClawは2つの環境変数駆動パターンをサポートします:

-   設定値内での `${VAR}` 文字列置換。
-   シークレット参照をサポートするフィールドのための SecretRef オブジェクト (`{ source: "env", provider: "default", id: "VAR" }`)。

どちらもアクティベーション時にプロセス環境から解決されます。SecretRefの詳細は [シークレット管理](../gateway/secrets.md) に記載されています。

## パス関連の環境変数

| 変数 | 目的 |
| --- | --- |
| `OPENCLAW_HOME` | すべての内部パス解決 (`~/.openclaw/`、エージェントディレクトリ、セッション、認証情報) に使用されるホームディレクトリを上書きします。OpenClawを専用サービスユーザーとして実行する場合に便利です。 |
| `OPENCLAW_STATE_DIR` | 状態ディレクトリを上書きします (デフォルト `~/.openclaw`)。 |
| `OPENCLAW_CONFIG_PATH` | 設定ファイルのパスを上書きします (デフォルト `~/.openclaw/openclaw.json`)。 |

## ロギング

| 変数 | 目的 |
| --- | --- |
| `OPENCLAW_LOG_LEVEL` | ファイルとコンソールの両方のログレベルを上書きします (例: `debug`, `trace`)。設定内の `logging.level` および `logging.consoleLevel` より優先されます。無効な値は警告とともに無視されます。 |

### OPENCLAW\_HOME

設定すると、`OPENCLAW_HOME` はすべての内部パス解決においてシステムのホームディレクトリ (`$HOME` / `os.homedir()`) を置き換えます。これにより、ヘッドレスサービスアカウントのための完全なファイルシステム分離が可能になります。**優先順位:** `OPENCLAW_HOME` > `$HOME` > `USERPROFILE` > `os.homedir()` **例** (macOS LaunchDaemon):

```
<key>EnvironmentVariables</key>
<dict>
  <key>OPENCLAW_HOME</key>
  <string>/Users/kira</string>
</dict>
```

`OPENCLAW_HOME` はチルダパス (例: `~/svc`) に設定することもでき、使用前に `$HOME` を使用して展開されます。

## 関連項目

-   [Gateway設定](../gateway/configuration.md)
-   [FAQ: 環境変数と .env の読み込み](./faq.md#env-vars-and-env-loading)
-   [モデル概要](../concepts/models.md)

[OpenClaw Lore](../start/lore.md)[デバッグ](./debugging.md)