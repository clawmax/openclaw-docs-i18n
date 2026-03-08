title: "OpenClaw Kilocode プロバイダー設定：APIキーとモデル"
description: "OpenClawでKilocodeプロバイダーを設定し、APIキーを取得し、統合されたKilo Gatewayを使用して複数のAIモデルにアクセスする方法を学びます。"
keywords: ["kilocode", "kilo gateway", "openclaw プロバイダー", "統合api", "モデルルーティング", "apiキー設定", "claude sonnet", "gpt-5.2"]
---

  プロバイダー

  
# Kilocode

Kilo Gatewayは、**統合API**を提供し、単一のエンドポイントとAPIキーの背後で多くのモデルにリクエストをルーティングします。OpenAI互換なので、ベースURLを切り替えるだけでほとんどのOpenAI SDKが動作します。

## APIキーの取得

1.  [app.kilo.ai](https://app.kilo.ai) にアクセスします
2.  サインインまたはアカウントを作成します
3.  APIキーに移動し、新しいキーを生成します

## CLI設定

```bash
openclaw onboard --kilocode-api-key <key>
```

または環境変数を設定します:

```bash
export KILOCODE_API_KEY="<your-kilocode-api-key>" # pragma: allowlist secret
```

## 設定スニペット

```json
{
  env: { KILOCODE_API_KEY: "<your-kilocode-api-key>" }, // pragma: allowlist secret
  agents: {
    defaults: {
      model: { primary: "kilocode/kilo/auto" },
    },
  },
}
```

## デフォルトモデル

デフォルトモデルは `kilocode/kilo/auto` です。これは、タスクに基づいて最適な基盤モデルを自動的に選択するスマートルーティングモデルです:

-   計画、デバッグ、オーケストレーションタスクはClaude Opusにルーティングされます
-   コード記述および探索タスクはClaude Sonnetにルーティングされます

## 利用可能なモデル

OpenClawは起動時にKilo Gatewayから利用可能なモデルを動的に検出します。アカウントで利用可能なモデルの完全なリストを表示するには `/models kilocode` を使用してください。ゲートウェイで利用可能なモデルはすべて `kilocode/` プレフィックスで使用できます:

```
kilocode/kilo/auto              (デフォルト - スマートルーティング)
kilocode/anthropic/claude-sonnet-4
kilocode/openai/gpt-5.2
kilocode/google/gemini-3-pro-preview
...その他多数
```

## 注意点

-   モデル参照は `kilocode/<model-id>` の形式です (例: `kilocode/anthropic/claude-sonnet-4`)。
-   デフォルトモデル: `kilocode/kilo/auto`
-   ベースURL: `https://api.kilo.ai/api/gateway/`
-   その他のモデル/プロバイダーオプションについては、[/concepts/model-providers](../concepts/model-providers.md) を参照してください。
-   Kilo Gatewayは、内部でBearerトークンとあなたのAPIキーを使用します。

[Hugging Face (推論)](./huggingface.md)[Litellm](./litellm.md)