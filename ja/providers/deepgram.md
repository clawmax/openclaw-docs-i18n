

  プロバイダー

  
# Deepgram

Deepgramは音声認識APIです。OpenClawでは、`tools.media.audio`を介した**インバウンド音声/ボイスノートの文字起こし**に使用されます。有効にすると、OpenClawは音声ファイルをDeepgramにアップロードし、文字起こし結果を返信パイプラインに注入します（`{{Transcript}}` + `[Audio]`ブロック）。これは**ストリーミングではありません**。事前録音された文字起こしエンドポイントを使用します。ウェブサイト: [https://deepgram.com](https://deepgram.com)  
ドキュメント: [https://developers.deepgram.com](https://developers.deepgram.com)

## クイックスタート

1.  APIキーを設定します:

```
DEEPGRAM_API_KEY=dg_...
```

2.  プロバイダーを有効にします:

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "deepgram", model: "nova-3" }],
      },
    },
  },
}
```

## オプション

-   `model`: DeepgramモデルID（デフォルト: `nova-3`）
-   `language`: 言語ヒント（オプション）
-   `tools.media.audio.providerOptions.deepgram.detect_language`: 言語検出を有効化（オプション）
-   `tools.media.audio.providerOptions.deepgram.punctuate`: 句読点を有効化（オプション）
-   `tools.media.audio.providerOptions.deepgram.smart_format`: スマートフォーマットを有効化（オプション）

言語を指定した例:

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "deepgram", model: "nova-3", language: "en" }],
      },
    },
  },
}
```

Deepgramオプションを指定した例:

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        providerOptions: {
          deepgram: {
            detect_language: true,
            punctuate: true,
            smart_format: true,
          },
        },
        models: [{ provider: "deepgram", model: "nova-3" }],
      },
    },
  },
}
```

## 注意事項

-   認証は標準的なプロバイダー認証順序に従います。`DEEPGRAM_API_KEY`が最もシンプルな方法です。
-   プロキシを使用する場合は、`tools.media.audio.baseUrl`と`tools.media.audio.headers`でエンドポイントやヘッダーをオーバーライドできます。
-   出力は他のプロバイダーと同じ音声ルールに従います（サイズ制限、タイムアウト、文字起こし結果の注入）。

[Claude Max API Proxy](./claude-max-api-proxy.md)[GitHub Copilot](./github-copilot.md)