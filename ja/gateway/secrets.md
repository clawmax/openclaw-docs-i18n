

  設定と運用

  
# シークレット管理

OpenClaw は追加的な SecretRefs をサポートしており、サポート対象の認証情報を設定ファイルに平文で保存する必要がありません。平文も引き続き機能します。SecretRefs は認証情報ごとにオプトインです。

## 目標とランタイムモデル

シークレットはメモリ内のランタイムスナップショットに解決されます。

-   解決はアクティベーション時に積極的に行われ、リクエストパスでは遅延されません。
-   実質的にアクティブな SecretRef が解決できない場合、起動は早期に失敗します。
-   リロードはアトミックスワップを使用します: 完全な成功、または最後に正常だったスナップショットを保持します。
-   ランタイムリクエストは、アクティブなメモリ内スナップショットからのみ読み取ります。

これにより、シークレットプロバイダーの障害がホットなリクエストパスから排除されます。

## アクティブサーフェスのフィルタリング

SecretRefs は、実質的にアクティブなサーフェスでのみ検証されます。

-   有効なサーフェス: 未解決の参照は起動/リロードをブロックします。
-   非アクティブなサーフェス: 未解決の参照は起動/リロードをブロックしません。
-   非アクティブな参照は、コード `SECRETS_REF_IGNORED_INACTIVE_SURFACE` で非致命的な診断情報を出力します。

非アクティブなサーフェスの例:

-   無効化されたチャネル/アカウントエントリ。
-   有効なアカウントが継承しないトップレベルのチャネル認証情報。
-   無効化されたツール/機能サーフェス。
-   `tools.web.search.provider` で選択されていない Web 検索プロバイダー固有のキー。自動モード（プロバイダー未設定）では、プロバイダー固有のキーもプロバイダーの自動検出のためにアクティブです。
-   `gateway.remote.token` / `gateway.remote.password` の SecretRefs は（`gateway.remote.enabled` が `false` でない場合）、以下のいずれかが真の場合にアクティブになります:
    -   `gateway.mode=remote`
    -   `gateway.remote.url` が設定されている
    -   `gateway.tailscale.mode` が `serve` または `funnel` である
    これらのリモートサーフェスがないローカルモードでは:
    -   `gateway.remote.token` は、トークン認証が優先され、環境変数/認証トークンが設定されていない場合にアクティブです。
    -   `gateway.remote.password` は、パスワード認証が優先され、環境変数/認証パスワードが設定されていない場合にのみアクティブです。
-   `gateway.auth.token` の SecretRef は、`OPENCLAW_GATEWAY_TOKEN`（または `CLAWDBOT_GATEWAY_TOKEN`）が設定されている場合、起動時の認証解決では非アクティブです。なぜなら、そのランタイムでは環境変数のトークン入力が優先されるためです。

## ゲートウェイ認証サーフェスの診断

`gateway.auth.token`、`gateway.auth.password`、`gateway.remote.token`、または `gateway.remote.password` に SecretRef が設定されている場合、ゲートウェイの起動/リロードログはサーフェスの状態を明示的に記録します:

-   `active`: SecretRef が有効な認証サーフェスの一部であり、解決されなければなりません。
-   `inactive`: 別の認証サーフェスが優先される、またはリモート認証が無効/アクティブでないため、このランタイムでは SecretRef は無視されます。

これらのエントリは `SECRETS_GATEWAY_AUTH_SURFACE` でログに記録され、アクティブサーフェスポリシーで使用された理由が含まれるため、認証情報がアクティブまたは非アクティブとして扱われた理由を確認できます。

## オンボーディング参照の事前チェック

オンボーディングがインタラクティブモードで実行され、SecretRef ストレージを選択すると、OpenClaw は保存前に事前チェック検証を実行します:

-   環境変数参照: 環境変数名を検証し、オンボーディング中に空でない値が見えることを確認します。
-   プロバイダー参照 (`file` または `exec`): プロバイダー選択を検証し、`id` を解決し、解決された値の型をチェックします。
-   クイックスタート再利用パス: `gateway.auth.token` がすでに SecretRef である場合、オンボーディングはプローブ/ダッシュボードブートストラップの前に（`env`、`file`、`exec` 参照に対して）同じ早期失敗ゲートを使用してそれを解決します。

検証が失敗した場合、オンボーディングはエラーを表示し、再試行を許可します。

## SecretRef 契約

すべての場所で同じオブジェクト形状を使用します:

```json
{ source: "env" | "file" | "exec", provider: "default", id: "..." }
```

### source: "env"

```json
{ source: "env", provider: "default", id: "OPENAI_API_KEY" }
```

検証:

-   `provider` は `^[a-z][a-z0-9_-]{0,63}$` に一致しなければなりません。
-   `id` は `^[A-Z][A-Z0-9_]{0,127}$` に一致しなければなりません。

### source: "file"

```json
{ source: "file", provider: "filemain", id: "/providers/openai/apiKey" }
```

検証:

-   `provider` は `^[a-z][a-z0-9_-]{0,63}$` に一致しなければなりません。
-   `id` は絶対 JSON ポインター (`/...`) でなければなりません。
-   セグメント内の RFC6901 エスケープ: `~` => `~0`, `/` => `~1`

### source: "exec"

```json
{ source: "exec", provider: "vault", id: "providers/openai/apiKey" }
```

検証:

-   `provider` は `^[a-z][a-z0-9_-]{0,63}$` に一致しなければなりません。
-   `id` は `^[A-Za-z0-9][A-Za-z0-9._:/-]{0,255}$` に一致しなければなりません。

## プロバイダー設定

`secrets.providers` の下にプロバイダーを定義します:

```json
{
  secrets: {
    providers: {
      default: { source: "env" },
      filemain: {
        source: "file",
        path: "~/.openclaw/secrets.json",
        mode: "json", // または "singleValue"
      },
      vault: {
        source: "exec",
        command: "/usr/local/bin/openclaw-vault-resolver",
        args: ["--profile", "prod"],
        passEnv: ["PATH", "VAULT_ADDR"],
        jsonOnly: true,
      },
    },
    defaults: {
      env: "default",
      file: "filemain",
      exec: "vault",
    },
    resolution: {
      maxProviderConcurrency: 4,
      maxRefsPerProvider: 512,
      maxBatchBytes: 262144,
    },
  },
}
```

### 環境変数プロバイダー

-   オプションの許可リスト via `allowlist`。
-   欠落/空の環境変数値は解決を失敗させます。

### ファイルプロバイダー

-   `path` からローカルファイルを読み取ります。
-   `mode: "json"` は JSON オブジェクトペイロードを期待し、`id` をポインターとして解決します。
-   `mode: "singleValue"` は参照 ID `"value"` を期待し、ファイルの内容を返します。
-   パスは所有権/パーミッションチェックに合格しなければなりません。
-   Windows のフェイルクローズに関する注意: パスに対して ACL 検証が利用できない場合、解決は失敗します。信頼されたパスのみの場合、そのプロバイダーに `allowInsecurePath: true` を設定してパスセキュリティチェックをバイパスします。

### 実行プロバイダー

-   設定された絶対バイナリパスを実行し、シェルは使用しません。
-   デフォルトでは、`command` は通常ファイル（シンボリックリンクではない）を指さなければなりません。
-   `allowSymlinkCommand: true` を設定して、シンボリックリンクのコマンドパスを許可します（例: Homebrew のシム）。OpenClaw は解決されたターゲットパスを検証します。
-   `allowSymlinkCommand` を `trustedDirs` とペアにして、パッケージマネージャーのパス（例: `["/opt/homebrew"]`）に対応します。
-   タイムアウト、無出力タイムアウト、出力バイト制限、環境変数許可リスト、および信頼されたディレクトリをサポートします。
-   Windows のフェイルクローズに関する注意: コマンドパスに対して ACL 検証が利用できない場合、解決は失敗します。信頼されたパスのみの場合、そのプロバイダーに `allowInsecurePath: true` を設定してパスセキュリティチェックをバイパスします。

リクエストペイロード (stdin):

```json
{ "protocolVersion": 1, "provider": "vault", "ids": ["providers/openai/apiKey"] }
```

レスポンスペイロード (stdout):

```json
{ "protocolVersion": 1, "values": { "providers/openai/apiKey": "<openai-api-key>" } } // pragma: allowlist secret
```

オプションの ID ごとのエラー:

```json
{
  "protocolVersion": 1,
  "values": {},
  "errors": { "providers/openai/apiKey": { "message": "not found" } }
}
```

## 実行統合の例

### 1Password CLI

```json
{
  secrets: {
    providers: {
      onepassword_openai: {
        source: "exec",
        command: "/opt/homebrew/bin/op",
        allowSymlinkCommand: true, // Homebrew のシンボリックリンクバイナリに必要
        trustedDirs: ["/opt/homebrew"],
        args: ["read", "op://Personal/OpenClaw QA API Key/password"],
        passEnv: ["HOME"],
        jsonOnly: false,
      },
    },
  },
  models: {
    providers: {
      openai: {
        baseUrl: "https://api.openai.com/v1",
        models: [{ id: "gpt-5", name: "gpt-5" }],
        apiKey: { source: "exec", provider: "onepassword_openai", id: "value" },
      },
    },
  },
}
```

### HashiCorp Vault CLI

```json
{
  secrets: {
    providers: {
      vault_openai: {
        source: "exec",
        command: "/opt/homebrew/bin/vault",
        allowSymlinkCommand: true, // Homebrew のシンボリックリンクバイナリに必要
        trustedDirs: ["/opt/homebrew"],
        args: ["kv", "get", "-field=OPENAI_API_KEY", "secret/openclaw"],
        passEnv: ["VAULT_ADDR", "VAULT_TOKEN"],
        jsonOnly: false,
      },
    },
  },
  models: {
    providers: {
      openai: {
        baseUrl: "https://api.openai.com/v1",
        models: [{ id: "gpt-5", name: "gpt-5" }],
        apiKey: { source: "exec", provider: "vault_openai", id: "value" },
      },
    },
  },
}
```

### sops

```json
{
  secrets: {
    providers: {
      sops_openai: {
        source: "exec",
        command: "/opt/homebrew/bin/sops",
        allowSymlinkCommand: true, // Homebrew のシンボリックリンクバイナリに必要
        trustedDirs: ["/opt/homebrew"],
        args: ["-d", "--extract", '["providers"]["openai"]["apiKey"]', "/path/to/secrets.enc.json"],
        passEnv: ["SOPS_AGE_KEY_FILE"],
        jsonOnly: false,
      },
    },
  },
  models: {
    providers: {
      openai: {
        baseUrl: "https://api.openai.com/v1",
        models: [{ id: "gpt-5", name: "gpt-5" }],
        apiKey: { source: "exec", provider: "sops_openai", id: "value" },
      },
    },
  },
}
```

## サポートされる認証情報サーフェス

正規のサポート対象および非サポート対象の認証情報は以下にリストされています:

-   [SecretRef 認証情報サーフェス](../reference/secretref-credential-surface.md)

ランタイムで発行される、またはローテーションする認証情報、および OAuth リフレッシュマテリアルは、読み取り専用の SecretRef 解決から意図的に除外されています。

## 必要な動作と優先順位

-   参照なしのフィールド: 変更なし。
-   参照ありのフィールド: アクティベーション中のアクティブサーフェスで必須。
-   平文と参照の両方が存在する場合、サポートされる優先順位パスでは参照が優先されます。

警告と監査シグナル:

-   `SECRETS_REF_OVERRIDES_PLAINTEXT` (ランタイム警告)
-   `REF_SHADOWED` (`auth-profiles.json` の認証情報が `openclaw.json` の参照より優先される場合の監査結果)

Google Chat 互換動作:

-   `serviceAccountRef` は平文の `serviceAccount` より優先されます。
-   兄弟参照が設定されている場合、平文の値は無視されます。

## アクティベーショントリガー

シークレットアクティベーションは以下のタイミングで実行されます:

-   起動時 (事前チェックおよび最終アクティベーション)
-   設定リロードのホット適用パス
-   設定リロードの再起動チェックパス
-   `secrets.reload` による手動リロード

アクティベーション契約:

-   成功すると、スナップショットがアトミックにスワップされます。
-   起動失敗はゲートウェイの起動を中止します。
-   ランタイムリロード失敗は、最後に正常だったスナップショットを保持します。

## 劣化および回復シグナル

正常な状態の後にリロード時のアクティベーションが失敗すると、OpenClaw は劣化したシークレット状態に入ります。ワンショットのシステムイベントとログコード:

-   `SECRETS_RELOADER_DEGRADED`
-   `SECRETS_RELOADER_RECOVERED`

動作:

-   劣化: ランタイムは最後に正常だったスナップショットを保持します。
-   回復: 次の成功したアクティベーションの後に一度だけ出力されます。
-   すでに劣化している状態での繰り返しの失敗は警告をログに記録しますが、イベントをスパムしません。
-   起動時の早期失敗は、ランタイムがアクティブになったことがないため、劣化イベントを出力しません。

## コマンドパス解決

コマンドパスは、ゲートウェイスナップショット RPC を介してサポートされる SecretRef 解決をオプトインできます。大きく2つの動作があります:

-   厳格なコマンドパス（例: `openclaw memory` リモートメモリパスや `openclaw qr --remote`）はアクティブスナップショットから読み取り、必要な SecretRef が利用できない場合に早期に失敗します。
-   読み取り専用コマンドパス（例: `openclaw status`、`openclaw status --all`、`openclaw channels status`、`openclaw channels resolve`、および読み取り専用の doctor/config 修復フロー）もアクティブスナップショットを優先しますが、対象の SecretRef がそのコマンドパスで利用できない場合、中止する代わりに劣化します。

読み取り専用動作:

-   ゲートウェイが実行中の場合、これらのコマンドはまずアクティブスナップショットから読み取ります。
-   ゲートウェイ解決が不完全、またはゲートウェイが利用できない場合、特定のコマンドサーフェスに対して対象を絞ったローカルフォールバックを試みます。
-   対象の SecretRef がまだ利用できない場合、コマンドは劣化した読み取り専用出力と「設定されているがこのコマンドパスでは利用できません」などの明示的な診断情報を出して続行します。
-   この劣化動作はコマンドローカルのみです。ランタイムの起動、リロード、または送信/認証パスを弱めることはありません。

その他の注意点:

-   バックエンドシークレットローテーション後のスナップショット更新は `openclaw secrets reload` で処理されます。
-   これらのコマンドパスで使用されるゲートウェイ RPC メソッド: `secrets.resolve`。

## 監査と設定ワークフロー

デフォルトのオペレーターフロー:

```bash
openclaw secrets audit --check
openclaw secrets configure
openclaw secrets audit --check
```

### secrets audit

結果には以下が含まれます:

-   保存されている平文の値 (`openclaw.json`、`auth-profiles.json`、`.env`、および生成された `agents/*/agent/models.json`)
-   生成された `models.json` エントリ内の平文の機密プロバイダーヘッダーの残骸
-   未解決の参照
-   優先順位のシャドウイング (`auth-profiles.json` が `openclaw.json` の参照より優先される)
-   レガシー残骸 (`auth.json`、OAuth リマインダー)

ヘッダー残骸に関する注意:

-   機密プロバイダーヘッダーの検出は、名前のヒューリスティックに基づいています（一般的な認証/認証情報ヘッダー名とフラグメント、例: `authorization`、`x-api-key`、`token`、`secret`、`password`、`credential`）。

### secrets configure

インタラクティブヘルパーで以下を行います:

-   まず `secrets.providers` を設定します (`env`/`file`/`exec`、追加/編集/削除)
-   `openclaw.json` 内のサポートされるシークレットを含むフィールドと、1つのエージェントスコープに対する `auth-profiles.json` を選択できます
-   ターゲットピッカーで直接新しい `auth-profiles.json` マッピングを作成できます
-   SecretRef の詳細 (`source`、`provider`、`id`) をキャプチャします
-   事前チェック解決を実行します
-   すぐに適用できます

便利なモード:

-   `openclaw secrets configure --providers-only`
-   `openclaw secrets configure --skip-provider-setup`
-   `openclaw secrets configure --agent `

`configure` 適用のデフォルト:

-   対象プロバイダーに対して `auth-profiles.json` から一致する静的認証情報を削除します
-   `auth.json` からレガシーな静的 `api_key` エントリを削除します
-   `<config-dir>/.env` から一致する既知のシークレット行を削除します

### secrets apply

保存されたプランを適用します:

```bash
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run
```

厳密なターゲット/パス契約の詳細と正確な拒否ルールについては、以下を参照してください:

-   [Secrets Apply Plan Contract](./secrets-plan-contract.md)

## 一方向の安全ポリシー

OpenClaw は、履歴の平文シークレット値を含むロールバックバックアップを意図的に書き込みません。安全モデル:

-   書き込みモードの前に事前チェックが成功しなければなりません
-   ランタイムアクティベーションはコミット前に検証されます
-   適用は、アトミックなファイル置換と、失敗時のベストエフォートでの復元を使用してファイルを更新します

## レガシー認証互換性に関する注意

静的認証情報の場合、ランタイムは平文のレガシー認証ストレージに依存しなくなりました。

-   ランタイム認証情報ソースは、解決されたメモリ内スナップショットです。
-   レガシーな静的 `api_key` エントリは、発見されたときに削除されます。
-   OAuth 関連の互換性動作は別個のままです。

## Web UI に関する注意

一部の SecretInput ユニオンは、フォームモードよりも生のエディターモードで設定する方が簡単です。

## 関連ドキュメント

-   CLI コマンド: [secrets](../cli/secrets.md)
-   プラン契約の詳細: [Secrets Apply Plan Contract](./secrets-plan-contract.md)
-   認証情報サーフェス: [SecretRef 認証情報サーフェス](../reference/secretref-credential-surface.md)
-   認証設定: [認証](./authentication.md)
-   セキュリティ態勢: [セキュリティ](./security.md)
-   環境変数の優先順位: [環境変数](../help/environment.md)

[認証認証情報セマンティクス](../auth-credential-semantics.md)[Secrets Apply Plan Contract](./secrets-plan-contract.md)