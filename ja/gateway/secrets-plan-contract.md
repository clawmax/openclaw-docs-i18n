

  設定と操作

  
# Secrets Apply プラン契約

このページでは、`openclaw secrets apply` によって強制される厳格な契約を定義します。ターゲットがこれらのルールに一致しない場合、設定を変更する前に適用は失敗します。

## プランファイルの構造

`openclaw secrets apply --from <plan.json>` は、プランターゲットの `targets` 配列を期待します：

```json
{
  version: 1,
  protocolVersion: 1,
  targets: [
    {
      type: "models.providers.apiKey",
      path: "models.providers.openai.apiKey",
      pathSegments: ["models", "providers", "openai", "apiKey"],
      providerId: "openai",
      ref: { source: "env", provider: "default", id: "OPENAI_API_KEY" },
    },
    {
      type: "auth-profiles.api_key.key",
      path: "profiles.openai:default.key",
      pathSegments: ["profiles", "openai:default", "key"],
      agentId: "main",
      ref: { source: "env", provider: "default", id: "OPENAI_API_KEY" },
    },
  ],
}
```

## サポート対象のターゲットスコープ

プランターゲットは、以下のサポート対象認証情報パスに対して受け入れられます：

-   [SecretRef 認証情報サーフェス](../reference/secretref-credential-surface.md)

## ターゲットタイプの動作

一般的なルール：

-   `target.type` は認識可能でなければならず、正規化された `target.path` の形状と一致しなければなりません。

既存のプランに対しては互換性エイリアスが引き続き受け入れられます：

-   `models.providers.apiKey`
-   `skills.entries.apiKey`
-   `channels.googlechat.serviceAccount`

## パス検証ルール

各ターゲットは以下のすべてで検証されます：

-   `type` は認識可能なターゲットタイプでなければなりません。
-   `path` は空でないドットパスでなければなりません。
-   `pathSegments` は省略可能です。提供される場合、`path` とまったく同じパスに正規化されなければなりません。
-   禁止されたセグメントは拒否されます：`__proto__`、`prototype`、`constructor`。
-   正規化されたパスは、ターゲットタイプに対して登録されたパス形状と一致しなければなりません。
-   `providerId` または `accountId` が設定されている場合、パスにエンコードされたIDと一致しなければなりません。
-   `auth-profiles.json` ターゲットには `agentId` が必要です。
-   新しい `auth-profiles.json` マッピングを作成する場合、`authProfileProvider` を含めてください。

## 失敗時の動作

ターゲットが検証に失敗した場合、適用は以下のようなエラーで終了します：

```bash
Invalid plan target path for models.providers.apiKey: models.providers.openai.baseUrl
```

無効なプランに対して書き込みはコミットされません。

## ランタイムおよび監査スコープに関する注意

-   参照専用の `auth-profiles.json` エントリ（`keyRef`/`tokenRef`）は、ランタイム解決と監査範囲に含まれます。
-   `secrets apply` は、サポート対象の `openclaw.json` ターゲット、サポート対象の `auth-profiles.json` ターゲット、およびオプションのスクラブターゲットを書き込みます。

## オペレーターチェック

```bash
# 書き込みなしでプランを検証
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json --dry-run

# その後、実際に適用
openclaw secrets apply --from /tmp/openclaw-secrets-plan.json
```

適用が無効なターゲットパスメッセージで失敗した場合、`openclaw secrets configure` でプランを再生成するか、上記のサポート対象形状にターゲットパスを修正してください。

## 関連ドキュメント

-   [シークレット管理](./secrets.md)
-   [CLI `secrets`](../cli/secrets.md)
-   [SecretRef 認証情報サーフェス](../reference/secretref-credential-surface.md)
-   [設定リファレンス](./configuration-reference.md)

[シークレット管理](./secrets.md)[信頼済みプロキシ認証](./trusted-proxy-auth.md)