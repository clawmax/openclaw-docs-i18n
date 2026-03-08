

  設定

  
# モデルフェイルオーバー

OpenClawは障害を2段階で処理します：

1.  現在のプロバイダー内での**認証プロファイルローテーション**。
2.  **モデルフォールバック**：`agents.defaults.model.fallbacks`の次のモデルへ。

このドキュメントでは、ランタイムルールとそれを支えるデータについて説明します。

## 認証ストレージ（キー + OAuth）

OpenClawはAPIキーとOAuthトークンの両方に**認証プロファイル**を使用します。

-   シークレットは`~/.openclaw/agents//agent/auth-profiles.json`に保存されます（レガシー：`~/.openclaw/agent/auth-profiles.json`）。
-   設定`auth.profiles` / `auth.order`は**メタデータとルーティングのみ**を対象とします（シークレットは含まれません）。
-   レガシーインポート専用OAuthファイル：`~/.openclaw/credentials/oauth.json`（初回使用時に`auth-profiles.json`にインポートされます）。

詳細：[/concepts/oauth](./oauth.md) 認証情報の種類：

-   `type: "api_key"` → `{ provider, key }`
-   `type: "oauth"` → `{ provider, access, refresh, expires, email? }`（一部のプロバイダーでは`projectId`/`enterpriseUrl`も含む）

## プロファイルID

OAuthログインは個別のプロファイルを作成するため、複数のアカウントを共存させることができます。

-   デフォルト：メールアドレスが利用できない場合は`provider:default`。
-   メールアドレス付きOAuth：`provider:`（例：`google-antigravity:user@gmail.com`）。

プロファイルは`~/.openclaw/agents//agent/auth-profiles.json`の`profiles`以下に保存されます。

## ローテーション順序

プロバイダーに複数のプロファイルがある場合、OpenClawは以下のような順序で選択します：

1.  **明示的な設定**：`auth.order[provider]`（設定されている場合）。
2.  **設定済みプロファイル**：プロバイダーでフィルタリングされた`auth.profiles`。
3.  **保存済みプロファイル**：`auth-profiles.json`内のそのプロバイダーのエントリ。

明示的な順序が設定されていない場合、OpenClawはラウンドロビン順序を使用します：

-   **プライマリキー：** プロファイルタイプ（**APIキーよりOAuthを優先**）。
-   **セカンダリキー：** `usageStats.lastUsed`（各タイプ内で最も古いものから順に）。
-   **クールダウン中/無効化されたプロファイル**は末尾に移動され、有効期限が最も早いものから順に並べられます。

### セッションスティッキネス（キャッシュに優しい）

OpenClawは**選択された認証プロファイルをセッションごとに固定**し、プロバイダーのキャッシュをウォームに保ちます。**リクエストごとにローテーションしません**。固定されたプロファイルは以下のいずれかが発生するまで再利用されます：

-   セッションがリセットされる（`/new` / `/reset`）
-   コンパクションが完了する（コンパクションカウントが増加する）
-   プロファイルがクールダウン中/無効化されている

`/model …@`による手動選択は、そのセッションの**ユーザーオーバーライド**を設定し、新しいセッションが開始されるまで自動ローテーションされません。自動固定プロファイル（セッションルーターによって選択されたもの）は**優先設定**として扱われます：最初に試行されますが、レート制限/タイムアウトが発生した場合、OpenClawは別のプロファイルにローテーションする可能性があります。ユーザー固定プロファイルはそのプロファイルにロックされ、それが失敗しモデルフォールバックが設定されている場合、OpenClawはプロファイルを切り替える代わりに次のモデルに移動します。

### OAuthが「失われたように見える」理由

同じプロバイダーに対してOAuthプロファイルとAPIキープロファイルの両方を持っている場合、固定されていない限り、ラウンドロビンによってメッセージ間でそれらが切り替わる可能性があります。単一のプロファイルを強制するには：

-   `auth.order[provider] = ["provider:profileId"]`で固定する、または
-   プロファイルオーバーライド付きの`/model …`を使用してセッションごとのオーバーライドを使用する（UI/チャット画面でサポートされている場合）。

## クールダウン

プロファイルが認証/レート制限エラー（またはレート制限のように見えるタイムアウト）で失敗した場合、OpenClawはそれをクールダウン中としてマークし、次のプロファイルに移動します。フォーマット/無効リクエストエラー（例：Cloud Code Assistツール呼び出しID検証失敗）は、フェイルオーバー対象として扱われ、同じクールダウンを使用します。`Unhandled stop reason: error`、`stop reason: error`、`reason: error`などのOpenAI互換の停止理由エラーは、タイムアウト/フェイルオーバーシグナルとして分類されます。クールダウンは指数バックオフを使用します：

-   1分
-   5分
-   25分
-   1時間（上限）

状態は`auth-profiles.json`の`usageStats`以下に保存されます：

```json
{
  "usageStats": {
    "provider:profile": {
      "lastUsed": 1736160000000,
      "cooldownUntil": 1736160600000,
      "errorCount": 2
    }
  }
}
```

## 課金無効化

課金/クレジット失敗（例：「クレジット不足」/「クレジット残高が少なすぎます」）はフェイルオーバー対象として扱われますが、通常は一時的なものではありません。短いクールダウンの代わりに、OpenClawはプロファイルを**無効化**（より長いバックオフ付き）としてマークし、次のプロファイル/プロバイダーにローテーションします。状態は`auth-profiles.json`に保存されます：

```json
{
  "usageStats": {
    "provider:profile": {
      "disabledUntil": 1736178000000,
      "disabledReason": "billing"
    }
  }
}
```

デフォルト：

-   課金バックオフは**5時間**から開始し、課金失敗ごとに倍増し、**24時間**で上限に達します。
-   バックオフカウンターは、プロファイルが**24時間**失敗していない場合にリセットされます（設定可能）。

## モデルフォールバック

プロバイダーのすべてのプロファイルが失敗した場合、OpenClawは`agents.defaults.model.fallbacks`の次のモデルに移動します。これは、認証失敗、レート制限、およびプロファイルローテーションを枯渇させたタイムアウトに適用されます（他のエラーではフォールバックは進みません）。実行がモデルオーバーライド（フックまたはCLI）で開始された場合、フォールバックは設定されたフォールバックを試行した後、`agents.defaults.model.primary`で終了します。

## 関連設定

以下の設定については[ゲートウェイ設定](../gateway/configuration.md)を参照してください：

-   `auth.profiles` / `auth.order`
-   `auth.cooldowns.billingBackoffHours` / `auth.cooldowns.billingBackoffHoursByProvider`
-   `auth.cooldowns.billingMaxHours` / `auth.cooldowns.failureWindowHours`
-   `agents.defaults.model.primary` / `agents.defaults.model.fallbacks`
-   `agents.defaults.imageModel` ルーティング

より広範なモデル選択とフォールバックの概要については[モデル](./models.md)を参照してください。

[モデルプロバイダー](./model-providers.md)[Anthropic](../providers/anthropic.md)