

  CLI コマンド

  
# onboard

インタラクティブなオンボーディングウィザード（ローカルまたはリモートゲートウェイ設定）。

## 関連ガイド

-   CLI オンボーディングハブ: [オンボーディングウィザード (CLI)](../start/wizard.md)
-   オンボーディング概要: [オンボーディング概要](../start/onboarding-overview.md)
-   CLI オンボーディングリファレンス: [CLI オンボーディングリファレンス](../start/wizard-cli-reference.md)
-   CLI 自動化: [CLI 自動化](../start/wizard-cli-automation.md)
-   macOS オンボーディング: [オンボーディング (macOS アプリ)](../start/onboarding.md)

## 例

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url wss://gateway-host:18789
```

プレーンテキストのプライベートネットワーク `ws://` ターゲット（信頼されたネットワークのみ）の場合、オンボーディングプロセスの環境で `OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1` を設定します。非対話型カスタムプロバイダー:

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --secret-input-mode plaintext \
  --custom-compatibility openai
```

`--custom-api-key` は非対話型モードではオプションです。省略された場合、オンボーディングは `CUSTOM_API_KEY` をチェックします。プロバイダーキーをプレーンテキストではなく参照として保存:

```bash
openclaw onboard --non-interactive \
  --auth-choice openai-api-key \
  --secret-input-mode ref \
  --accept-risk
```

`--secret-input-mode ref` を使用すると、オンボーディングはプレーンテキストのキー値の代わりに環境変数に基づく参照を書き込みます。auth-profile ベースのプロバイダーの場合、これは `keyRef` エントリを書き込みます。カスタムプロバイダーの場合、これは `models.providers..apiKey` を環境参照として書き込みます（例: `{ source: "env", provider: "default", id: "CUSTOM_API_KEY" }`）。非対話型 `ref` モードの契約:

-   プロバイダーの環境変数をオンボーディングプロセスの環境に設定します（例: `OPENAI_API_KEY`）。
-   その環境変数も設定されていない限り、インラインキーフラグ（例: `--openai-api-key`）を渡さないでください。
-   必要な環境変数なしにインラインキーフラグが渡された場合、オンボーディングはガイダンスとともに早期に失敗します。

非対話型モードでのゲートウェイトークンオプション:

-   `--gateway-auth token --gateway-token ` はプレーンテキストトークンを保存します。
-   `--gateway-auth token --gateway-token-ref-env ` は `gateway.auth.token` を環境 SecretRef として保存します。
-   `--gateway-token` と `--gateway-token-ref-env` は相互排他的です。
-   `--gateway-token-ref-env` は、オンボーディングプロセス環境内の空でない環境変数を必要とします。
-   `--install-daemon` を使用する場合、トークン認証がトークンを必要とするとき、SecretRef で管理されるゲートウェイトークンは検証されますが、スーパーバイザーサービスの環境メタデータ内で解決されたプレーンテキストとして永続化されません。
-   `--install-daemon` を使用する場合、トークンモードがトークンを必要とし、設定されたトークン SecretRef が未解決の場合、オンボーディングは修正ガイダンスとともに閉じて失敗します。
-   `--install-daemon` を使用する場合、`gateway.auth.token` と `gateway.auth.password` の両方が設定され、`gateway.auth.mode` が未設定の場合、オンボーディングはモードが明示的に設定されるまでインストールをブロックします。

例:

```bash
export OPENCLAW_GATEWAY_TOKEN="your-token"
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice skip \
  --gateway-auth token \
  --gateway-token-ref-env OPENCLAW_GATEWAY_TOKEN \
  --accept-risk
```

参照モードでのインタラクティブオンボーディングの動作:

-   プロンプトが表示されたら **シークレット参照を使用** を選択します。
-   次に、以下のいずれかを選択します:
    -   環境変数
    -   設定済みシークレットプロバイダー (`file` または `exec`)
-   オンボーディングは参照を保存する前に高速な事前検証を実行します。
    -   検証が失敗した場合、オンボーディングはエラーを表示し、再試行を許可します。

非対話型 Z.AI エンドポイント選択: 注: `--auth-choice zai-api-key` は現在、あなたのキーに最適な Z.AI エンドポイントを自動検出します（一般的な API と `zai/glm-5` を優先します）。GLM Coding Plan エンドポイントを特に希望する場合は、`zai-coding-global` または `zai-coding-cn` を選択してください。

```bash
# プロンプトなしのエンドポイント選択
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# その他の Z.AI エンドポイント選択肢:
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

非対話型 Mistral の例:

```bash
openclaw onboard --non-interactive \
  --auth-choice mistral-api-key \
  --mistral-api-key "$MISTRAL_API_KEY"
```

フローの注意点:

-   `quickstart`: 最小限のプロンプト、ゲートウェイトークンを自動生成。
-   `manual`: ポート/バインド/認証に関する完全なプロンプト (`advanced` のエイリアス)。
-   ローカルオンボーディング DM スコープの動作: [CLI オンボーディングリファレンス](../start/wizard-cli-reference.md#outputs-and-internals)。
-   最速の最初のチャット: `openclaw dashboard` (Control UI、チャネル設定不要)。
-   カスタムプロバイダー: リストにないホスト型プロバイダーを含む、任意の OpenAI または Anthropic 互換エンドポイントに接続。自動検出には Unknown を使用。

## 一般的なフォローアップコマンド

```bash
openclaw configure
openclaw agents add <name>
```

> **ℹ️** `--json` は非対話型モードを意味しません。スクリプトには `--non-interactive` を使用してください。

[nodes](./nodes.md)[pairing](./pairing.md)

---