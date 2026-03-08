

  メディアとデバイス

  
# 音声とボイスノート

## 対応機能

-   **メディア理解 (音声)**: 音声理解が有効（または自動検出）されている場合、OpenClaw は以下の処理を行います:
    1.  最初の音声添付ファイル（ローカルパスまたは URL）を特定し、必要に応じてダウンロードします。
    2.  各モデルエントリに送信する前に `maxBytes` を適用します。
    3.  順序に従って最初の適格なモデルエントリ（プロバイダーまたは CLI）を実行します。
    4.  失敗またはスキップ（サイズ/タイムアウト）した場合、次のエントリを試行します。
    5.  成功すると、`Body` を `[Audio]` ブロックに置き換え、`{{Transcript}}` を設定します。
-   **コマンド解析**: 文字起こしが成功すると、`CommandBody`/`RawBody` はトランスクリプトに設定されるため、スラッシュコマンドが引き続き機能します。
-   **詳細ログ**: `--verbose` モードでは、文字起こしが実行された時点と本文が置き換えられた時点をログに記録します。

## 自動検出 (デフォルト)

**モデルを設定せず**、かつ `tools.media.audio.enabled` が `false` に**設定されていない**場合、OpenClaw は以下の順序で自動検出を行い、最初に動作するオプションで停止します:

1.  **ローカル CLI** (インストール済みの場合)
    -   `sherpa-onnx-offline` (`SHERPA_ONNX_MODEL_DIR` にエンコーダー/デコーダー/ジョイナー/トークンが必要)
    -   `whisper-cli` (`whisper-cpp` から; `WHISPER_CPP_MODEL` またはバンドルされた tiny モデルを使用)
    -   `whisper` (Python CLI; モデルを自動ダウンロード)
2.  **Gemini CLI** (`gemini`) `read_many_files` を使用
3.  **プロバイダーキー** (OpenAI → Groq → Deepgram → Google)

自動検出を無効にするには、`tools.media.audio.enabled: false` を設定します。カスタマイズするには、`tools.media.audio.models` を設定します。注: バイナリ検出は macOS/Linux/Windows 間でベストエフォートです。CLI が `PATH` 上にあること（`~` を展開します）、または完全なコマンドパスで明示的な CLI モデルを設定することを確認してください。

## 設定例

### プロバイダー + CLI フォールバック (OpenAI + Whisper CLI)

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        maxBytes: 20971520,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
            timeoutSeconds: 45,
          },
        ],
      },
    },
  },
}
```

### スコープ制御付きプロバイダーのみ

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        scope: {
          default: "allow",
          rules: [{ action: "deny", match: { chatType: "group" } }],
        },
        models: [{ provider: "openai", model: "gpt-4o-mini-transcribe" }],
      },
    },
  },
}
```

### プロバイダーのみ (Deepgram)

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

### プロバイダーのみ (Mistral Voxtral)

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

### トランスクリプトをチャットにエコー (オプトイン)

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        echoTranscript: true, // デフォルトは false
        echoFormat: '📝 "{transcript}"', // オプション、{transcript} をサポート
        models: [{ provider: "openai", model: "gpt-4o-mini-transcribe" }],
      },
    },
  },
}
```

## 注意点と制限

-   プロバイダー認証は標準のモデル認証順序に従います（認証プロファイル、環境変数、`models.providers.*.apiKey`）。
-   Deepgram は `provider: "deepgram"` が使用されるときに `DEEPGRAM_API_KEY` を取得します。
-   Deepgram のセットアップ詳細: [Deepgram (音声文字起こし)](../providers/deepgram.md)。
-   Mistral のセットアップ詳細: [Mistral](../providers/mistral.md)。
-   音声プロバイダーは `tools.media.audio` 経由で `baseUrl`、`headers`、`providerOptions` をオーバーライドできます。
-   デフォルトのサイズ上限は 20MB (`tools.media.audio.maxBytes`) です。サイズ超過の音声はそのモデルではスキップされ、次のエントリが試行されます。
-   1024 バイト未満の小さな/空の音声ファイルは、プロバイダー/CLI による文字起こしの前にスキップされます。
-   音声のデフォルト `maxChars` は**未設定**（完全なトランスクリプト）です。出力を切り詰めるには `tools.media.audio.maxChars` またはエントリごとの `maxChars` を設定します。
-   OpenAI の自動デフォルトは `gpt-4o-mini-transcribe` です。より高い精度が必要な場合は `model: "gpt-4o-transcribe"` を設定します。
-   複数のボイスノートを処理するには `tools.media.audio.attachments` を使用します (`mode: "all"` + `maxAttachments`)。
-   トランスクリプトはテンプレート内で `{{Transcript}}` として利用可能です。
-   `tools.media.audio.echoTranscript` はデフォルトでオフです。エージェント処理の前に文字起こし確認を元のチャットに送信するには有効にします。
-   `tools.media.audio.echoFormat` はエコーテキストをカスタマイズします（プレースホルダー: `{transcript}`）。
-   CLI の標準出力は上限があります（5MB）。CLI の出力は簡潔に保ってください。

### プロキシ環境サポート

プロバイダーベースの音声文字起こしは、標準のアウトバウンドプロキシ環境変数を尊重します:

-   `HTTPS_PROXY`
-   `HTTP_PROXY`
-   `https_proxy`
-   `http_proxy`

プロキシ環境変数が設定されていない場合、直接のエグレスが使用されます。プロキシ設定が不正な場合、OpenClaw は警告をログに記録し、直接フェッチにフォールバックします。

## グループ内でのメンション検出

グループチャットで `requireMention: true` が設定されている場合、OpenClaw はメンションをチェックする**前に**音声を文字起こしするようになりました。これにより、メンションを含むボイスノートも処理できるようになります。**仕組み:**

1.  ボイスメッセージにテキスト本文がなく、グループがメンションを必要とする場合、OpenClaw は「プレフライト」文字起こしを実行します。
2.  トランスクリプトがメンションパターン（例: `@BotName`、絵文字トリガー）についてチェックされます。
3.  メンションが見つかった場合、メッセージは完全な返信パイプラインを通じて処理されます。
4.  トランスクリプトはメンション検出に使用されるため、ボイスノートがメンションゲートを通過できます。

**フォールバック動作:**

-   プレフライト中に文字起こしが失敗した場合（タイムアウト、API エラーなど）、メッセージはテキストのみのメンション検出に基づいて処理されます。
-   これにより、混合メッセージ（テキスト + 音声）が誤ってドロップされることがありません。

**Telegram グループ/トピックごとのオプトアウト:**

-   `channels.telegram.groups..disableAudioPreflight: true` を設定すると、そのグループのプレフライト文字起こしメンションチェックをスキップします。
-   `channels.telegram.groups..topics..disableAudioPreflight` を設定すると、トピックごとにオーバーライドします（スキップするには `true`、強制有効化には `false`）。
-   デフォルトは `false` です（メンションゲート条件に一致する場合、プレフライトが有効）。

**例:** ユーザーが `requireMention: true` が設定された Telegram グループで「ねえ @Claude、天気はどう？」と言うボイスノートを送信します。ボイスノートは文字起こしされ、メンションが検出され、エージェントが返信します。

## 注意すべき点

-   スコープルールは最初に一致したものが優先されます。`chatType` は `direct`、`group`、`room` に正規化されます。
-   CLI が終了コード 0 で終了し、プレーンテキストを出力することを確認してください。JSON の場合は `jq -r .text` などを介して整形する必要があります。
-   `parakeet-mlx` の場合、`--output-dir` を渡すと、OpenClaw は `--output-format` が `txt`（または省略）の場合に `<output-dir>/<media-basename>.txt` を読み取ります。`txt` 以外の出力形式の場合は、標準出力の解析にフォールバックします。
-   返信キューをブロックしないように、タイムアウトは適切に設定してください（`timeoutSeconds`、デフォルト 60 秒）。
-   プレフライト文字起こしは、メンション検出のために**最初の**音声添付ファイルのみを処理します。追加の音声はメインのメディア理解フェーズで処理されます。

[画像とメディアサポート](./images.md)[カメラキャプチャ](./camera.md)