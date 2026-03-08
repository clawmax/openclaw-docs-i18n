

  プロバイダー

  
# Together

[Together AI](https://together.ai) は、統一されたAPIを通じて、Llama、DeepSeek、Kimiなどを含む主要なオープンソースモデルへのアクセスを提供します。

-   プロバイダー: `together`
-   認証: `TOGETHER_API_KEY`
-   API: OpenAI互換

## クイックスタート

1.  APIキーを設定します（推奨: Gateway用に保存）:

```bash
openclaw onboard --auth-choice together-api-key
```

2.  デフォルトモデルを設定します:

```json
{
  agents: {
    defaults: {
      model: { primary: "together/moonshotai/Kimi-K2.5" },
    },
  },
}
```

## 非対話型の例

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice together-api-key \
  --together-api-key "$TOGETHER_API_KEY"
```

これにより、`together/moonshotai/Kimi-K2.5`がデフォルトモデルとして設定されます。

## 環境に関する注意

Gatewayがデーモン（launchd/systemd）として実行される場合、`TOGETHER_API_KEY`がそのプロセス（例: `~/.openclaw/.env` または `env.shellEnv`経由）で利用可能であることを確認してください。

## 利用可能なモデル

Together AIは、多くの人気のあるオープンソースモデルへのアクセスを提供します:

-   **GLM 4.7 Fp8** - 200Kコンテキストウィンドウを備えたデフォルトモデル
-   **Llama 3.3 70B Instruct Turbo** - 高速で効率的な指示追従
-   **Llama 4 Scout** - 画像理解機能を備えたビジョンモデル
-   **Llama 4 Maverick** - 高度なビジョンと推論
-   **DeepSeek V3.1** - 強力なコーディングと推論モデル
-   **DeepSeek R1** - 高度な推論モデル
-   **Kimi K2 Instruct** - 262Kコンテキストウィンドウを備えた高性能モデル

すべてのモデルは標準的なチャット補完をサポートし、OpenAI APIと互換性があります。

[Synthetic](./synthetic.md)[Vercel AI Gateway](./vercel-ai-gateway.md)