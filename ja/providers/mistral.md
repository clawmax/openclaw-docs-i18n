

  プロバイダー

  
# Mistral

OpenClawは、テキスト/画像モデルルーティング (`mistral/...`) と、メディア理解におけるVoxtral経由の音声文字起こしの両方でMistralをサポートしています。Mistralはメモリ埋め込み (`memorySearch.provider = "mistral"`) にも使用できます。

## CLIセットアップ

```bash
openclaw onboard --auth-choice mistral-api-key
# または非対話型
openclaw onboard --mistral-api-key "$MISTRAL_API_KEY"
```

## 設定スニペット (LLMプロバイダー)

```json
{
  env: { MISTRAL_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "mistral/mistral-large-latest" } } },
}
```

## 設定スニペット (Voxtralによる音声文字起こし)

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "mistral", model: "voxtral-mini-latest" }],
      },
    },
  },
}
```

## 注意点

-   Mistral認証には `MISTRAL_API_KEY` を使用します。
-   プロバイダーのベースURLはデフォルトで `https://api.mistral.ai/v1` です。
-   オンボーディングのデフォルトモデルは `mistral/mistral-large-latest` です。
-   Mistralのメディア理解用デフォルト音声モデルは `voxtral-mini-latest` です。
-   メディア文字起こしのパスは `/v1/audio/transcriptions` を使用します。
-   メモリ埋め込みのパスは `/v1/embeddings` を使用します (デフォルトモデル: `mistral-embed`)。

[Moonshot AI](./moonshot.md)[NVIDIA](./nvidia.md)