

  ガイド

  
# CLI自動化

`openclaw onboard` を自動化するには `--non-interactive` を使用します。

> **ℹ️** `--json` は非対話型モードを意味しません。スクリプトでは `--non-interactive` (および `--workspace`) を使用してください。

## 基本的な非対話型の例

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --secret-input-mode plaintext \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

機械可読なサマリーには `--json` を追加します。プレーンテキスト値の代わりに環境変数に基づく参照を認証プロファイルに保存するには `--secret-input-mode ref` を使用します。環境変数参照と構成済みプロバイダー参照 (`file` または `exec`) の対話型選択は、オンボーディングウィザードフローで利用可能です。非対話型の `ref` モードでは、プロバイダーの環境変数がプロセス環境に設定されている必要があります。一致する環境変数なしでインラインキーフラグを渡すと、すぐに失敗します。例:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice openai-api-key \
  --secret-input-mode ref \
  --accept-risk
```

## プロバイダー固有の例

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice gemini-api-key \
  --gemini-api-key "$GEMINI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice zai-api-key \
  --zai-api-key "$ZAI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice cloudflare-ai-gateway-api-key \
  --cloudflare-ai-gateway-account-id "your-account-id" \
  --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
  --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice moonshot-api-key \
  --moonshot-api-key "$MOONSHOT_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice mistral-api-key \
  --mistral-api-key "$MISTRAL_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice synthetic-api-key \
  --synthetic-api-key "$SYNTHETIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice opencode-zen \
  --opencode-zen-api-key "$OPENCODE_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --custom-provider-id "my-custom" \
  --custom-compatibility anthropic \
  --gateway-port 18789 \
  --gateway-bind loopback
```

`--custom-api-key` はオプションです。省略した場合、オンボーディングは `CUSTOM_API_KEY` をチェックします。Refモードのバリアント:

```bash
export CUSTOM_API_KEY="your-key"
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --secret-input-mode ref \
  --custom-provider-id "my-custom" \
  --custom-compatibility anthropic \
  --gateway-port 18789 \
  --gateway-bind loopback
```

このモードでは、オンボーディングは `apiKey` を `{ source: "env", provider: "default", id: "CUSTOM_API_KEY" }` として保存します。

## 別のエージェントを追加する

独自のワークスペース、セッション、認証プロファイルを持つ別のエージェントを作成するには、`openclaw agents add ` を使用します。`--workspace` なしで実行するとウィザードが起動します。

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

設定される内容:

-   `agents.list[].name`
-   `agents.list[].workspace`
-   `agents.list[].agentDir`

注意点:

-   デフォルトのワークスペースは `~/.openclaw/workspace-` に従います。
-   インバウンドメッセージをルーティングするには `bindings` を追加します（ウィザードでこれを行うことができます）。
-   非対話型フラグ: `--model`, `--agent-dir`, `--bind`, `--non-interactive`.

## 関連ドキュメント

-   オンボーディングハブ: [オンボーディングウィザード (CLI)](./wizard.md)
-   完全なリファレンス: [CLIオンボーディングリファレンス](./wizard-cli-reference.md)
-   コマンドリファレンス: [`openclaw onboard`](../cli/onboard.md)

[CLIリファレンス](./wizard-cli-reference.md)