

  メディアとデバイス

  
# Text-to-Speech

OpenClawは、送信する返信をElevenLabs、OpenAI、またはEdge TTSを使用して音声に変換できます。OpenClawが音声を送信できる場所であればどこでも動作し、Telegramでは丸い音声メッセージのバブルとして表示されます。

## サポートされているサービス

-   **ElevenLabs** (プライマリまたはフォールバックプロバイダー)
-   **OpenAI** (プライマリまたはフォールバックプロバイダー、要約にも使用)
-   **Edge TTS** (プライマリまたはフォールバックプロバイダー、`node-edge-tts`を使用、APIキーがない場合のデフォルト)

### Edge TTSに関する注意

Edge TTSは、`node-edge-tts`ライブラリを介してMicrosoft Edgeのオンライン神経TTSサービスを使用します。これはホスト型サービス（ローカルではありません）で、Microsoftのエンドポイントを使用し、APIキーは必要ありません。`node-edge-tts`は音声設定オプションと出力フォーマットを公開しますが、すべてのオプションがEdgeサービスでサポートされているわけではありません。citeturn2search0 Edge TTSは公開されたSLAやクォータのない公開ウェブサービスであるため、ベストエフォートとして扱ってください。保証された制限とサポートが必要な場合は、OpenAIまたはElevenLabsを使用してください。MicrosoftのSpeech REST APIはリクエストごとに10分の音声制限を文書化しています。Edge TTSは制限を公開していないため、同様またはそれ以下の制限があると想定してください。citeturn0search3

## オプションのキー

OpenAIまたはElevenLabsを使用したい場合:

-   `ELEVENLABS_API_KEY` (または `XI_API_KEY`)
-   `OPENAI_API_KEY`

Edge TTSはAPIキーを**必要としません**。APIキーが見つからない場合、OpenClawはデフォルトでEdge TTSを使用します（`messages.tts.edge.enabled=false`で無効にしていない限り）。複数のプロバイダーが設定されている場合、選択されたプロバイダーが最初に使用され、他のプロバイダーはフォールバックオプションとなります。自動要約は設定された`summaryModel`（または`agents.defaults.model.primary`）を使用するため、要約を有効にする場合はそのプロバイダーも認証されている必要があります。

## サービスリンク

-   [OpenAI Text-to-Speechガイド](https://platform.openai.com/docs/guides/text-to-speech)
-   [OpenAI Audio APIリファレンス](https://platform.openai.com/docs/api-reference/audio)
-   [ElevenLabs Text to Speech](https://elevenlabs.io/docs/api-reference/text-to-speech)
-   [ElevenLabs Authentication](https://elevenlabs.io/docs/api-reference/authentication)
-   [node-edge-tts](https://github.com/SchneeHertz/node-edge-tts)
-   [Microsoft Speech出力フォーマット](https://learn.microsoft.com/azure/ai-services/speech-service/rest-text-to-speech#audio-outputs)

## デフォルトで有効になっていますか？

いいえ。自動TTSはデフォルトで**オフ**です。設定で`messages.tts.auto`を設定するか、セッションごとに`/tts always`（エイリアス: `/tts on`）で有効にしてください。Edge TTSは、TTSがオンになるとデフォルトで**有効**になり、OpenAIまたはElevenLabsのAPIキーが利用できない場合に自動的に使用されます。

## 設定

TTS設定は`openclaw.json`内の`messages.tts`にあります。完全なスキーマは[Gateway設定](./gateway/configuration.md)にあります。

### 最小限の設定（有効化 + プロバイダー）

```json
{
  messages: {
    tts: {
      auto: "always",
      provider: "elevenlabs",
    },
  },
}
```

### OpenAIプライマリ、ElevenLabsフォールバック

```json
{
  messages: {
    tts: {
      auto: "always",
      provider: "openai",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: {
        enabled: true,
      },
      openai: {
        apiKey: "openai_api_key",
        baseUrl: "https://api.openai.com/v1",
        model: "gpt-4o-mini-tts",
        voice: "alloy",
      },
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api.elevenlabs.io",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        languageCode: "en",
        voiceSettings: {
          stability: 0.5,
          similarityBoost: 0.75,
          style: 0.0,
          useSpeakerBoost: true,
          speed: 1.0,
        },
      },
    },
  },
}
```

### Edge TTSプライマリ（APIキー不要）

```json
{
  messages: {
    tts: {
      auto: "always",
      provider: "edge",
      edge: {
        enabled: true,
        voice: "en-US-MichelleNeural",
        lang: "en-US",
        outputFormat: "audio-24khz-48kbitrate-mono-mp3",
        rate: "+10%",
        pitch: "-5%",
      },
    },
  },
}
```

### Edge TTSを無効化

```json
{
  messages: {
    tts: {
      edge: {
        enabled: false,
      },
    },
  },
}
```

### カスタム制限 + 設定パス

```json
{
  messages: {
    tts: {
      auto: "always",
      maxTextLength: 4000,
      timeoutMs: 30000,
      prefsPath: "~/.openclaw/settings/tts.json",
    },
  },
}
```

### 受信音声メッセージの後にのみ音声で返信

```json
{
  messages: {
    tts: {
      auto: "inbound",
    },
  },
}
```

### 長い返信の自動要約を無効化

```json
{
  messages: {
    tts: {
      auto: "always",
    },
  },
}
```

その後、実行:

```bash
/tts summary off
```

### フィールドに関する注意

-   `auto`: 自動TTSモード (`off`, `always`, `inbound`, `tagged`)。
    -   `inbound`は受信音声メッセージの後にのみ音声を送信します。
    -   `tagged`は返信に`[[tts]]`タグが含まれる場合にのみ音声を送信します。
-   `enabled`: レガシートグル（doctorはこれを`auto`に移行します）。
-   `mode`: `"final"`（デフォルト）または`"all"`（ツール/ブロック返信を含む）。
-   `provider`: `"elevenlabs"`、`"openai"`、または`"edge"`（フォールバックは自動）。
-   `provider`が**未設定**の場合、OpenClawは`openai`（キーがあれば）、次に`elevenlabs`（キーがあれば）、それ以外は`edge`を優先します。
-   `summaryModel`: 自動要約用のオプションの安価なモデル。デフォルトは`agents.defaults.model.primary`。
    -   `provider/model`または設定済みのモデルエイリアスを受け入れます。
-   `modelOverrides`: モデルがTTSディレクティブを出力できるようにします（デフォルトでオン）。
    -   `allowProvider`はデフォルトで`false`（プロバイダー切り替えはオプトイン）。
-   `maxTextLength`: TTS入力のハードキャップ（文字数）。超過すると`/tts audio`は失敗します。
-   `timeoutMs`: リクエストタイムアウト（ミリ秒）。
-   `prefsPath`: ローカル設定JSONパスを上書きします（プロバイダー/制限/要約）。
-   `apiKey`の値は環境変数にフォールバックします（`ELEVENLABS_API_KEY`/`XI_API_KEY`、`OPENAI_API_KEY`）。
-   `elevenlabs.baseUrl`: ElevenLabs APIベースURLを上書きします。
-   `openai.baseUrl`: OpenAI TTSエンドポイントを上書きします。
    -   解決順序: `messages.tts.openai.baseUrl` -> `OPENAI_TTS_BASE_URL` -> `https://api.openai.com/v1`
    -   デフォルト以外の値はOpenAI互換のTTSエンドポイントとして扱われるため、カスタムモデル名と音声名が受け入れられます。
-   `elevenlabs.voiceSettings`:
    -   `stability`、`similarityBoost`、`style`: `0..1`
    -   `useSpeakerBoost`: `true|false`
    -   `speed`: `0.5..2.0` (1.0 = 通常)
-   `elevenlabs.applyTextNormalization`: `auto|on|off`
-   `elevenlabs.languageCode`: 2文字のISO 639-1（例: `en`、`de`）
-   `elevenlabs.seed`: 整数 `0..4294967295`（ベストエフォートの決定性）
-   `edge.enabled`: Edge TTSの使用を許可（デフォルト `true`、APIキー不要）。
-   `edge.voice`: Edgeニューラル音声名（例: `en-US-MichelleNeural`）。
-   `edge.lang`: 言語コード（例: `en-US`）。
-   `edge.outputFormat`: Edge出力フォーマット（例: `audio-24khz-48kbitrate-mono-mp3`）。
    -   有効な値についてはMicrosoft Speech出力フォーマットを参照。すべてのフォーマットがEdgeでサポートされているわけではありません。
-   `edge.rate` / `edge.pitch` / `edge.volume`: パーセント文字列（例: `+10%`、`-5%`）。
-   `edge.saveSubtitles`: 音声ファイルと一緒にJSON字幕を書き込みます。
-   `edge.proxy`: Edge TTSリクエスト用のプロキシURL。
-   `edge.timeoutMs`: リクエストタイムアウトの上書き（ミリ秒）。

## モデル駆動のオーバーライド（デフォルトオン）

デフォルトでは、モデルは単一の返信に対してTTSディレクティブを出力**できます**。`messages.tts.auto`が`tagged`の場合、これらのディレクティブは音声をトリガーするために必要です。有効にすると、モデルは`[[tts:...]]`ディレクティブを出力して単一の返信の音声を上書きでき、さらにオプションの`[[tts:text]]...[[/tts:text]]`ブロックを提供して、音声でのみ表示されるべき表現的なタグ（笑い声、歌唱の合図など）を含めることができます。`provider=...`ディレクティブは`modelOverrides.allowProvider: true`でない限り無視されます。返信ペイロードの例:

```
Here you go.

[[tts:voiceId=pMsXgVXv3BLzUgSXRplE model=eleven_v3 speed=1.1]]
[[tts:text]](laughs) Read the song once more.[[/tts:text]]
```

利用可能なディレクティブキー（有効時）:

-   `provider` (`openai` | `elevenlabs` | `edge`、`allowProvider: true`が必要)
-   `voice` (OpenAI音声) または `voiceId` (ElevenLabs)
-   `model` (OpenAI TTSモデルまたはElevenLabsモデルID)
-   `stability`、`similarityBoost`、`style`、`speed`、`useSpeakerBoost`
-   `applyTextNormalization` (`auto|on|off`)
-   `languageCode` (ISO 639-1)
-   `seed`

すべてのモデルオーバーライドを無効化:

```json
{
  messages: {
    tts: {
      modelOverrides: {
        enabled: false,
      },
    },
  },
}
```

オプションの許可リスト（他の設定ノブを構成可能に保ちながらプロバイダー切り替えを有効化）:

```json
{
  messages: {
    tts: {
      modelOverrides: {
        enabled: true,
        allowProvider: true,
        allowSeed: false,
      },
    },
  },
}
```

## ユーザーごとの設定

スラッシュコマンドはローカルの上書きを`prefsPath`（デフォルト: `~/.openclaw/settings/tts.json`、`OPENCLAW_TTS_PREFS`または`messages.tts.prefsPath`で上書き可能）に書き込みます。保存されるフィールド:

-   `enabled`
-   `provider`
-   `maxLength` (要約の閾値、デフォルト1500文字)
-   `summarize` (デフォルト `true`)

これらはそのホストの`messages.tts.*`を上書きします。

## 出力フォーマット（固定）

-   **Telegram**: Opus音声メッセージ（ElevenLabsからは`opus_48000_64`、OpenAIからは`opus`）。
    -   48kHz / 64kbpsは音声メッセージの良いトレードオフであり、丸いバブルに必要です。
-   **その他のチャンネル**: MP3（ElevenLabsからは`mp3_44100_128`、OpenAIからは`mp3`）。
    -   44.1kHz / 128kbpsは音声の明瞭さのデフォルトバランスです。
-   **Edge TTS**: `edge.outputFormat`を使用（デフォルト `audio-24khz-48kbitrate-mono-mp3`）。
    -   `node-edge-tts`は`outputFormat`を受け入れますが、すべてのフォーマットがEdgeサービスから利用できるわけではありません。citeturn2search0
    -   出力フォーマット値はMicrosoft Speech出力フォーマットに従います（Ogg/WebM Opusを含む）。citeturn1search0
    -   Telegramの`sendVoice`はOGG/MP3/M4Aを受け入れます。保証されたOpus音声メッセージが必要な場合はOpenAI/ElevenLabsを使用してください。citeturn1search1
    -   設定されたEdge出力フォーマットが失敗した場合、OpenClawはMP3で再試行します。

OpenAI/ElevenLabsのフォーマットは固定です。Telegramは音声メッセージUXのためにOpusを期待します。

## 自動TTSの動作

有効にすると、OpenClawは以下のように動作します:

-   返信にすでにメディアまたは`MEDIA:`ディレクティブが含まれている場合はTTSをスキップします。
-   非常に短い返信（< 10文字）をスキップします。
-   長い返信は、有効になっている場合、`agents.defaults.model.primary`（または`summaryModel`）を使用して要約します。
-   生成された音声を返信に添付します。

返信が`maxLength`を超え、要約がオフ（または要約モデルのAPIキーがない）の場合、音声はスキップされ、通常のテキスト返信が送信されます。

## フロー図

```
返信 -> TTS有効？
  いいえ -> テキスト送信
  はい -> メディア / MEDIA: / 短い？
          はい -> テキスト送信
          いいえ -> 長さ > 制限？
                   いいえ -> TTS -> 音声添付
                   はい -> 要约有効？
                            いいえ -> テキスト送信
                            はい -> 要約 (summaryModel または agents.defaults.model.primary)
                                      -> TTS -> 音声添付
```

## スラッシュコマンドの使用法

単一のコマンドがあります: `/tts`。有効化の詳細については[スラッシュコマンド](./tools/slash-commands.md)を参照してください。Discordに関する注意: `/tts`はDiscordの組み込みコマンドであるため、OpenClawはネイティブコマンドとして`/voice`を登録します。テキスト`/tts ...`も機能します。

```bash
/tts off
/tts always
/tts inbound
/tts tagged
/tts status
/tts provider openai
/tts limit 2000
/tts summary off
/tts audio Hello from OpenClaw
```

注意:

-   コマンドには認証された送信者が必要です（許可リスト/所有者ルールが引き続き適用されます）。
-   `commands.text`またはネイティブコマンド登録が有効になっている必要があります。
-   `off|always|inbound|tagged`はセッションごとのトグルです（`/tts on`は`/tts always`のエイリアスです）。
-   `limit`と`summary`はメイン設定ではなくローカル設定に保存されます。
-   `/tts audio`は一回限りの音声返信を生成します（TTSをオンにしません）。

## エージェントツール

`tts`ツールはテキストを音声に変換し、`MEDIA:`パスを返します。結果がTelegram互換の場合、ツールは`[[audio_as_voice]]`を含めるため、Telegramは音声バブルを送信します。

## Gateway RPC

Gatewayメソッド:

-   `tts.status`
-   `tts.enable`
-   `tts.disable`
-   `tts.convert`
-   `tts.setProvider`
-   `tts.providers`

[Location Command](./nodes/location-command.md)

---