

  拡張機能

  
# 音声通話プラグイン

プラグインを介したOpenClawの音声通話。発信通知と、着信ポリシーによるマルチターン会話をサポートします。現在のプロバイダー:

-   `twilio` (Programmable Voice + Media Streams)
-   `telnyx` (Call Control v2)
-   `plivo` (Voice API + XML transfer + GetInput speech)
-   `mock` (開発/ネットワークなし)

簡単なメンタルモデル:

-   プラグインをインストール
-   Gatewayを再起動
-   `plugins.entries.voice-call.config` で設定
-   `openclaw voicecall ...` または `voice_call` ツールを使用

## 実行場所 (ローカル vs リモート)

音声通話プラグインは **Gatewayプロセス内で実行されます**。リモートGatewayを使用する場合は、**Gatewayを実行しているマシン**にプラグインをインストール/設定し、Gatewayを再起動して読み込みます。

## インストール

### オプションA: npmからインストール (推奨)

```bash
openclaw plugins install @openclaw/voice-call
```

その後、Gatewayを再起動します。

### オプションB: ローカルフォルダからインストール (開発用、コピー不要)

```bash
openclaw plugins install ./extensions/voice-call
cd ./extensions/voice-call && pnpm install
```

その後、Gatewayを再起動します。

## 設定

`plugins.entries.voice-call.config` の下に設定します:

```json
{
  plugins: {
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio", // または "telnyx" | "plivo" | "mock"
          fromNumber: "+15550001234",
          toNumber: "+15550005678",

          twilio: {
            accountSid: "ACxxxxxxxx",
            authToken: "...",
          },

          telnyx: {
            apiKey: "...",
            connectionId: "...",
            // Telnyx Mission Control Portalから取得したTelnyxウェブフック公開鍵
            // (Base64文字列; TELNYX_PUBLIC_KEY経由でも設定可能)。
            publicKey: "...",
          },

          plivo: {
            authId: "MAxxxxxxxxxxxxxxxxxxxx",
            authToken: "...",
          },

          // ウェブフックサーバー
          serve: {
            port: 3334,
            path: "/voice/webhook",
          },

          // ウェブフックセキュリティ (トンネル/プロキシ使用時に推奨)
          webhookSecurity: {
            allowedHosts: ["voice.example.com"],
            trustedProxyIPs: ["100.64.0.1"],
          },

          // 公開アクセス (いずれか一つを選択)
          // publicUrl: "https://example.ngrok.app/voice/webhook",
          // tunnel: { provider: "ngrok" },
          // tailscale: { mode: "funnel", path: "/voice/webhook" }

          outbound: {
            defaultMode: "notify", // notify | conversation
          },

          streaming: {
            enabled: true,
            streamPath: "/voice/stream",
            preStartTimeoutMs: 5000,
            maxPendingConnections: 32,
            maxPendingConnectionsPerIp: 4,
            maxConnections: 128,
          },
        },
      },
    },
  },
}
```

注意点:

-   Twilio/Telnyxは **公開到達可能な** ウェブフックURLを必要とします。
-   Plivoは **公開到達可能な** ウェブフックURLを必要とします。
-   `mock` はローカル開発用プロバイダーです (ネットワーク呼び出しなし)。
-   Telnyxは、`skipSignatureVerification` がtrueでない限り、`telnyx.publicKey` (または `TELNYX_PUBLIC_KEY`) を必要とします。
-   `skipSignatureVerification` はローカルテスト専用です。
-   ngrok無料ティアを使用する場合は、`publicUrl` を正確なngrok URLに設定してください。署名検証は常に適用されます。
-   `tunnel.allowNgrokFreeTierLoopbackBypass: true` は、`tunnel.provider="ngrok"` かつ `serve.bind` がループバック (ngrokローカルエージェント) の場合にのみ、無効な署名を持つTwilioウェブフックを許可します。ローカル開発専用に使用してください。
-   ngrok無料ティアのURLは変更されたり、中間動作が追加されたりする可能性があります。`publicUrl` がずれると、Twilio署名は失敗します。本番環境では、安定したドメインまたはTailscale funnelを優先してください。
-   ストリーミングセキュリティのデフォルト:
    -   `streaming.preStartTimeoutMs` は有効な `start` フレームを送信しないソケットを閉じます。
    -   `streaming.maxPendingConnections` は認証前の開始前ソケットの総数を制限します。
    -   `streaming.maxPendingConnectionsPerIp` は送信元IPごとの認証前開始前ソケット数を制限します。
    -   `streaming.maxConnections` は開いているメディアストリームソケット (保留中 + アクティブ) の総数を制限します。

## 古い通話のクリーンアップ

`staleCallReaperSeconds` を使用して、終端ウェブフックを決して受信しない通話 (例えば、完了しない通知モードの通話) を終了させます。デフォルトは `0` (無効) です。推奨範囲:

-   **本番環境:** 通知スタイルのフローには `120`–`300` 秒。
-   この値を **`maxDurationSeconds` より高く** 保ち、通常の通話が完了できるようにします。良い開始点は `maxDurationSeconds + 30–60` 秒です。

例:

```json
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          maxDurationSeconds: 300,
          staleCallReaperSeconds: 360,
        },
      },
    },
  },
}
```

## ウェブフックセキュリティ

プロキシまたはトンネルがGatewayの前に配置されている場合、プラグインは署名検証のために公開URLを再構築します。これらのオプションは、どの転送ヘッダーが信頼されるかを制御します。`webhookSecurity.allowedHosts` は転送ヘッダーからのホストを許可リストに登録します。`webhookSecurity.trustForwardingHeaders` は許可リストなしで転送ヘッダーを信頼します。`webhookSecurity.trustedProxyIPs` は、リクエストのリモートIPがリストと一致する場合にのみ転送ヘッダーを信頼します。TwilioとPlivoではウェブフックリプレイ保護が有効になっています。リプレイされた有効なウェブフックリクエストは確認されますが、副作用のためにスキップされます。Twilio会話ターンには `` コールバックにターンごとのトークンが含まれるため、古い/リプレイされた音声コールバックは新しい保留中の文字起こしターンを満たすことができません。安定した公開ホストの例:

```json
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          publicUrl: "https://voice.example.com/voice/webhook",
          webhookSecurity: {
            allowedHosts: ["voice.example.com"],
          },
        },
      },
    },
  },
}
```

## 通話用TTS

音声通話は、通話でのストリーミング音声にコアの `messages.tts` 設定 (OpenAIまたはElevenLabs) を使用します。プラグイン設定の下で、**同じ形式** で上書きできます — これは `messages.tts` とディープマージされます。

```json
{
  tts: {
    provider: "elevenlabs",
    elevenlabs: {
      voiceId: "pMsXgVXv3BLzUgSXRplE",
      modelId: "eleven_multilingual_v2",
    },
  },
}
```

注意点:

-   **Edge TTSは音声通話では無視されます** (電話音声にはPCMが必要; Edge出力は信頼性が低い)。
-   コアTTSは、Twilioメディアストリーミングが有効な場合に使用されます。それ以外の場合は、通話はプロバイダーのネイティブ音声にフォールバックします。

### その他の例

コアTTSのみを使用 (上書きなし):

```json
{
  messages: {
    tts: {
      provider: "openai",
      openai: { voice: "alloy" },
    },
  },
}
```

通話専用にElevenLabsに上書き (他の場所ではコアデフォルトを維持):

```json
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          tts: {
            provider: "elevenlabs",
            elevenlabs: {
              apiKey: "elevenlabs_key",
              voiceId: "pMsXgVXv3BLzUgSXRplE",
              modelId: "eleven_multilingual_v2",
            },
          },
        },
      },
    },
  },
}
```

通話用にOpenAIモデルのみを上書き (ディープマージの例):

```json
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          tts: {
            openai: {
              model: "gpt-4o-mini-tts",
              voice: "marin",
            },
          },
        },
      },
    },
  },
}
```

## 着信通話

着信ポリシーのデフォルトは `disabled` です。着信通話を有効にするには、以下を設定します:

```json
{
  inboundPolicy: "allowlist",
  allowFrom: ["+15550001234"],
  inboundGreeting: "こんにちは！どうされましたか？",
}
```

自動応答はエージェントシステムを使用します。以下で調整できます:

-   `responseModel`
-   `responseSystemPrompt`
-   `responseTimeoutMs`

## CLI

```bash
openclaw voicecall call --to "+15555550123" --message "OpenClawからのメッセージです"
openclaw voicecall continue --call-id <id> --message "何か質問はありますか？"
openclaw voicecall speak --call-id <id> --message "少々お待ちください"
openclaw voicecall end --call-id <id>
openclaw voicecall status --call-id <id>
openclaw voicecall tail
openclaw voicecall expose --mode funnel
```

## エージェントツール

ツール名: `voice_call` アクション:

-   `initiate_call` (message, to?, mode?)
-   `continue_call` (callId, message)
-   `speak_to_user` (callId, message)
-   `end_call` (callId)
-   `get_status` (callId)

このリポジトリには、`skills/voice-call/SKILL.md` に一致するスキルドキュメントが同梱されています。

## Gateway RPC

-   `voicecall.initiate` (`to?`, `message`, `mode?`)
-   `voicecall.continue` (`callId`, `message`)
-   `voicecall.speak` (`callId`, `message`)
-   `voicecall.end` (`callId`)
-   `voicecall.status` (`callId`)

[コミュニティプラグイン](./community.md)[Zalo個人プラグイン](./zalouser.md)