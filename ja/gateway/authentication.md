title: "OpenClawゲートウェイ認証設定：OAuthとAPIキー"
description: "APIキー、OAuth、セットアップトークンを使用してOpenClawゲートウェイの認証を設定する方法を学びます。セットアップ手順、トラブルシューティング、認証情報管理を含みます。"
keywords: ["openclaw 認証", "ゲートウェイ api キー", "oauth 設定", "anthropic セットアップトークン", "認証資格情報セマンティクス", "モデルステータスプローブ", "secretref 認証", "api キーローテーション"]
---

  設定と運用

  
# 認証

OpenClawはモデルプロバイダーに対してOAuthとAPIキーをサポートしています。常時稼働するゲートウェイホストの場合、APIキーが通常最も予測可能な選択肢です。サブスクリプション/OAuthフローも、お使いのプロバイダーアカウントモデルに合致する場合にサポートされます。完全なOAuthフローとストレージレイアウトについては、[/concepts/oauth](../concepts/oauth.md)を参照してください。SecretRefベースの認証（`env`/`file`/`exec`プロバイダー）については、[シークレット管理](./secrets.md)を参照してください。`models status --probe`で使用される資格情報の適格性/理由コードルールについては、[認証資格情報セマンティクス](../auth-credential-semantics.md)を参照してください。

## 推奨設定（APIキー、任意のプロバイダー）

長期間稼働するゲートウェイを運用する場合は、選択したプロバイダーのAPIキーから始めてください。特にAnthropicの場合、APIキー認証が安全な道筋であり、サブスクリプションのセットアップトークン認証よりも推奨されます。

1.  プロバイダーのコンソールでAPIキーを作成します。
2.  それを**ゲートウェイホスト**（`openclaw gateway`を実行しているマシン）に配置します。

```bash
export <PROVIDER>_API_KEY="..."
openclaw models status
```

3.  ゲートウェイがsystemd/launchdの下で実行される場合、デーモンが読み取れるようにキーを`~/.openclaw/.env`に配置することを推奨します：

```bash
cat >> ~/.openclaw/.env <<'EOF'
<PROVIDER>_API_KEY=...
EOF
```

その後、デーモン（またはゲートウェイプロセス）を再起動し、再確認します：

```bash
openclaw models status
openclaw doctor
```

環境変数を自分で管理したくない場合は、オンボーディングウィザードがデーモン使用のためにAPIキーを保存できます：`openclaw onboard`。環境継承（`env.shellEnv`、`~/.openclaw/.env`、systemd/launchd）の詳細については、[ヘルプ](../help.md)を参照してください。

## Anthropic：セットアップトークン（サブスクリプション認証）

Claudeサブスクリプションを使用している場合、セットアップトークンフローがサポートされています。**ゲートウェイホスト**で実行してください：

```bash
claude setup-token
```

その後、OpenClawに貼り付けます：

```bash
openclaw models auth setup-token --provider anthropic
```

トークンが別のマシンで作成された場合は、手動で貼り付けます：

```bash
openclaw models auth paste-token --provider anthropic
```

以下のようなAnthropicエラーが表示された場合：

```bash
This credential is only authorized for use with Claude Code and cannot be used for other API requests.
```

…代わりにAnthropic APIキーを使用してください。

> **⚠️** Anthropicセットアップトークンのサポートは技術的な互換性のみを目的としています。Anthropicは過去にClaude Code以外での一部のサブスクリプション使用をブロックしたことがあります。ポリシーリスクが許容できると判断した場合にのみ使用し、Anthropicの現在の利用規約を自身で確認してください。

手動トークン入力（任意のプロバイダー；`auth-profiles.json`を書き込み + 設定を更新）：

```bash
openclaw models auth paste-token --provider anthropic
openclaw models auth paste-token --provider openrouter
```

静的認証情報に対して認証プロファイル参照もサポートされています：

-   `api_key`認証情報は`keyRef: { source, provider, id }`を使用できます
-   `token`認証情報は`tokenRef: { source, provider, id }`を使用できます

自動化に適したチェック（期限切れ/欠落時は終了コード`1`、期限切れ間近時は`2`）：

```bash
openclaw models status --check
```

オプションの運用スクリプト（systemd/Termux）はこちらに文書化されています：[/automation/auth-monitoring](../automation/auth-monitoring.md)

> `claude setup-token`は対話型TTYを必要とします。

## モデル認証ステータスの確認

```bash
openclaw models status
openclaw doctor
```

## APIキーローテーション動作（ゲートウェイ）

一部のプロバイダーは、API呼び出しがプロバイダーのレート制限に達した場合に、代替キーでリクエストを再試行することをサポートしています。

-   優先順位：
    -   `OPENCLAW_LIVE__KEY`（単一のオーバーライド）
    -   `_API_KEYS`
    -   `_API_KEY`
    -   `_API_KEY_*`
-   Googleプロバイダーは追加のフォールバックとして`GOOGLE_API_KEY`も含みます。
-   同じキーリストは使用前に重複排除されます。
-   OpenClawはレート制限エラー（例：`429`、`rate_limit`、`quota`、`resource exhausted`）に対してのみ次のキーで再試行します。
-   レート制限以外のエラーは代替キーで再試行されません。
-   すべてのキーが失敗した場合、最後の試行からの最終エラーが返されます。

## 使用する認証情報の制御

### セッションごと（チャットコマンド）

現在のセッションで特定のプロバイダー認証情報を固定するには、`/model <alias-or-id>@`を使用します（プロファイルIDの例：`anthropic:default`、`anthropic:work`）。コンパクトなピッカーには`/model`（または`/model list`）を使用します；完全なビュー（候補 + 次の認証プロファイル、および設定時にプロバイダーエンドポイントの詳細）には`/model status`を使用します。

### エージェントごと（CLIオーバーライド）

エージェントに対して明示的な認証プロファイル順序オーバーライドを設定します（そのエージェントの`auth-profiles.json`に保存されます）：

```bash
openclaw models auth order get --provider anthropic
openclaw models auth order set --provider anthropic anthropic:default
openclaw models auth order clear --provider anthropic
```

特定のエージェントを対象にするには`--agent `を使用します；省略すると設定されたデフォルトエージェントが使用されます。

## トラブルシューティング

### 「資格情報が見つかりません」

Anthropicトークンプロファイルが欠落している場合、**ゲートウェイホスト**で`claude setup-token`を実行し、再確認します：

```bash
openclaw models status
```

### トークンの期限切れ間近/期限切れ

`openclaw models status`を実行して、どのプロファイルが期限切れ間近かを確認します。プロファイルが欠落している場合は、`claude setup-token`を再実行し、トークンを再度貼り付けます。

## 要件

-   Anthropicサブスクリプションアカウント（`claude setup-token`用）
-   Claude Code CLIがインストール済み（`claude`コマンドが利用可能）

[設定例](./configuration-examples.md)[認証資格情報セマンティクス](../auth-credential-semantics.md)