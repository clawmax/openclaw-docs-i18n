

  メッセージングプラットフォーム

  
# Telegram

ステータス: grammY 経由のボットDMおよびグループで本番環境対応。デフォルトモードはロングポーリング。Webhook モードはオプション。

## クイックセットアップ

### ステップ 1: BotFather でボットトークンを作成する

Telegram を開き、**@BotFather** とチャットします（ハンドルが正確に `@BotFather` であることを確認してください）。`/newbot` を実行し、プロンプトに従い、トークンを保存します。

### ステップ 2: トークンと DM ポリシーを設定する

```json
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```

環境変数フォールバック: `TELEGRAM_BOT_TOKEN=...` (デフォルトアカウントのみ)。Telegram は `openclaw channels login telegram` を使用**しません**。設定/環境変数でトークンを設定し、ゲートウェイを起動します。

### ステップ 3: ゲートウェイを起動し、最初の DM を承認する

```bash
openclaw gateway
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

ペアリングコードは1時間で期限切れになります。

### ステップ 4: ボットをグループに追加する

ボットをグループに追加し、`channels.telegram.groups` と `groupPolicy` を設定してアクセスモデルに合わせます。

 

> **ℹ️** トークン解決順序はアカウントを認識します。実際には、設定値が環境変数フォールバックよりも優先され、`TELEGRAM_BOT_TOKEN` はデフォルトアカウントにのみ適用されます。

## Telegram 側の設定

Telegram ボットはデフォルトで**プライバシーモード**になり、グループメッセージの受信が制限されます。ボットがすべてのグループメッセージを見る必要がある場合は、以下のいずれかを実行します:

-   `/setprivacy` でプライバシーモードを無効にする、または
-   ボットをグループ管理者にする。

プライバシーモードを切り替える際は、変更を適用するために各グループでボットを削除して再追加してください。

管理者ステータスは Telegram グループ設定で制御されます。管理者ボットはすべてのグループメッセージを受信し、常時オン状態のグループ動作に役立ちます。

-   `/setjoingroups` でグループ追加を許可/拒否
-   `/setprivacy` でグループ可視性動作を設定

## アクセス制御とアクティベーション

/getUpdates"', lang: 'bash' }, { label: 'グループポリシーと許可リスト', code: '{\n  channels: {\n    telegram: {\n      groups: {\n        "-1001234567890": {\n          groupPolicy: "open",\n          requireMention: false,\n        },\n      },\n    },\n  },\n}', lang: 'json' }, { label: 'メンション動作', code: '{\n  channels: {\n    telegram: {\n      groups: {\n        "*": { requireMention: false },\n      },\n    },\n  },\n}', lang: 'json' }]} />

## ランタイム動作

-   Telegram はゲートウェイプロセスによって所有されます。
-   ルーティングは決定的です: Telegram からの受信メッセージは Telegram に返信されます（モデルはチャネルを選択しません）。
-   受信メッセージは、返信メタデータとメディアプレースホルダーを含む共有チャネルエンベロープに正規化されます。
-   グループセッションはグループ ID によって分離されます。フォーラムトピックは `:topic:` を追加してトピックを分離します。
-   DM メッセージは `message_thread_id` を保持できます。OpenClaw はスレッドを認識したセッションキーでそれらをルーティングし、返信用にスレッド ID を保持します。
-   ロングポーリングは、チャットごと/スレッドごとのシーケンシングで grammY ランナーを使用します。全体のランナーシンク同時実行性には `agents.defaults.maxConcurrent` を使用します。
-   Telegram Bot API には既読サポートがありません（`sendReadReceipts` は適用されません）。

## 機能リファレンス

OpenClaw は部分的な返信をリアルタイムでストリーミングできます:

-   ダイレクトチャット: `sendMessageDraft` による Telegram ネイティブドラフトストリーミング
-   グループ/トピック: プレビューメッセージ + `editMessageText`

要件:

-   `channels.telegram.streaming` が `off | partial | block | progress` (デフォルト: `partial`)
-   `progress` は Telegram 上で `partial` にマッピングされます（クロスチャネル命名との互換性）
-   レガシーな `channels.telegram.streamMode` とブール値 `streaming` は自動マッピングされます

Telegram は Bot API 9.5 (2026年3月1日) で全てのボットに対して `sendMessageDraft` を有効にしました。テキストのみの返信の場合:

-   DM: OpenClaw はドラフトをその場で更新します（追加のプレビューメッセージなし）
-   グループ/トピック: OpenClaw は同じプレビューメッセージを保持し、最終的な編集をその場で実行します（2番目のメッセージなし）

複雑な返信（例: メディアペイロード）の場合、OpenClaw は通常の最終配信にフォールバックし、その後プレビューメッセージをクリーンアップします。プレビューストリーミングはブロックストリーミングとは別です。Telegram に対してブロックストリーミングが明示的に有効になっている場合、OpenClaw は二重ストリーミングを避けるためにプレビューストリームをスキップします。ネイティブドラフトトランスポートが利用できない/拒否された場合、OpenClaw は自動的に `sendMessage` + `editMessageText` にフォールバックします。Telegram 専用推論ストリーム:

-   `/reasoning stream` は生成中に推論をライブプレビューに送信します
-   最終回答は推論テキストなしで送信されます

送信テキストは Telegram `parse_mode: "HTML"` を使用します。

-   Markdown 風のテキストは Telegram セーフな HTML にレンダリングされます。
-   生のモデル HTML は Telegram パース失敗を減らすためにエスケープされます。
-   Telegram がパースされた HTML を拒否した場合、OpenClaw はプレーンテキストとして再試行します。

リンクプレビューはデフォルトで有効で、`channels.telegram.linkPreview: false` で無効にできます。

Telegram コマンドメニュー登録は、`setMyCommands` で起動時に処理されます。ネイティブコマンドのデフォルト:

-   `commands.native: "auto"` は Telegram のネイティブコマンドを有効にします

カスタムコマンドメニューエントリを追加:

```json
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Git バックアップ" },
        { command: "generate", description: "画像を作成" },
      ],
    },
  },
}
```

ルール:

-   名前は正規化されます（先頭の `/` を削除、小文字化）
-   有効なパターン: `a-z`, `0-9`, `_`, 長さ `1..32`
-   カスタムコマンドはネイティブコマンドをオーバーライドできません
-   競合/重複はスキップされログに記録されます

注意:

-   カスタムコマンドはメニューエントリのみです。動作は自動的に実装されません
-   プラグイン/スキルコマンドは、Telegram メニューに表示されなくても、入力されれば動作する可能性があります

ネイティブコマンドが無効になっている場合、ビルトインコマンドは削除されます。カスタム/プラグインコマンドは設定されていれば登録される可能性があります。一般的なセットアップ失敗:

-   `setMyCommands failed` は通常、`api.telegram.org` への送信 DNS/HTTPS がブロックされていることを意味します。

### デバイスペアリングコマンド (device-pair プラグイン)

`device-pair` プラグインがインストールされている場合:

1.  `/pair` でセットアップコードを生成
2.  iOS アプリにコードを貼り付け
3.  `/pair approve` で最新の保留中リクエストを承認

詳細: [ペアリング](./pairing.md#pair-via-telegram-recommended-for-ios)。

インラインキーボードスコープを設定:

```json
{
  channels: {
    telegram: {
      capabilities: {
        inlineButtons: "allowlist",
      },
    },
  },
}
```

アカウントごとのオーバーライド:

```json
{
  channels: {
    telegram: {
      accounts: {
        main: {
          capabilities: {
            inlineButtons: "allowlist",
          },
        },
      },
    },
  },
}
```

スコープ:

-   `off`
-   `dm`
-   `group`
-   `all`
-   `allowlist` (デフォルト)

レガシー `capabilities: ["inlineButtons"]` は `inlineButtons: "all"` にマッピングされます。メッセージアクション例:

```json
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  message: "オプションを選択:",
  buttons: [
    [
      { text: "はい", callback_data: "yes" },
      { text: "いいえ", callback_data: "no" },
    ],
    [{ text: "キャンセル", callback_data: "cancel" }],
  ],
}
```

コールバッククリックはテキストとしてエージェントに渡されます: `callback_data: `

Telegram ツールアクションには以下が含まれます:

-   `sendMessage` (`to`, `content`, オプション `mediaUrl`, `replyToMessageId`, `messageThreadId`)
-   `react` (`chatId`, `messageId`, `emoji`)
-   `deleteMessage` (`chatId`, `messageId`)
-   `editMessage` (`chatId`, `messageId`, `content`)
-   `createForumTopic` (`chatId`, `name`, オプション `iconColor`, `iconCustomEmojiId`)

チャネルメッセージアクションは使いやすいエイリアスを公開します (`send`, `react`, `delete`, `edit`, `sticker`, `sticker-search`, `topic-create`)。ゲーティング制御:

-   `channels.telegram.actions.sendMessage`
-   `channels.telegram.actions.deleteMessage`
-   `channels.telegram.actions.reactions`
-   `channels.telegram.actions.sticker` (デフォルト: 無効)

注意: `edit` と `topic-create` は現在デフォルトで有効で、個別の `channels.telegram.actions.*` トグルはありません。リアクション削除セマンティクス: [/tools/reactions](../tools/reactions.md)

Telegram は生成された出力で明示的な返信スレッドタグをサポートします:

-   `[[reply_to_current]]` はトリガーメッセージに返信します
-   `[[reply_to:]]` は特定の Telegram メッセージ ID に返信します

`channels.telegram.replyToMode` が処理を制御します:

-   `off` (デフォルト)
-   `first`
-   `all`

注意: `off` は暗黙的な返信スレッドを無効にします。明示的な `[[reply_to_*]]` タグは引き続き尊重されます。

フォーラムスーパーグループ:

-   トピックセッションキーは `:topic:` を追加します
-   返信とタイピングはトピックスレッドをターゲットにします
-   トピック設定パス: `channels.telegram.groups..topics.`

一般トピック (`threadId=1`) 特別ケース:

-   メッセージ送信は `message_thread_id` を省略します (Telegram は `sendMessage(...thread_id=1)` を拒否します)
-   タイピングアクションは引き続き `message_thread_id` を含みます

トピック継承: トピックエントリはオーバーライドされない限りグループ設定を継承します (`requireMention`, `allowFrom`, `skills`, `systemPrompt`, `enabled`, `groupPolicy`)。 `agentId` はトピック専用で、グループデフォルトから継承しません。**トピックごとのエージェントルーティング**: 各トピックは、トピック設定で `agentId` を設定することで異なるエージェントにルーティングできます。これにより、各トピックが独自の分離されたワークスペース、メモリ、セッションを持ちます。例:

```json
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          topics: {
            "1": { agentId: "main" },      // 一般トピック → main エージェント
            "3": { agentId: "zu" },        // 開発トピック → zu エージェント
            "5": { agentId: "coder" }      // コードレビュー → coder エージェント
          }
        }
      }
    }
  }
}
```

各トピックは独自のセッションキーを持ちます: `agent:zu:telegram:group:-1001234567890:topic:3`**永続的 ACP トピックバインディング**: フォーラムトピックは、トップレベルの型付き ACP バインディングを通じて ACP ハーネスセッションを固定できます:

-   `type: "acp"` と `match.channel: "telegram"` を持つ `bindings[]`

例:

```json
{
  agents: {
    list: [
      {
        id: "codex",
        runtime: {
          type: "acp",
          acp: {
            agent: "codex",
            backend: "acpx",
            mode: "persistent",
            cwd: "/workspace/openclaw",
          },
        },
      },
    ],
  },
  bindings: [
    {
      type: "acp",
      agentId: "codex",
      match: {
        channel: "telegram",
        accountId: "default",
        peer: { kind: "group", id: "-1001234567890:topic:42" },
      },
    },
  ],
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          topics: {
            "42": {
              requireMention: false,
            },
          },
        },
      },
    },
  },
}
```

これは現在、グループおよびスーパーグループ内のフォーラムトピックにスコープされています。**チャットからのスレッドバインド ACP スポーン**:

-   `/acp spawn  --thread here|auto` は現在の Telegram トピックを新しい ACP セッションにバインドできます。
-   フォローアップのトピックメッセージは、バインドされた ACP セッションに直接ルーティングされます (`/acp steer` は不要)。
-   OpenClaw は成功したバインド後、トピック内でスポーン確認メッセージを固定します。
-   `channels.telegram.threadBindings.spawnAcpSessions=true` が必要です。

テンプレートコンテキストには以下が含まれます:

-   `MessageThreadId`
-   `IsForum`

DM スレッド動作:

-   `message_thread_id` を持つプライベートチャットは DM ルーティングを保持しますが、スレッドを認識したセッションキー/返信ターゲットを使用します。

### 音声メッセージ

Telegram はボイスノートとオーディオファイルを区別します。

-   デフォルト: オーディオファイル動作
-   エージェント返信にタグ `[[audio_as_voice]]` を付けてボイスノート送信を強制

メッセージアクション例:

```json
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

### 動画メッセージ

Telegram は動画ファイルと動画ノートを区別します。メッセージアクション例:

```json
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

動画ノートはキャプションをサポートしません。提供されたメッセージテキストは別途送信されます。

### ステッカー

受信ステッカー処理:

-   静的 WEBP: ダウンロードされ処理されます (プレースホルダ `<media:sticker>`)
-   アニメーション TGS: スキップ
-   動画 WEBM: スキップ

ステッカーコンテキストフィールド:

-   `Sticker.emoji`
-   `Sticker.setName`
-   `Sticker.fileId`
-   `Sticker.fileUniqueId`
-   `Sticker.cachedDescription`

ステッカーキャッシュファイル:

-   `~/.openclaw/telegram/sticker-cache.json`

ステッカーは可能な場合に一度だけ記述され、繰り返しのビジョン呼び出しを減らすためにキャッシュされます。ステッカーアクションを有効化:

```json
{
  channels: {
    telegram: {
      actions: {
        sticker: true,
      },
    },
  },
}
```

ステッカー送信アクション:

```json
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

キャッシュされたステッカーを検索:

```json
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

Telegram リアクションは `message_reaction` 更新として到着します（メッセージペイロードとは別）。有効になっている場合、OpenClaw は以下のようなシステムイベントをエンキューします:

-   `Telegram reaction added: 👍 by Alice (@alice) on msg 42`

設定:

-   `channels.telegram.reactionNotifications`: `off | own | all` (デフォルト: `own`)
-   `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` (デフォルト: `minimal`)

注意:

-   `own` は、ボットが送信したメッセージに対するユーザーリアクションのみを意味します（送信メッセージキャッシュによるベストエフォート）。
-   リアクションイベントは引き続き Telegram アクセス制御を尊重します (`dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`)。許可されていない送信者は破棄されます。
-   Telegram はリアクション更新でスレッド ID を提供しません。
    -   非フォーラムグループはグループチャットセッションにルーティングされます
    -   フォーラムグループは正確な発信トピックではなく、グループの一般トピックセッション (`:topic:1`) にルーティングされます

ポーリング/webhook の `allowed_updates` には自動的に `message_reaction` が含まれます。

`ackReaction` は、OpenClaw が受信メッセージを処理している間に確認絵文字を送信します。解決順序:

-   `channels.telegram.accounts..ackReaction`
-   `channels.telegram.ackReaction`
-   `messages.ackReaction`
-   エージェントアイデンティティ絵文字フォールバック (`agents.list[].identity.emoji`, それ以外は ”👀”)

注意:

-   Telegram は Unicode 絵文字を期待します（例: ”👀”）。
-   チャネルまたはアカウントのリアクションを無効にするには `""` を使用します。

チャネル設定書き込みはデフォルトで有効です (`configWrites !== false`)。Telegram トリガーによる書き込みには以下が含まれます:

-   グループ移行イベント (`migrate_to_chat_id`) による `channels.telegram.groups` の更新
-   `/config set` と `/config unset` (コマンド有効化が必要)

無効化:

```json
{
  channels: {
    telegram: {
      configWrites: false,
    },
  },
}
```

デフォルト: ロングポーリング。Webhook モード:

-   `channels.telegram.webhookUrl` を設定
-   `channels.telegram.webhookSecret` を設定 (webhook URL 設定時必須)
-   オプション `channels.telegram.webhookPath` (デフォルト `/telegram-webhook`)
-   オプション `channels.telegram.webhookHost` (デフォルト `127.0.0.1`)
-   オプション `channels.telegram.webhookPort` (デフォルト `8787`)

Webhook モードのデフォルトローカルリスナーは `127.0.0.1:8787` にバインドします。パブリックエンドポイントが異なる場合は、リバースプロキシを前に配置し、`webhookUrl` をパブリック URL に向けます。外部イングレスを意図的に必要とする場合は `webhookHost` (例: `0.0.0.0`) を設定します。

-   `channels.telegram.textChunkLimit` デフォルトは 4000。
-   `channels.telegram.chunkMode="newline"` は長さ分割の前に段落境界（空行）を優先します。
-   `channels.telegram.mediaMaxMb` (デフォルト 100) は受信および送信 Telegram メディアサイズを制限します。
-   `channels.telegram.timeoutSeconds` は Telegram API クライアントタイムアウトをオーバーライドします（未設定の場合、grammY デフォルトが適用されます）。
-   グループコンテキスト履歴は `channels.telegram.historyLimit` または `messages.groupChat.historyLimit` (デフォルト 50) を使用します。`0` は無効化します。
-   DM 履歴制御:
    -   `channels.telegram.dmHistoryLimit`
    -   `channels.telegram.dms["<user_id>"].historyLimit`
-   `channels.telegram.retry` 設定は、回復可能な送信 API エラーに対して Telegram 送信ヘルパー (CLI/ツール/アクション) に適用されます。

CLI 送信ターゲットは数値チャット ID またはユーザー名:

```bash
openclaw message send --channel telegram --target 123456789 --message "hi"
openclaw message send --channel telegram --target @name --message "hi"
```

Telegram 投票は `openclaw message poll` を使用し、フォーラムトピックをサポートします:

```bash
openclaw message poll --channel telegram --target 123456789 \
  --poll-question "Ship it?" --poll-option "Yes" --poll-option "No"
openclaw message poll --channel telegram --target -1001234567890:topic:42 \
  --poll-question "Pick a time" --poll-option "10am" --poll-option "2pm" \
  --poll-duration-seconds 300 --poll-public
```

Telegram 専用投票フラグ:

-   `--poll-duration-seconds` (5-600)
-   `--poll-anonymous`
-   `--poll-public`
-   フォーラムトピック用の `--thread-id` (または `:topic:` ターゲットを使用)

アクションゲーティング:

-   `channels.telegram.actions.sendMessage=false` は投票を含む送信 Telegram メッセージを無効にします
-   `channels.telegram.actions.poll=false` は通常の送信を有効にしたまま Telegram 投票作成を無効にします

## トラブルシューティング

-   `requireMention=false` の場合、Telegram プライバシーモードが完全な可視性を許可する必要があります。
    -   BotFather: `/setprivacy` -> 無効化
    -   その後、ボットをグループから削除して再追加
-   `openclaw channels status` は、設定がメンションなしグループメッセージを期待する場合に警告します。
-   `openclaw channels status --probe` は明示的な数値グループ ID をチェックできます。ワイルドカード `"*"` はメンバーシッププローブできません。
-   クイックセッションテスト: `/activation always`。

-   `channels.telegram.groups` が存在する場合、グループはリストされている必要があります（または `"*"` を含む）
-   グループ内のボットメンバーシップを確認
-   ログを確認: `openclaw logs --follow` でスキップ理由を確認

-   送信者アイデンティティを承認（ペアリングおよび/または数値 `allowFrom`）
-   グループポリシーが `open` であってもコマンド承認は適用されます
-   `setMyCommands failed` は通常、`api.telegram.org` への DNS/HTTPS 到達性の問題を示します

-   Node 22+ + カスタム fetch/プロキシは、AbortSignal タイプが一致しない場合、即時中止動作を引き起こす可能性があります。
-   一部のホストは `api.telegram.org` を IPv6 優先で解決します。壊れた IPv6 出力は断続的な Telegram API 失敗を引き起こす可能性があります。
-   ログに `TypeError: fetch failed` または `Network request for 'getUpdates' failed!` が含まれる場合、OpenClaw はこれらを回復可能なネットワークエラーとして再試行します。
-   不安定な直接出力/TLS を持つ VPS ホストでは、Telegram API 呼び出しを `channels.telegram.proxy` 経由でルーティング:

```yaml
channels:
  telegram:
    proxy: socks5://<user>:<password>@proxy-host:1080
```

-   Node 22+ はデフォルトで `autoSelectFamily=true` (WSL2 を除く) および `dnsResultOrder=ipv4first` です。
-   ホストが WSL2 の場合、または明示的に IPv4 のみの動作でより良く動作する場合は、ファミリー選択を強制:

```yaml
channels:
  telegram:
    network:
      autoSelectFamily: false
```

-   環境変数オーバーライド (一時的):
    -   `OPENCLAW_TELEGRAM_DISABLE_AUTO_SELECT_FAMILY=1`
    -   `OPENCLAW_TELEGRAM_ENABLE_AUTO_SELECT_FAMILY=1`
    -   `OPENCLAW_TELEGRAM_DNS_RESULT_ORDER=ipv4first`
-   DNS 応答を検証:

```bash
dig +short api.telegram.org A
dig +short api.telegram.org AAAA
```

 さらなるヘルプ: [チャネルトラブルシューティング](./troubleshooting.md)。

## Telegram 設定リファレンスポインタ

主要リファレンス:

-   `channels.telegram.enabled`: チャネル起動の有効/無効。
-   `channels.telegram.botToken`: ボットトークン (BotFather)。
-   `channels.telegram.tokenFile`: ファイルパスからトークンを読み込み。
-   `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (デフォルト: pairing)。
-   `channels.telegram.allowFrom`: DM 許可リスト (数値 Telegram ユーザー ID)。 `allowlist` には少なくとも1つの送信者 ID が必要です。 `open` には `"*"` が必要です。 `openclaw doctor --fix` はレガシーな `@username` エントリを ID に解決でき、許可リスト移行フローでペアリングストアファイルから許可リストエントリを回復できます。
-   `channels.telegram.actions.poll`: Telegram 投票作成の有効/無効 (デフォルト: 有効; 引き続き `sendMessage` が必要)。
-   `channels.telegram.defaultTo`: 明示的な `--reply-to` が提供されない場合に CLI `--deliver` で使用されるデフォルト Telegram ターゲット。
-   `channels.telegram.groupPolicy`: `open | allowlist | disabled` (デフォルト: allowlist)。
-   `channels.tele