

  基本

  
# OAuth

OpenClawは、それを提供するプロバイダー（特に**OpenAI Codex (ChatGPT OAuth)**）に対して、OAuthによる「サブスクリプション認証」をサポートしています。Anthropicサブスクリプションの場合は、**setup-token**フローを使用してください。AnthropicサブスクリプションのClaude Code外での使用は、過去に一部のユーザーに対して制限されたことがあります。そのため、ユーザー自身の判断によるリスクとして扱い、現在のAnthropicポリシーを自身で確認してください。OpenAI Codex OAuthは、OpenClawのような外部ツールでの使用が明示的にサポートされています。このページでは以下を説明します：本番環境でのAnthropicについては、サブスクリプションsetup-token認証よりもAPIキー認証が安全で推奨されるパスです。

-   OAuth**トークン交換**の仕組み（PKCE）
-   トークンが**保存**される場所（とその理由）
-   **複数アカウント**の扱い方（プロファイル＋セッションごとのオーバーライド）

OpenClawは、独自のOAuthまたはAPIキーフローを提供する**プロバイダープラグイン**もサポートしています。以下のコマンドで実行します：

```bash
openclaw models auth login --provider <id>
```

## トークンシンク（存在する理由）

OAuthプロバイダーは、ログイン/リフレッシュフロー中に**新しいリフレッシュトークン**を発行することが一般的です。一部のプロバイダー（またはOAuthクライアント）は、同じユーザー/アプリに対して新しいトークンが発行された場合、古いリフレッシュトークンを無効化することがあります。実際の症状：

-   OpenClaw*と*Claude Code / Codex CLIの両方でログインすると、後でどちらかがランダムに「ログアウト」される

これを軽減するため、OpenClawは`auth-profiles.json`を**トークンシンク**として扱います：

-   ランタイムは**一箇所**から認証情報を読み取る
-   複数のプロファイルを保持し、確定的にルーティングできる

## ストレージ（トークンの保存場所）

シークレットは**エージェントごと**に保存されます：

-   認証プロファイル（OAuth + APIキー + オプションの値レベル参照）：`~/.openclaw/agents//agent/auth-profiles.json`
-   レガシー互換ファイル：`~/.openclaw/agents//agent/auth.json`（静的`api_key`エントリは発見時に削除されます）

レガシーインポート専用ファイル（まだサポートされていますが、メインストアではありません）：

-   `~/.openclaw/credentials/oauth.json`（初回使用時に`auth-profiles.json`にインポートされます）

上記のすべては`$OPENCLAW_STATE_DIR`（状態ディレクトリのオーバーライド）も尊重します。完全なリファレンス：[/gateway/configuration](../gateway/configuration.md#auth-storage-oauth--api-keys) 静的シークレット参照とランタイムスナップショットのアクティベーション動作については、[シークレット管理](../gateway/secrets.md)を参照してください。

## Anthropic setup-token（サブスクリプション認証）

> **⚠️** Anthropic setup-tokenサポートは技術的な互換性であり、ポリシーの保証ではありません。Anthropicは過去にClaude Code外での一部のサブスクリプション使用をブロックしたことがあります。サブスクリプション認証を使用するかどうかは自身で判断し、Anthropicの現在の利用規約を確認してください。

 任意のマシンで`claude setup-token`を実行し、そのトークンをOpenClawに貼り付けます：

```bash
openclaw models auth setup-token --provider anthropic
```

他の場所でトークンを生成した場合は、手動で貼り付けます：

```bash
openclaw models auth paste-token --provider anthropic
```

確認：

```bash
openclaw models status
```

## OAuth交換（ログインの仕組み）

OpenClawのインタラクティブログインフローは`@mariozechner/pi-ai`に実装され、ウィザード/コマンドに接続されています。

### Anthropic setup-token

フローの概要：

1.  `claude setup-token`を実行
2.  トークンをOpenClawに貼り付け
3.  トークン認証プロファイルとして保存（リフレッシュなし）

ウィザードのパスは`openclaw onboard` → 認証選択`setup-token`（Anthropic）です。

### OpenAI Codex (ChatGPT OAuth)

OpenAI Codex OAuthは、Codex CLI外での使用、OpenClawワークフローを含む使用が明示的にサポートされています。フローの概要（PKCE）：

1.  PKCE検証コード/チャレンジ + ランダムな`state`を生成
2.  `https://auth.openai.com/oauth/authorize?...`を開く
3.  `http://127.0.0.1:1455/auth/callback`でのコールバックをキャプチャしようとする
4.  コールバックがバインドできない場合（またはリモート/ヘッドレスの場合）、リダイレクトURL/コードを貼り付ける
5.  `https://auth.openai.com/oauth/token`で交換
6.  アクセストークンから`accountId`を抽出し、`{ access, refresh, expires, accountId }`を保存

ウィザードのパスは`openclaw onboard` → 認証選択`openai-codex`です。

## リフレッシュ + 有効期限

プロファイルは`expires`タイムスタンプを保存します。ランタイムでは：

-   `expires`が未来の場合 → 保存されたアクセストークンを使用
-   期限切れの場合 → （ファイルロック下で）リフレッシュし、保存された認証情報を上書き

リフレッシュフローは自動的です。通常、トークンを手動で管理する必要はありません。

## 複数アカウント（プロファイル）+ ルーティング

2つのパターン：

### 1) 推奨：別々のエージェント

「個人用」と「仕事用」が決して干渉しないようにしたい場合は、分離されたエージェント（別々のセッション + 認証情報 + ワークスペース）を使用します：

```bash
openclaw agents add work
openclaw agents add personal
```

その後、エージェントごとに認証を設定（ウィザード）し、チャットを適切なエージェントにルーティングします。

### 2) 上級：1つのエージェント内の複数プロファイル

`auth-profiles.json`は、同じプロバイダーに対して複数のプロファイルIDをサポートします。使用するプロファイルを選択します：

-   グローバルに設定の順序（`auth.order`）で
-   セッションごとに`/model ...@`で

例（セッションオーバーライド）：

-   `/model Opus@anthropic:work`

存在するプロファイルIDを確認する方法：

-   `openclaw channels list --json`（`auth[]`を表示）

関連ドキュメント：

-   [/concepts/model-failover](./model-failover.md)（ローテーション + クールダウンルール）
-   [/tools/slash-commands](../tools/slash-commands.md)（コマンドサーフェス）

[エージェントワークスペース](./agent-workspace.md)[ブートストラップ](../start/bootstrapping.md)