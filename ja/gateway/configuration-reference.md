title: "OpenClaw Gateway 設定リファレンスと設定ガイド"
description: "OpenClaw Gateway のすべての設定フィールドの完全なリファレンス。チャネル、DMポリシー、モデルオーバーライドの設定、WhatsApp、Telegram、Discord の構成方法を学びます。"
keywords: ["openclaw 設定", "ゲートウェイ セットアップ", "チャネル 設定", "dm ポリシー", "モデル オーバーライド", "whatsapp ボット", "telegram ボット", "discord ボット"]
---

  設定と運用

  
# 設定リファレンス

`~/.openclaw/openclaw.json` で利用可能なすべてのフィールド。タスク指向の概要については、[設定](./configuration.md) を参照してください。設定形式は **JSON5** です（コメントと末尾のカンマが許可されます）。すべてのフィールドはオプションです — OpenClaw は省略された場合に安全なデフォルトを使用します。

* * *

## チャネル

各チャネルは、その設定セクションが存在する場合に自動的に開始します（`enabled: false` でない限り）。

### DM とグループアクセス

すべてのチャネルは DM ポリシーとグループポリシーをサポートします：

| DM ポリシー | 動作 |
| --- | --- |
| `pairing` (デフォルト) | 未知の送信者には一度限りのペアリングコードが送信される；所有者が承認する必要あり |
| `allowlist` | `allowFrom` 内の送信者（またはペアリング済みの許可ストア）のみ |
| `open` | すべての受信 DM を許可（`allowFrom: ["*"]` が必要） |
| `disabled` | すべての受信 DM を無視 |

| グループポリシー | 動作 |
| --- | --- |
| `allowlist` (デフォルト) | 設定された許可リストに一致するグループのみ |
| `open` | グループ許可リストをバイパス（メンションゲーティングは依然として適用） |
| `disabled` | すべてのグループ/ルームメッセージをブロック |

> **ℹ️** `channels.defaults.groupPolicy` は、プロバイダーの `groupPolicy` が未設定の場合のデフォルトを設定します。ペアリングコードは1時間後に期限切れになります。保留中の DM ペアリングリクエストは **チャネルごとに最大3件** に制限されます。プロバイダーブロックが完全に欠落している場合（`channels.` が存在しない）、ランタイムグループポリシーは起動時の警告とともに `allowlist`（フェイルクローズ）にフォールバックします。

### チャネルモデルオーバーライド

`channels.modelByChannel` を使用して、特定のチャネル ID をモデルに固定します。値は `provider/model` または設定済みのモデルエイリアスを受け入れます。このチャネルマッピングは、セッションが既にモデルオーバーライド（例えば `/model` で設定されたもの）を持っていない場合に適用されます。

```json
{
  channels: {
    modelByChannel: {
      discord: {
        "123456789012345678": "anthropic/claude-opus-4-6",
      },
      slack: {
        C1234567890: "openai/gpt-4.1",
      },
      telegram: {
        "-1001234567890": "openai/gpt-4.1-mini",
        "-1001234567890:topic:99": "anthropic/claude-sonnet-4-6",
      },
    },
  },
}
```

### チャネルデフォルトとハートビート

`channels.defaults` を使用して、プロバイダー間で共有されるグループポリシーとハートビートの動作を設定します：

```json
{
  channels: {
    defaults: {
      groupPolicy: "allowlist", // open | allowlist | disabled
      heartbeat: {
        showOk: false,
        showAlerts: true,
        useIndicator: true,
      },
    },
  },
}
```

-   `channels.defaults.groupPolicy`: プロバイダーレベルの `groupPolicy` が未設定の場合のフォールバックグループポリシー。
-   `channels.defaults.heartbeat.showOk`: ハートビート出力に正常なチャネルステータスを含める。
-   `channels.defaults.heartbeat.showAlerts`: ハートビート出力に低下/エラーステータスを含める。
-   `channels.defaults.heartbeat.useIndicator`: コンパクトなインジケータースタイルのハートビート出力をレンダリング。

### WhatsApp

WhatsApp はゲートウェイのウェブチャネル（Baileys Web）を介して実行されます。リンクされたセッションが存在すると自動的に開始します。

```json
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000,
      chunkMode: "length", // length | newline
      mediaMaxMb: 50,
      sendReadReceipts: true, // 青いチェックマーク（セルフチャットモードでは false）
      groups: {
        "*": { requireMention: true },
      },
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
  },
  web: {
    enabled: true,
    heartbeatSeconds: 60,
    reconnect: {
      initialMs: 2000,
      maxMs: 120000,
      factor: 1.4,
      jitter: 0.2,
      maxAttempts: 0,
    },
  },
}
```

```json
{
  channels: {
    whatsapp: {
      accounts: {
        default: {},
        personal: {},
        biz: {
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

-   アウトバウンドコマンドは、`default` アカウントが存在する場合、デフォルトでそれを使用します。それ以外の場合は、最初に設定されたアカウント ID（ソート済み）を使用します。
-   オプションの `channels.whatsapp.defaultAccount` は、設定済みのアカウント ID と一致する場合、そのフォールバックデフォルトアカウント選択をオーバーライドします。
-   レガシーシングルアカウントの Baileys 認証ディレクトリは、`openclaw doctor` によって `whatsapp/default` に移行されます。
-   アカウントごとのオーバーライド: `channels.whatsapp.accounts..sendReadReceipts`, `channels.whatsapp.accounts..dmPolicy`, `channels.whatsapp.accounts..allowFrom`.

### Telegram

```json
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "your-bot-token",
      dmPolicy: "pairing",
      allowFrom: ["tg:123456789"],
      groups: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFrom: ["@admin"],
          systemPrompt: "回答は簡潔に。",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "トピックに沿って。",
            },
          },
        },
      },
      customCommands: [
        { command: "backup", description: "Git バックアップ" },
        { command: "generate", description: "画像を生成" },
      ],
      historyLimit: 50,
      replyToMode: "first", // off | first | all
      linkPreview: true,
      streaming: "partial", // off | partial | block | progress (デフォルト: off)
      actions: { reactions: true, sendMessage: true },
      reactionNotifications: "own", // off | own | all
      mediaMaxMb: 100,
      retry: {
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
      network: {
        autoSelectFamily: true,
        dnsResultOrder: "ipv4first",
      },
      proxy: "socks5://localhost:9050",
      webhookUrl: "https://example.com/telegram-webhook",
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook",
    },
  },
}
```

-   ボットトークン: `channels.telegram.botToken` または `channels.telegram.tokenFile`、デフォルトアカウントのフォールバックとして `TELEGRAM_BOT_TOKEN`。
-   オプションの `channels.telegram.defaultAccount` は、設定済みのアカウント ID と一致する場合、デフォルトアカウント選択をオーバーライドします。
-   マルチアカウント設定（2つ以上のアカウント ID）では、明示的なデフォルト（`channels.telegram.defaultAccount` または `channels.telegram.accounts.default`）を設定して、フォールバックルーティングを回避します。`openclaw doctor` はこれが欠落または無効な場合に警告します。
-   `configWrites: false` は、Telegram によって開始された設定書き込み（スーパーグループ ID 移行、`/config set|unset`）をブロックします。
-   トップレベルの `bindings[]` エントリで `type: "acp"` を使用すると、フォーラムトピックの永続的な ACP バインディングを設定します（`match.peer.id` に正規の `chatId:topic:topicId` を使用）。フィールドのセマンティクスは [ACP エージェント](../tools/acp-agents.md#channel-specific-settings) で共有されています。
-   Telegram ストリームプレビューは `sendMessage` + `editMessageText` を使用します（ダイレクトチャットとグループチャットで動作）。
-   再試行ポリシー: [再試行ポリシー](../concepts/retry.md) を参照。

### Discord

```json
{
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      mediaMaxMb: 8,
      allowBots: false,
      actions: {
        reactions: true,
        stickers: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        voiceStatus: true,
        events: true,
        moderation: false,
      },
      replyToMode: "off", // off | first | all
      dmPolicy: "pairing",
      allowFrom: ["1234567890", "123456789012345678"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["openclaw-dm"] },
      guilds: {
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          ignoreOtherMentions: true,
          reactionNotifications: "own",
          users: ["987654321098765432"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["docs"],
              systemPrompt: "回答は短く。",
            },
          },
        },
      },
      historyLimit: 20,
      textChunkLimit: 2000,
      chunkMode: "length", // length | newline
      streaming: "off", // off | partial | block | progress (progress は Discord では partial にマップ)
      maxLinesPerMessage: 17,
      ui: {
        components: {
          accentColor: "#5865F2",
        },
      },
      threadBindings: {
        enabled: true,
        idleHours: 24,
        maxAgeHours: 0,
        spawnSubagentSessions: false, // sessions_spawn({ thread: true }) のオプトイン
      },
      voice: {
        enabled: true,
        autoJoin: [
          {
            guildId: "123456789012345678",
            channelId: "234567890123456789",
          },
        ],
        daveEncryption: true,
        decryptionFailureTolerance: 24,
        tts: {
          provider: "openai",
          openai: { voice: "alloy" },
        },
      },
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

-   トークン: `channels.discord.token`、デフォルトアカウントのフォールバックとして `DISCORD_BOT_TOKEN`。
-   オプションの `channels.discord.defaultAccount` は、設定済みのアカウント ID と一致する場合、デフォルトアカウント選択をオーバーライドします。
-   配信ターゲットには `user:`（DM）または `channel:`（ギルドチャネル）を使用します。単なる数値 ID は拒否されます。
-   ギルドスラッグは小文字でスペースは `-` に置き換えられます。チャネルキーはスラッグ化された名前を使用します（`#` なし）。ギルド ID を優先してください。
-   ボット作成メッセージはデフォルトで無視されます。`allowBots: true` で有効化します。`allowBots: "mentions"` を使用すると、ボットにメンションするボットメッセージのみを受け入れます（自身のメッセージは依然としてフィルタリングされます）。
-   `channels.discord.guilds..ignoreOtherMentions`（およびチャネルオーバーライド）は、ボット以外のユーザーやロールにメンションしているがボットにはメンションしていないメッセージをドロップします（@everyone/@here を除く）。
-   `maxLinesPerMessage`（デフォルト 17）は、2000 文字未満でも長いメッセージを分割します。
-   `channels.discord.threadBindings` は、Discord スレッドバインドルーティングを制御します：
    -   `enabled`: スレッドバインドセッション機能（`/focus`、`/unfocus`、`/agents`、`/session idle`、`/session max-age`、およびバインド配信/ルーティング）の Discord オーバーライド
    -   `idleHours`: 非アクティブ自動アンフォーカスの Discord オーバーライド（時間単位、`0` で無効）
    -   `maxAgeHours`: ハード最大経過時間の Discord オーバーライド（時間単位、`0` で無効）
    -   `spawnSubagentSessions`: `sessions_spawn({ thread: true })` 自動スレッド作成/バインドのオプトインスイッチ
-   トップレベルの `bindings[]` エントリで `type: "acp"` を使用すると、チャネルとスレッドの永続的な ACP バインディングを設定します（`match.peer.id` にチャネル/スレッド ID を使用）。フィールドのセマンティクスは [ACP エージェント](../tools/acp-agents.md#channel-specific-settings) で共有されています。
-   `channels.discord.ui.components.accentColor` は、Discord コンポーネント v2 コンテナのアクセントカラーを設定します。
-   `channels.discord.voice` は、Discord ボイスチャネル会話とオプションの自動参加 + TTS オーバーライドを有効にします。
-   `channels.discord.voice.daveEncryption` と `channels.discord.voice.decryptionFailureTolerance` は、`@discordjs/voice` DAVE オプションに渡されます（デフォルトは `true` と `24`）。
-   OpenClaw はさらに、繰り返される復号化失敗後にボイスセッションを退出/再参加することで、ボイス受信回復を試みます。
-   `channels.discord.streaming` は正規のストリームモードキーです。レガシーな `streamMode` とブール値 `streaming` は自動移行されます。
-   `channels.discord.autoPresence` は、ランタイム可用性をボットのプレゼンスにマッピングします（正常 => オンライン、低下 => 退席中、枯渇 => 取り込み中）。オプションのステータステキストオーバーライドも許可します。
-   `channels.discord.dangerouslyAllowNameMatching` は、可変の名前/タグマッチングを再度有効にします（非常用互換モード）。

**リアクション通知モード:** `off`（なし）、`own`（ボットのメッセージ、デフォルト）、`all`（すべてのメッセージ）、`allowlist`（`guilds..users` からのすべてのメッセージ）。

### Google Chat

```json
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url", // app-url | project-number
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890",
      dm: {
        enabled: true,
        policy: "pairing",
        allowFrom: ["users/1234567890"],
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": { allow: true, requireMention: true },
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```

-   サービスアカウント JSON: インライン（`serviceAccount`）またはファイルベース（`serviceAccountFile`）。
-   サービスアカウント SecretRef もサポートされています（`serviceAccountRef`）。
-   環境変数フォールバック: `GOOGLE_CHAT_SERVICE_ACCOUNT` または `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`。
-   配信ターゲットには `spaces/` または `users/` を使用します。
-   `channels.googlechat.dangerouslyAllowNameMatching` は、可変のメールプリンシパルマッチングを再度有効にします（非常用互換モード）。

### Slack

```json
{
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      dmPolicy: "pairing",
      allowFrom: ["U123", "U456", "*"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["G123"] },
      channels: {
        C123: { allow: true, requireMention: true, allowBots: false },
        "#general": {
          allow: true,
          requireMention: true,
          allowBots: false,
          users: ["U123"],
          skills: ["docs"],
          systemPrompt: "回答は短く。",
        },
      },
      historyLimit: 50,
      allowBots: false,
      reactionNotifications: "own",
      reactionAllowlist: ["U123"],
      replyToMode: "off", // off | first | all
      thread: {
        historyScope: "thread", // thread | channel
        inheritParent: false,
      },
      actions: {
        reactions: true,
        messages: true,
        pins: true,
        memberInfo: true,
        emojiList: true,
      },
      slashCommand: {
        enabled: true,
        name: "openclaw",
        sessionPrefix: "slack:slash",
        ephemeral: true,
      },
      typingReaction: "hourglass_flowing_sand",
      textChunkLimit: 4000,
      chunkMode: "length",
      streaming: "partial", // off | partial | block | progress (プレビューモード)
      nativeStreaming: true, // streaming=partial 時に Slack ネイティブストリーミング API を使用
      mediaMaxMb: 20,
    },
  },
}
```

-   **ソケットモード** には `botToken` と `appToken` の両方が必要です（デフォルトアカウントの環境変数フォールバックには `SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN`）。
-   **HTTP モード** には `botToken` に加えて `signingSecret`（ルートまたはアカウントごと）が必要です。
-   `configWrites: false` は、Slack によって開始された設定書き込みをブロックします。
-   オプションの `channels.slack.defaultAccount` は、設定済みのアカウント ID と一致する場合、デフォルトアカウント選択をオーバーライドします。
-   `channels.slack.streaming` は正規のストリームモードキーです。レガシーな `streamMode` とブール値 `streaming` は自動移行されます。
-   配信ターゲットには `user:`（DM）または `channel:` を使用します。

**リアクション通知モード:** `off`、`own`（デフォルト）、`all`、`allowlist`（`reactionAllowlist` から）。**スレッドセッション分離:** `thread.historyScope` はスレッドごと（デフォルト）またはチャネル全体で共有されます。`thread.inheritParent` は親チャネルのトランスクリプトを新しいスレッドにコピーします。

-   `typingReaction` は、返信実行中に受信 Slack メッセージに一時的なリアクションを追加し、完了時に削除します。`"hourglass_flowing_sand"` などの Slack 絵文字ショートコードを使用します。

| アクショングループ | デフォルト | 備考 |
| --- | --- | --- |
| reactions | enabled | リアクション + リアクション一覧 |
| messages | enabled | 読み取り/送信/編集/削除 |
| pins | enabled | ピン留め/ピン外し/一覧 |
| memberInfo | enabled | メンバー情報 |
| emojiList | enabled | カスタム絵文字一覧 |

### Mattermost

Mattermost はプラグインとして提供されます: `openclaw plugins install @openclaw/mattermost`。

```json
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
      chatmode: "oncall", // oncall | onmessage | onchar
      oncharPrefixes: [">", "!"],
      commands: {
        native: true, // オプトイン
        nativeSkills: true,
        callbackPath: "/api/channels/mattermost/command",
        // リバースプロキシ/公開デプロイ用のオプション明示的 URL
        callbackUrl: "https://gateway.example.com/api/channels/mattermost/command",
      },
      textChunkLimit: 4000,
      chunkMode: "length",
    },
  },
}
```

チャットモード: `oncall`（@メンション時に応答、デフォルト）、`onmessage`（すべてのメッセージ）、`onchar`（トリガープレフィックスで始まるメッセージ）。Mattermost ネイティブコマンドが有効な場合：

-   `commands.callbackPath` は完全な URL ではなく、パス（例: `/api/channels/mattermost/command`）である必要があります。
-   `commands.callbackUrl` は OpenClaw ゲートウェイエンドポイントに解決され、Mattermost サーバーから到達可能である必要があります。
-   プライベート/テイルネット/内部コールバックホストの場合、Mattermost は `ServiceSettings.AllowedUntrustedInternalConnections` にコールバックホスト/ドメインを含める必要があるかもしれません。完全な URL ではなく、ホスト/ドメイン値を使用してください。
-   `channels.mattermost.configWrites`: Mattermost によって開始された設定書き込みを許可または拒否します。
-   `channels.mattermost.requireMention`: チャネルでの返信前に `@mention` を要求します。
-   オプションの `channels.mattermost.defaultAccount` は、設定済みのアカウント ID と一致する場合、デフォルトアカウント選択をオーバーライドします。

### Signal

```json
{
  channels: {
    signal: {
      enabled: true,
      account: "+15555550123", // オプションのアカウントバインド
      dmPolicy: "pairing",
      allowFrom: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      configWrites: true,
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50,
    },
  },
}
```

**リアクション通知モード:** `off`、`own`（デフォルト）、`all`、`allowlist`（`reactionAllowlist` から）。

-   `channels.signal.account`: チャネル起動を特定の Signal アカウント ID に固定します。
-   `channels.signal.configWrites`: Signal によって開始された設定書き込みを許可または拒否します。
-   オプションの `channels.signal.defaultAccount` は、設定済みのアカウント ID と一致する場合、デフォルトアカウント選択をオーバーライドします。

### BlueBubbles

BlueBubbles は推奨される iMessage パスです（プラグインバック、`channels.bluebubbles` で設定）。

```json
{
  channels: {
    bluebubbles: {
      enabled: true,
      dmPolicy: "pairing",
      // serverUrl, password, webhookPath, group controls, and advanced actions:
      // see /channels/bluebubbles
    },
  },
}
```

-   ここでカバーされるコアキーパス: `channels.bluebubbles`、`channels.bluebubbles.dmPolicy`。
-   オプションの `channels.bluebubbles.defaultAccount` は、設定済みのアカウント ID と一致する場合、デフォルトアカウント選択をオーバーライドします。
-   完全な BlueBubbles チャネル設定は [BlueBubbles](../channels/bluebubbles.md) に記載されています。

### iMessage

OpenClaw は `imsg rpc` を生成します（stdio を介した JSON-RPC）。デーモンやポートは不要です。

```json
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat.db",
      remoteHost: "user@gateway-host",
      dmPolicy: "pairing",
      allowFrom: ["+15555550123", "user@example.com", "chat_id:123"],
      historyLimit: 50,
      includeAttachments: false,
      attachmentRoots: ["/Users/*/Library/Messages/Attachments"],
      remoteAttachmentRoots: ["/Users/*/Library/Messages/Attachments"],
      mediaMaxMb: 16,
      service: "auto",
      region: "US",
    },
  },
}
```

-   オプションの `channels.imessage.defaultAccount` は、設定済みのアカウント ID と一致する場合、デフォルトアカウント選択をオーバーライドします。
-   メッセージ DB への完全なディスクアクセスが必要です。
-   ターゲットには `chat_id:` を優先してください。チャット一覧は `imsg chats --limit 20` を使用します。
-   `cliPath` は SSH ラッパーを指すことができます。SCP 添付ファイル取得には `remoteHost`（`host` または `user@host`）を設定します。
-   `attachmentRoots` と `remoteAttachmentRoots` は受信添付ファイルパスを制限します（デフォルト: `/Users/*/Library/Messages/Attachments`）。
-   SCP は厳格なホストキーチェックを使用するため、リレーホストキーが既に `~/.ssh/known_hosts` に存在することを確認してください。
-   `channels.imessage.configWrites`: iMessage によって開始された設定書き込みを許可または拒否します。

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

### Microsoft Teams

Microsoft Teams は拡張機能バックで、`channels.msteams` で設定されます。

```json
{
  channels: {
    msteams: {
      enabled: true,
      configWrites: true,
      // appId, appPassword, tenantId, webhook, team/channel policies:
      // see /channels/msteams
    },
  },
}
```

-   ここでカバーされるコアキーパス: `channels.msteams`、`channels.msteams.configWrites`。
-   完全な Teams 設定（認証情報、ウェブフック、DM/グループポリシー、チーム/チャネルごとのオーバーライド）は [Microsoft Teams](../channels/msteams.md) に記載されています。

### IRC

IRC は拡張機能バックで、`channels.irc` で設定されます。

```json
{
  channels: {
    irc: {
      enabled: true,
      dmPolicy: "pairing",
      configWrites: true,
      nickserv: {
        enabled: true,
        service: "NickServ",
        password: "${IRC_NICKSERV_PASSWORD}",
        register: false,
        registerEmail: "bot@example.com",
      },
    },
  },
}
```

-   ここでカバーされるコアキーパス: `channels.irc`、`channels.irc.dmPolicy`、`channels.irc.configWrites`、`channels.irc.nickserv.*`。
-   オプションの `channels.irc.defaultAccount` は、設定済みのアカウント ID と一致する場合、デフォルトアカウント選択をオーバーライドします。
-   完全な IRC チャネル設定（ホスト/ポート/TLS/チャネル/許可リスト/メンションゲーティング）は [IRC](../channels/irc.md) に記載されています。

### マルチアカウント（すべてのチャネル）

チャネルごとに複数のアカウントを実行します（それぞれ独自の `accountId` を持ちます）：

```json
{
  channels: {
    telegram: {
      accounts: {
        default: {
          name: "Primary bot",
          botToken: "123456:ABC...",
        },
        alerts: {
          name: "Alerts bot",
          botToken: "987654:XYZ...",
        },
      },
    },
  },
}
```

-   `default` は `accountId` が省略された場合に使用されます（CLI + ルーティング）。
-   環境変数トークンは **デフォルト** アカウントにのみ適用されます。
-   ベースチャネル設定は、アカウントごとにオーバーライドされない限り、すべてのアカウントに適用されます。
-   `bindings[].match.accountId` を使用して、各アカウントを異なるエージェントにルーティングします。
-   トップレベルのシングルアカウントチャネル設定のまま `openclaw channels add`（またはチャネルオンボーディング）を介して非デフォルトアカウントを追加すると、OpenClaw は最初にアカウントスコープのトップレベルシングルアカウント値を `channels..accounts.default` に移動し、元のアカウントが動作し続けるようにします。
-   既存のチャネルのみのバインディング（`accountId` なし）はデフォルトアカウントに一致し続けます。アカウントスコープのバインディングはオプションのままです。
-   `openclaw doctor --fix` も、名前付きアカウントが存在するが `default` が欠落している場合、アカウントスコープのトップレベルシングルアカウント値を `accounts.default` に移動することで、混合形状を修復します。

### その他の拡張チャネル

多くの拡張チャネルは `channels.` として設定され、専用のチャネルページ（例: Feishu、Matrix、LINE、Nostr、Zalo、Nextcloud Talk、Synology Chat、Twitch）に記載されています。完全なチャネルインデックスを参照: [チャネル](../channels.md)。

### グループチャットメンション