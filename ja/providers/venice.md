

  プロバイダー

  
# Venice AI

**Venice**は、プライバシーを最優先にした推論と、プロプライエタリモデルへのオプションの匿名化アクセスを実現する、当社のハイライトVeniceセットアップです。Venice AIは、プライバシー重視のAI推論を提供し、検閲なしモデルのサポートと、主要なプロプライエタリモデルへの匿名化プロキシを介したアクセスを可能にします。すべての推論はデフォルトでプライベートです。データのトレーニングやロギングは行われません。

## OpenClawでVeniceを選ぶ理由

-   オープンソースモデルのための**プライベート推論**（ロギングなし）。
-   必要に応じて利用できる**検閲なしモデル**。
-   品質が重要な場合のプロプライエタリモデル（Opus/GPT/Gemini）への**匿名化アクセス**。
-   OpenAI互換の`/v1`エンドポイント。

## プライバシーモード

Veniceは2つのプライバシーレベルを提供します。これを理解することは、モデルを選択する上で重要です：

| モード | 説明 | モデル |
| --- | --- | --- |
| **プライベート** | 完全にプライベート。プロンプト/レスポンスは**決して保存またはログ記録されません**。一時的です。 | Llama, Qwen, DeepSeek, Kimi, MiniMax, Venice Uncensoredなど。 |
| **匿名化** | Veniceを介してプロキシされ、メタデータが除去されます。基盤となるプロバイダー（OpenAI, Anthropic, Google, xAI）には匿名化されたリクエストが見えます。 | Claude, GPT, Gemini, Grok |

## 特徴

-   **プライバシー重視**: 「プライベート」（完全プライベート）と「匿名化」（プロキシ経由）のモードを選択可能
-   **検閲なしモデル**: コンテンツ制限のないモデルへのアクセス
-   **主要モデルへのアクセス**: Veniceの匿名化プロキシを介してClaude、GPT、Gemini、Grokを利用可能
-   **OpenAI互換API**: 簡単な統合のための標準`/v1`エンドポイント
-   **ストリーミング**: ✅ すべてのモデルでサポート
-   **関数呼び出し**: ✅ 選択されたモデルでサポート（モデルの機能を確認）
-   **ビジョン**: ✅ ビジョン機能を持つモデルでサポート
-   **厳しいレート制限なし**: 極端な使用にはフェアユースによるスロットリングが適用される場合があります

## セットアップ

### 1\. APIキーの取得

1.  [venice.ai](https://venice.ai)でサインアップ
2.  **Settings → API Keys → Create new key**に移動
3.  APIキーをコピー（形式: `vapi_xxxxxxxxxxxx`）

### 2\. OpenClawの設定

**オプションA: 環境変数**

```bash
export VENICE_API_KEY="vapi_xxxxxxxxxxxx"
```

**オプションB: インタラクティブセットアップ（推奨）**

```bash
openclaw onboard --auth-choice venice-api-key
```

これにより以下が行われます：

1.  APIキーの入力プロンプト（または既存の`VENICE_API_KEY`を使用）
2.  利用可能なすべてのVeniceモデルの表示
3.  デフォルトモデルの選択
4.  プロバイダーの自動設定

**オプションC: 非インタラクティブ**

```bash
openclaw onboard --non-interactive \
  --auth-choice venice-api-key \
  --venice-api-key "vapi_xxxxxxxxxxxx"
```

### 3\. セットアップの確認

```bash
openclaw agent --model venice/kimi-k2-5 --message "Hello, are you working?"
```

## モデル選択

セットアップ後、OpenClawは利用可能なすべてのVeniceモデルを表示します。ニーズに基づいて選択してください：

-   **デフォルトモデル**: 強力なプライベート推論とビジョンを兼ね備えた`venice/kimi-k2-5`。
-   **高性能オプション**: 最も強力な匿名化Veniceパスである`venice/claude-opus-4-6`。
-   **プライバシー**: 完全にプライベートな推論には「プライベート」モデルを選択。
-   **性能**: Veniceのプロキシを介してClaude、GPT、Geminiにアクセスするには「匿名化」モデルを選択。

デフォルトモデルはいつでも変更できます：

```bash
openclaw models set venice/kimi-k2-5
openclaw models set venice/claude-opus-4-6
```

利用可能なすべてのモデルを一覧表示：

```bash
openclaw models list | grep venice
```

## openclaw configureによる設定

1.  `openclaw configure`を実行
2.  **Model/auth**を選択
3.  **Venice AI**を選択

## どのモデルを使用すべきですか？

| ユースケース | 推奨モデル | 理由 |
| --- | --- | --- |
| **一般的なチャット（デフォルト）** | `kimi-k2-5` | 強力なプライベート推論とビジョン |
| **最高の総合品質** | `claude-opus-4-6` | 最も強力な匿名化Veniceオプション |
| **プライバシー + コーディング** | `qwen3-coder-480b-a35b-instruct` | 大規模コンテキストを持つプライベートコーディングモデル |
| **プライベートビジョン** | `kimi-k2-5` | プライベートモードを離れずにビジョンサポート |
| **高速 + 低コスト** | `qwen3-4b` | 軽量推論モデル |
| **複雑なプライベートタスク** | `deepseek-v3.2` | 強力な推論、ただしVeniceツールサポートなし |
| **検閲なし** | `venice-uncensored` | コンテンツ制限なし |

## 利用可能なモデル（合計41）

### プライベートモデル（26）— 完全プライベート、ロギングなし

| モデルID | 名前 | コンテキスト | 特徴 |
| --- | --- | --- | --- |
| `kimi-k2-5` | Kimi K2.5 | 256k | デフォルト、推論、ビジョン |
| `kimi-k2-thinking` | Kimi K2 Thinking | 256k | 推論 |
| `llama-3.3-70b` | Llama 3.3 70B | 128k | 汎用 |
| `llama-3.2-3b` | Llama 3.2 3B | 128k | 汎用 |
| `hermes-3-llama-3.1-405b` | Hermes 3 Llama 3.1 405B | 128k | 汎用、ツール無効 |
| `qwen3-235b-a22b-thinking-2507` | Qwen3 235B Thinking | 128k | 推論 |
| `qwen3-235b-a22b-instruct-2507` | Qwen3 235B Instruct | 128k | 汎用 |
| `qwen3-coder-480b-a35b-instruct` | Qwen3 Coder 480B | 256k | コーディング |
| `qwen3-coder-480b-a35b-instruct-turbo` | Qwen3 Coder 480B Turbo | 256k | コーディング |
| `qwen3-5-35b-a3b` | Qwen3.5 35B A3B | 256k | 推論、ビジョン |
| `qwen3-next-80b` | Qwen3 Next 80B | 256k | 汎用 |
| `qwen3-vl-235b-a22b` | Qwen3 VL 235B (Vision) | 256k | ビジョン |
| `qwen3-4b` | Venice Small (Qwen3 4B) | 32k | 高速、推論 |
| `deepseek-v3.2` | DeepSeek V3.2 | 160k | 推論、ツール無効 |
| `venice-uncensored` | Venice Uncensored (Dolphin-Mistral) | 32k | 検閲なし、ツール無効 |
| `mistral-31-24b` | Venice Medium (Mistral) | 128k | ビジョン |
| `google-gemma-3-27b-it` | Google Gemma 3 27B Instruct | 198k | ビジョン |
| `openai-gpt-oss-120b` | OpenAI GPT OSS 120B | 128k | 汎用 |
| `nvidia-nemotron-3-nano-30b-a3b` | NVIDIA Nemotron 3 Nano 30B | 128k | 汎用 |
| `olafangensan-glm-4.7-flash-heretic` | GLM 4.7 Flash Heretic | 128k | 推論 |
| `zai-org-glm-4.6` | GLM 4.6 | 198k | 汎用 |
| `zai-org-glm-4.7` | GLM 4.7 | 198k | 推論 |
| `zai-org-glm-4.7-flash` | GLM 4.7 Flash | 128k | 推論 |
| `zai-org-glm-5` | GLM 5 | 198k | 推論 |
| `minimax-m21` | MiniMax M2.1 | 198k | 推論 |
| `minimax-m25` | MiniMax M2.5 | 198k | 推論 |

### 匿名化モデル（15）— Veniceプロキシ経由

| モデルID | 名前 | コンテキスト | 特徴 |
| --- | --- | --- | --- |
| `claude-opus-4-6` | Claude Opus 4.6 (via Venice) | 1M | 推論、ビジョン |
| `claude-opus-4-5` | Claude Opus 4.5 (via Venice) | 198k | 推論、ビジョン |
| `claude-sonnet-4-6` | Claude Sonnet 4.6 (via Venice) | 1M | 推論、ビジョン |
| `claude-sonnet-4-5` | Claude Sonnet 4.5 (via Venice) | 198k | 推論、ビジョン |
| `openai-gpt-54` | GPT-5.4 (via Venice) | 1M | 推論、ビジョン |
| `openai-gpt-53-codex` | GPT-5.3 Codex (via Venice) | 400k | 推論、ビジョン、コーディング |
| `openai-gpt-52` | GPT-5.2 (via Venice) | 256k | 推論 |
| `openai-gpt-52-codex` | GPT-5.2 Codex (via Venice) | 256k | 推論、ビジョン、コーディング |
| `openai-gpt-4o-2024-11-20` | GPT-4o (via Venice) | 128k | ビジョン |
| `openai-gpt-4o-mini-2024-07-18` | GPT-4o Mini (via Venice) | 128k | ビジョン |
| `gemini-3-1-pro-preview` | Gemini 3.1 Pro (via Venice) | 1M | 推論、ビジョン |
| `gemini-3-pro-preview` | Gemini 3 Pro (via Venice) | 198k | 推論、ビジョン |
| `gemini-3-flash-preview` | Gemini 3 Flash (via Venice) | 256k | 推論、ビジョン |
| `grok-41-fast` | Grok 4.1 Fast (via Venice) | 1M | 推論、ビジョン |
| `grok-code-fast-1` | Grok Code Fast 1 (via Venice) | 256k | 推論、コーディング |

## モデルディスカバリー

OpenClawは、`VENICE_API_KEY`が設定されている場合、Venice APIからモデルを自動的に検出します。APIに到達できない場合、静的カタログにフォールバックします。`/models`エンドポイントは公開されています（一覧表示に認証は不要）が、推論には有効なAPIキーが必要です。

## ストリーミングとツールサポート

| 機能 | サポート |
| --- | --- |
| **ストリーミング** | ✅ すべてのモデル |
| **関数呼び出し** | ✅ ほとんどのモデル（APIの`supportsFunctionCalling`を確認） |
| **ビジョン/画像** | ✅ 「ビジョン」機能がマークされたモデル |
| **JSONモード** | ✅ `response_format`を介してサポート |

## 料金

Veniceはクレジットベースのシステムを使用しています。最新の料金は[venice.ai/pricing](https://venice.ai/pricing)で確認してください：

-   **プライベートモデル**: 一般的に低コスト
-   **匿名化モデル**: 直接APIの料金 + わずかなVenice手数料に類似

## 比較: Venice vs 直接API

| 側面 | Venice（匿名化） | 直接API |
| --- | --- | --- |
| **プライバシー** | メタデータ除去、匿名化 | アカウントに紐付け |
| **レイテンシ** | +10-50ms（プロキシ） | 直接 |
| **機能** | ほとんどの機能をサポート | 全機能 |
| **課金** | Veniceクレジット | プロバイダー課金 |

## 使用例

```bash
# デフォルトのプライベートモデルを使用
openclaw agent --model venice/kimi-k2-5 --message "Quick health check"

# Venice経由でClaude Opusを使用（匿名化）
openclaw agent --model venice/claude-opus-4-6 --message "Summarize this task"

# 検閲なしモデルを使用
openclaw agent --model venice/venice-uncensored --message "Draft options"

# 画像付きビジョンモデルを使用
openclaw agent --model venice/qwen3-vl-235b-a22b --message "Review attached image"

# コーディングモデルを使用
openclaw agent --model venice/qwen3-coder-480b-a35b-instruct --message "Refactor this function"
```

## トラブルシューティング

### APIキーが認識されない

```bash
echo $VENICE_API_KEY
openclaw models list | grep venice
```

キーが`vapi_`で始まっていることを確認してください。

### モデルが利用できない

Veniceモデルカタログは動的に更新されます。現在利用可能なモデルを確認するには`openclaw models list`を実行してください。一部のモデルは一時的にオフラインの場合があります。

### 接続の問題

Venice APIは`https://api.venice.ai/api/v1`にあります。ネットワークがHTTPS接続を許可していることを確認してください。

## 設定ファイルの例

```json
{
  env: { VENICE_API_KEY: "vapi_..." },
  agents: { defaults: { model: { primary: "venice/kimi-k2-5" } } },
  models: {
    mode: "merge",
    providers: {
      venice: {
        baseUrl: "https://api.venice.ai/api/v1",
        apiKey: "${VENICE_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-k2-5",
            name: "Kimi K2.5",
            reasoning: true,
            input: ["text", "image"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 65536,
          },
        ],
      },
    },
  },
}
```

## リンク

-   [Venice AI](https://venice.ai)
-   [APIドキュメント](https://docs.venice.ai)
-   [料金](https://venice.ai/pricing)
-   [ステータス](https://status.venice.ai)

[Vercel AI Gateway](./vercel-ai-gateway.md)[vLLM](./vllm.md)