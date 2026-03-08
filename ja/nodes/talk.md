

  メディアとデバイス

  
# Talkモード

Talkモードは連続的な音声会話ループです：

1.  音声を待ち受ける
2.  文字起こしをモデルに送信（メインセッション、chat.send）
3.  応答を待つ
4.  ElevenLabs経由で発話（ストリーミング再生）

## 動作（macOS）

-   Talkモードが有効な間は**常にオーバーレイ表示**されます。
-   **待機 → 思考中 → 発話中**のフェーズ遷移を行います。
-   **短い間（無音ウィンドウ）** で、現在の文字起こしが送信されます。
-   返答は**WebChatに書き込まれます**（タイピング時と同じ）。
-   **発話で割り込み**（デフォルトオン）：アシスタントが発話中にユーザーが話し始めた場合、再生を停止し、次のプロンプト用に割り込みタイムスタンプを記録します。

## 返答内の音声ディレクティブ

アシスタントは、音声を制御するために返答の先頭に**単一のJSON行**を付ける場合があります：

```json
{ "voice": "<voice-id>", "once": true }
```

ルール：

-   最初の空でない行のみが対象です。
-   不明なキーは無視されます。
-   `once: true` は現在の返答にのみ適用されます。
-   `once` なしの場合、その音声はTalkモードの新しいデフォルトになります。
-   JSON行はTTS再生前に取り除かれます。

サポートされるキー：

-   `voice` / `voice_id` / `voiceId`
-   `model` / `model_id` / `modelId`
-   `speed`, `rate` (WPM), `stability`, `similarity`, `style`, `speakerBoost`
-   `seed`, `normalize`, `lang`, `output_format`, `latency_tier`
-   `once`

## 設定 (~/.openclaw/openclaw.json)

```json
{
  talk: {
    voiceId: "elevenlabs_voice_id",
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128",
    apiKey: "elevenlabs_api_key",
    interruptOnSpeech: true,
  },
}
```

デフォルト：

-   `interruptOnSpeech`: true
-   `voiceId`: `ELEVENLABS_VOICE_ID` / `SAG_VOICE_ID` にフォールバック（またはAPIキーが利用可能な場合は最初のElevenLabs音声）
-   `modelId`: 未設定時はデフォルトで `eleven_v3`
-   `apiKey`: `ELEVENLABS_API_KEY` にフォールバック（または利用可能な場合はゲートウェイシェルプロファイル）
-   `outputFormat`: macOS/iOSではデフォルト `pcm_44100`、Androidでは `pcm_24000`（MP3ストリーミングを強制するには `mp3_*` を設定）

## macOS UI

-   メニューバートグル：**Talk**
-   設定タブ：**Talkモード**グループ（音声ID + 割り込みトグル）
-   オーバーレイ：
    -   **待機中**: マイクレベルと共に雲が脈動
    -   **思考中**: 沈み込むアニメーション
    -   **発話中**: 放射状に広がるリング
    -   雲をクリック：発話を停止
    -   Xをクリック：Talkモードを終了

## 注意事項

-   音声認識 + マイクの権限が必要です。
-   セッションキー `main` に対して `chat.send` を使用します。
-   TTSはElevenLabsストリーミングAPIと `ELEVENLABS_API_KEY` を使用し、macOS/iOS/Androidでは低遅延のためのインクリメンタル再生を行います。
-   `eleven_v3` の `stability` は `0.0`、`0.5`、`1.0` に検証されます。他のモデルは `0..1` を受け入れます。
-   `latency_tier` は設定時、`0..4` に検証されます。
-   Androidは低遅延のAudioTrackストリーミングのために、`pcm_16000`、`pcm_22050`、`pcm_24000`、`pcm_44100` の出力フォーマットをサポートしています。

[カメラキャプチャ](./camera.md)[音声ウェイク](./voicewake.md)