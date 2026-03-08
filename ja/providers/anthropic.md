

  プロバイダー

  
# Anthropic

Anthropicは**Claude**モデルファミリーを構築し、API経由でアクセスを提供しています。OpenClawでは、APIキーまたは**setup-token**を使用して認証できます。

## オプションA: Anthropic APIキー

**最適な用途:** 標準的なAPIアクセスと使用量ベースの課金。AnthropicコンソールでAPIキーを作成します。

### CLIセットアップ

```bash
openclaw onboard
# 選択: Anthropic API key

# または非対話型
openclaw onboard --anthropic-api-key "$ANTHROPIC_API_KEY"
```

### 設定スニペット

```json
{
  env: { ANTHROPIC_API_KEY: "sk-ant-..." },
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## 思考のデフォルト (Claude 4.6)

-   Anthropic Claude 4.6モデルは、明示的な思考レベルが設定されていない場合、OpenClawではデフォルトで`adaptive`思考を使用します。
-   メッセージごとに上書き（`/think:<レベル>`）するか、モデルパラメータで上書きできます: `agents.defaults.models["anthropic/<モデル>"].params.thinking`。
-   関連するAnthropicドキュメント:
    -   [適応的思考](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking)
    -   [拡張思考](https://platform.claude.com/docs/en/build-with-claude/extended-thinking)

## プロンプトキャッシング (Anthropic API)

OpenClawはAnthropicのプロンプトキャッシング機能をサポートしています。これは**API専用**です。サブスクリプション認証ではキャッシュ設定は適用されません。

### 設定

モデル設定で`cacheRetention`パラメータを使用します:

| 値 | キャッシュ期間 | 説明 |
| --- | --- | --- |
| `none` | キャッシングなし | プロンプトキャッシングを無効化 |
| `short` | 5分 | APIキー認証のデフォルト |
| `long` | 1時間 | 拡張キャッシュ (ベータフラグが必要) |

```json
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": {
          params: { cacheRetention: "long" },
        },
      },
    },
  },
}
```

### デフォルト

Anthropic APIキー認証を使用する場合、OpenClawはすべてのAnthropicモデルに対して自動的に`cacheRetention: "short"` (5分キャッシュ) を適用します。設定で明示的に`cacheRetention`を設定することでこれを上書きできます。

### エージェントごとのcacheRetention上書き

モデルレベルのパラメータをベースラインとして使用し、`agents.list[].params`を介して特定のエージェントを上書きします。

```json
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-opus-4-6" },
      models: {
        "anthropic/claude-opus-4-6": {
          params: { cacheRetention: "long" }, // ほとんどのエージェントのベースライン
        },
      },
    },
    list: [
      { id: "research", default: true },
      { id: "alerts", params: { cacheRetention: "none" } }, // このエージェントのみ上書き
    ],
  },
}
```

キャッシュ関連パラメータの設定マージ順序:

1.  `agents.defaults.models["provider/model"].params`
2.  `agents.list[].params` (一致する`id`、キーごとに上書き)

これにより、1つのエージェントが長期間のキャッシュを保持し、同じモデル上の別のエージェントはバースト的/再利用率の低いトラフィックでの書き込みコストを避けるためにキャッシングを無効にできます。

### Bedrock Claudeに関する注意点

-   Bedrock上のAnthropic Claudeモデル (`amazon-bedrock/*anthropic.claude*`) は、設定時に`cacheRetention`のパススルーを受け入れます。
-   Anthropic以外のBedrockモデルは、実行時に`cacheRetention: "none"`に強制されます。
-   Anthropic APIキーのスマートデフォルトは、明示的な値が設定されていない場合、Claude-on-Bedrockモデル参照に対しても`cacheRetention: "short"`をシードします。

### レガシーパラメータ

古い`cacheControlTtl`パラメータは、下位互換性のために引き続きサポートされています:

-   `"5m"` は `short` にマッピング
-   `"1h"` は `long` にマッピング

新しい`cacheRetention`パラメータへの移行を推奨します。OpenClawにはAnthropic APIリクエスト用の`extended-cache-ttl-2025-04-11`ベータフラグが含まれています。プロバイダーヘッダーを上書きする場合は保持してください ([/gateway/configuration](../gateway/configuration.md)を参照)。

## 100万トークンコンテキストウィンドウ (Anthropic ベータ)

Anthropicの100万トークンコンテキストウィンドウはベータゲートされています。OpenClawでは、サポートされているOpus/Sonnetモデルに対して`params.context1m: true`を設定することで、モデルごとに有効化できます。

```json
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": {
          params: { context1m: true },
        },
      },
    },
  },
}
```

OpenClawはこれをAnthropicリクエスト上の`anthropic-beta: context-1m-2025-08-07`にマッピングします。これは、そのモデルに対して`params.context1m`が明示的に`true`に設定されている場合にのみ有効化されます。要件: Anthropicはその認証情報に対して長いコンテキストの使用を許可する必要があります (通常はAPIキー課金、またはExtra Usageが有効なサブスクリプションアカウント)。そうでない場合、Anthropicは`HTTP 429: rate_limit_error: Extra usage is required for long context requests`を返します。注: Anthropicは現在、OAuth/サブスクリプショントークン (`sk-ant-oat-*`) を使用する場合、`context-1m-*`ベータリクエストを拒否します。OpenClawはOAuth認証に対してcontext1mベータヘッダーを自動的にスキップし、必要なOAuthベータを保持します。

## オプションB: Claude setup-token

**最適な用途:** Claudeサブスクリプションの使用。

### setup-tokenの入手方法

Setup-tokenは**Anthropicコンソールではなく**、**Claude Code CLI**によって作成されます。これは**任意のマシン**で実行できます:

```bash
claude setup-token
```

トークンをOpenClawに貼り付けます (ウィザード: **Anthropic token (paste setup-token)**)、またはゲートウェイホストで実行します:

```bash
openclaw models auth setup-token --provider anthropic
```

別のマシンでトークンを生成した場合は、貼り付けます:

```bash
openclaw models auth paste-token --provider anthropic
```

### CLIセットアップ (setup-token)

```bash
# オンボーディング中にsetup-tokenを貼り付け
openclaw onboard --auth-choice setup-token
```

### 設定スニペット (setup-token)

```json
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## 注意点

-   `claude setup-token`でsetup-tokenを生成して貼り付けるか、ゲートウェイホストで`openclaw models auth setup-token`を実行します。
-   Claudeサブスクリプションで「OAuth token refresh failed …」が表示される場合は、setup-tokenで再認証してください。[/gateway/troubleshooting#oauth-token-refresh-failed-anthropic-claude-subscription](../gateway/troubleshooting.md#oauth-token-refresh-failed-anthropic-claude-subscription)を参照。
-   認証の詳細と再利用ルールは[/concepts/oauth](../concepts/oauth.md)にあります。

## トラブルシューティング

**401エラー / トークンが突然無効になる**

-   Claudeサブスクリプション認証は期限切れまたは取り消される可能性があります。`claude setup-token`を再実行し、**ゲートウェイホスト**に貼り付けます。
-   Claude CLIログインが別のマシンにある場合は、ゲートウェイホストで`openclaw models auth paste-token --provider anthropic`を使用します。

**プロバイダー「anthropic」のAPIキーが見つかりません**

-   認証は**エージェントごと**です。新しいエージェントはメインエージェントのキーを継承しません。
-   そのエージェントのオンボーディングを再実行するか、ゲートウェイホストにsetup-token / APIキーを貼り付け、`openclaw models status`で確認します。

**プロファイル`anthropic:default`の認証情報が見つかりません**

-   `openclaw models status`を実行して、どの認証プロファイルがアクティブか確認します。
-   オンボーディングを再実行するか、そのプロファイルのsetup-token / APIキーを貼り付けます。

**利用可能な認証プロファイルがありません (すべてクールダウン中/利用不可)**

-   `openclaw models status --json`で`auth.unusableProfiles`を確認します。
-   別のAnthropicプロファイルを追加するか、クールダウンを待ちます。

詳細: [/gateway/troubleshooting](../gateway/troubleshooting.md) および [/help/faq](../help/faq.md)。

[モデルフェイルオーバー](../concepts/model-failover.md)[Amazon Bedrock](./bedrock.md)