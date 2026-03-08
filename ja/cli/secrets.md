

  CLI コマンド

  
# secrets

`openclaw secrets` を使用して SecretRef を管理し、アクティブなランタイムスナップショットを健全に保ちます。コマンドの役割:

-   `reload`: ゲートウェイ RPC (`secrets.reload`) で、参照を再解決し、完全成功時のみランタイムスナップショットを交換します（設定書き込みなし）。
-   `audit`: 設定/認証/生成モデルストアおよびレガシー残留物を読み取り専用でスキャンし、平文、未解決の参照、優先順位のずれを検出します。
-   `configure`: プロバイダー設定、ターゲットマッピング、および事前チェックのための対話型プランナー（TTY 必須）。
-   `apply`: 保存されたプランを実行し（`--dry-run` は検証のみ）、ターゲットの平文残留物を消去します。

推奨オペレーターループ:

```bash
openclaw secrets audit --check
openclaw secrets configure
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
openclaw secrets audit --check
openclaw secrets reload
```

CI/ゲート用終了コードに関する注意:

-   `audit --check` は、所見がある場合に `1` を返します。
-   未解決の参照は `2` を返します。

関連情報:

-   シークレットガイド: [シークレット管理](../gateway/secrets.md)
-   認証情報サーフェス: [SecretRef 認証情報サーフェス](../reference/secretref-credential-surface.md)
-   セキュリティガイド: [セキュリティ](../gateway/security.md)

## ランタイムスナップショットの再読み込み

シークレット参照を再解決し、アトミックにランタイムスナップショットを交換します。

```bash
openclaw secrets reload
openclaw secrets reload --json
```

注意:

-   ゲートウェイ RPC メソッド `secrets.reload` を使用します。
-   解決に失敗した場合、ゲートウェイは最後に正常だったスナップショットを保持し、エラーを返します（部分的なアクティベーションはありません）。
-   JSON レスポンスには `warningCount` が含まれます。

## 監査

OpenClaw の状態をスキャンして以下を検出します:

-   平文でのシークレット保存
-   未解決の参照
-   優先順位のずれ (`auth-profiles.json` の認証情報が `openclaw.json` の参照をシャドウしている状態)
-   生成された `agents/*/agent/models.json` の残留物（プロバイダーの `apiKey` 値および機密プロバイダーヘッダー）
-   レガシー残留物（レガシー認証ストアエントリ、OAuth リマインダー）

ヘッダー残留物に関する注意:

-   機密プロバイダーヘッダーの検出は、名前のヒューリスティックに基づいています（一般的な認証/認証情報ヘッダー名およびフラグメント、例: `authorization`, `x-api-key`, `token`, `secret`, `password`, `credential`）。

```bash
openclaw secrets audit
openclaw secrets audit --check
openclaw secrets audit --json
```

終了動作:

-   `--check` は、所見がある場合に非ゼロで終了します。
-   未解決の参照は、より優先度の高い非ゼロコードで終了します。

レポート構造のハイライト:

-   `status`: `clean | findings | unresolved`
-   `summary`: `plaintextCount`, `unresolvedRefCount`, `shadowedRefCount`, `legacyResidueCount`
-   所見コード:
    -   `PLAINTEXT_FOUND`
    -   `REF_UNRESOLVED`
    -   `REF_SHADOWED`
    -   `LEGACY_RESIDUE`

## 設定（対話型ヘルパー）

プロバイダーと SecretRef の変更を対話的に構築し、事前チェックを実行し、オプションで適用します:

```bash
openclaw secrets configure
openclaw secrets configure --plan-out /tmp/openclaw-secrets-plan.json
openclaw secrets configure --apply --yes
openclaw secrets configure --providers-only
openclaw secrets configure --skip-provider-setup
openclaw secrets configure --agent ops
openclaw secrets configure --json
```

フロー:

-   最初にプロバイダー設定 (`secrets.providers` エイリアスの `add/edit/remove`)。
-   次に認証情報マッピング（フィールドを選択し、`{source, provider, id}` 参照を割り当てます）。
-   最後に事前チェックとオプションの適用。

フラグ:

-   `--providers-only`: `secrets.providers` のみを設定し、認証情報マッピングをスキップします。
-   `--skip-provider-setup`: プロバイダー設定をスキップし、既存のプロバイダーに認証情報をマッピングします。
-   `--agent `: `auth-profiles.json` のターゲット検出と書き込みを単一のエージェントストアに限定します。

注意:

-   対話型 TTY が必要です。
-   `--providers-only` と `--skip-provider-setup` を組み合わせることはできません。
-   `configure` は、選択されたエージェントスコープについて、`openclaw.json` 内のシークレットを含むフィールドと `auth-profiles.json` をターゲットとします。
-   `configure` は、ピッカーフロー内で直接新しい `auth-profiles.json` マッピングを作成することをサポートします。
-   正規のサポートサーフェス: [SecretRef 認証情報サーフェス](../reference/secretref-credential-surface.md)。
-   適用前に事前解決チェックを実行します。
-   生成されたプランは、デフォルトで消去オプションが有効です (`scrubEnv`, `scrubAuthProfilesForProviderTargets`, `scrubLegacyAuthJson` すべて有効)。
-   適用パスは、消去された平文値に対して一方向です。
-   `--apply` なしの場合、CLI は事前チェック後に `Apply this plan now?` とプロンプトを表示します。
-   `--apply` あり（かつ `--yes` なし）の場合、CLI は取り消し不可能な追加の確認をプロンプトします。

Exec プロバイダー安全性に関する注意:

-   Homebrew インストールでは、`/opt/homebrew/bin/*` の下にシンボリックリンクされたバイナリが公開されることがよくあります。
-   `allowSymlinkCommand: true` は、信頼されたパッケージマネージャーパスが必要な場合にのみ設定し、`trustedDirs`（例: `["/opt/homebrew"]`）と組み合わせて使用してください。
-   Windows では、プロバイダーパスに対して ACL 検証が利用できない場合、OpenClaw は安全側に倒れて失敗します。信頼されたパスのみの場合、そのプロバイダーに `allowInsecurePath: true` を設定してパスセキュリティチェックをバイパスできます。

## 保存済みプランの適用

以前に生成されたプランを適用または事前チェックします:

```bash
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --json
```

プラン契約の詳細（許可されたターゲットパス、検証ルール、失敗時のセマンティクス）:

-   [シークレット適用プラン契約](../gateway/secrets-plan-contract.md)

`apply` が更新する可能性があるもの:

-   `openclaw.json` (SecretRef ターゲット + プロバイダーのアップサート/削除)
-   `auth-profiles.json` (プロバイダーターゲットの消去)
-   レガシー `auth.json` 残留物
-   移行された値を持つ `~/.openclaw/.env` の既知のシークレットキー

## ロールバックバックアップがない理由

`secrets apply` は、意図的に古い平文値を含むロールバックバックアップを書き込みません。安全性は、厳格な事前チェック + 失敗時のベストエフォートによるメモリ内復元を伴うアトミックな適用によって確保されます。

## 例

```bash
openclaw secrets audit --check
openclaw secrets configure
openclaw secrets audit --check
```

`audit --check` が依然として平文の所見を報告する場合は、残りの報告されたターゲットパスを更新し、監査を再実行してください。

[Sandbox CLI](./sandbox.md)[セキュリティ](./security.md)