

  CLI コマンド

  
# message

Discord/Google Chat/Slack/Mattermost (プラグイン)/Telegram/WhatsApp/Signal/iMessage/MS Teams 向けの、メッセージ送信とチャネルアクションのための単一出力コマンド。

## 使用方法

```bash
openclaw message <サブコマンド> [フラグ]
```

チャネル選択:

-   `--channel` 複数のチャネルが設定されている場合は必須。
-   設定されているチャネルがちょうど1つの場合、それがデフォルトになります。
-   値: `whatsapp|telegram|discord|googlechat|slack|mattermost|signal|imessage|msteams` (Mattermost はプラグインが必要)

ターゲット形式 (`--target`):

-   WhatsApp: E.164 またはグループ JID
-   Telegram: チャット ID または `@ユーザー名`
-   Discord: `channel:` または `user:` (または `<@id>` メンション; 生の数値 ID はチャネルとして扱われます)
-   Google Chat: `spaces/` または `users/`
-   Slack: `channel:` または `user:` (生のチャネル ID が受け入れられます)
-   Mattermost (プラグイン): `channel:`, `user:`, または `@ユーザー名` (単独の ID はチャネルとして扱われます)
-   Signal: `+E.164`, `group:`, `signal:+E.164`, `signal:group:`, または `username:<名前>`/`u:<名前>`
-   iMessage: ハンドル, `chat_id:`, `chat_guid:`, または `chat_identifier:`
-   MS Teams: 会話 ID (`19:...@thread.tacv2`) または `conversation:` または `user:<aad-object-id>`

名前解決:

-   サポートされているプロバイダー (Discord/Slack など) では、`Help` や `#help` などのチャネル名はディレクトリキャッシュを介して解決されます。
-   キャッシュミスの場合、OpenClaw はプロバイダーがサポートしている場合、ライブディレクトリ検索を試みます。

## 共通フラグ

-   `--channel <名前>`
-   `--account `
-   `--target <宛先>` (送信/投票/読み取りなどのターゲットチャネルまたはユーザー)
-   `--targets <名前>` (繰り返し; ブロードキャスト専用)
-   `--json`
-   `--dry-run`
-   `--verbose`

## アクション

### コア

-   `send`
    -   チャネル: WhatsApp/Telegram/Discord/Google Chat/Slack/Mattermost (プラグイン)/Signal/iMessage/MS Teams
    -   必須: `--target`、および `--message` または `--media`
    -   オプション: `--media`, `--reply-to`, `--thread-id`, `--gif-playback`
    -   Telegram のみ: `--buttons` (`channels.telegram.capabilities.inlineButtons` を有効にする必要があります)
    -   Telegram のみ: `--thread-id` (フォーラムトピック ID)
    -   Slack のみ: `--thread-id` (スレッドタイムスタンプ; `--reply-to` も同じフィールドを使用します)
    -   WhatsApp のみ: `--gif-playback`
-   `poll`
    -   チャネル: WhatsApp/Telegram/Discord/Matrix/MS Teams
    -   必須: `--target`, `--poll-question`, `--poll-option` (繰り返し)
    -   オプション: `--poll-multi`
    -   Discord のみ: `--poll-duration-hours`, `--silent`, `--message`
    -   Telegram のみ: `--poll-duration-seconds` (5-600), `--silent`, `--poll-anonymous` / `--poll-public`, `--thread-id`
-   `react`
    -   チャネル: Discord/Google Chat/Slack/Telegram/WhatsApp/Signal
    -   必須: `--message-id`, `--target`
    -   オプション: `--emoji`, `--remove`, `--participant`, `--from-me`, `--target-author`, `--target-author-uuid`
    -   注記: `--remove` には `--emoji` が必要です (`--emoji` を省略すると、サポートされている場合に自身のリアクションをクリアします; /tools/reactions を参照)
    -   WhatsApp のみ: `--participant`, `--from-me`
    -   Signal グループリアクション: `--target-author` または `--target-author-uuid` が必要
-   `reactions`
    -   チャネル: Discord/Google Chat/Slack
    -   必須: `--message-id`, `--target`
    -   オプション: `--limit`
-   `read`
    -   チャネル: Discord/Slack
    -   必須: `--target`
    -   オプション: `--limit`, `--before`, `--after`
    -   Discord のみ: `--around`
-   `edit`
    -   チャネル: Discord/Slack
    -   必須: `--message-id`, `--message`, `--target`
-   `delete`
    -   チャネル: Discord/Slack/Telegram
    -   必須: `--message-id`, `--target`
-   `pin` / `unpin`
    -   チャネル: Discord/Slack
    -   必須: `--message-id`, `--target`
-   `pins` (リスト)
    -   チャネル: Discord/Slack
    -   必須: `--target`
-   `permissions`
    -   チャネル: Discord
    -   必須: `--target`
-   `search`
    -   チャネル: Discord
    -   必須: `--guild-id`, `--query`
    -   オプション: `--channel-id`, `--channel-ids` (繰り返し), `--author-id`, `--author-ids` (繰り返し), `--limit`

### スレッド

-   `thread create`
    -   チャネル: Discord
    -   必須: `--thread-name`, `--target` (チャネル ID)
    -   オプション: `--message-id`, `--message`, `--auto-archive-min`
-   `thread list`
    -   チャネル: Discord
    -   必須: `--guild-id`
    -   オプション: `--channel-id`, `--include-archived`, `--before`, `--limit`
-   `thread reply`
    -   チャネル: Discord
    -   必須: `--target` (スレッド ID), `--message`
    -   オプション: `--media`, `--reply-to`

### 絵文字

-   `emoji list`
    -   Discord: `--guild-id`
    -   Slack: 追加フラグなし
-   `emoji upload`
    -   チャネル: Discord
    -   必須: `--guild-id`, `--emoji-name`, `--media`
    -   オプション: `--role-ids` (繰り返し)

### ステッカー

-   `sticker send`
    -   チャネル: Discord
    -   必須: `--target`, `--sticker-id` (繰り返し)
    -   オプション: `--message`
-   `sticker upload`
    -   チャネル: Discord
    -   必須: `--guild-id`, `--sticker-name`, `--sticker-desc`, `--sticker-tags`, `--media`

### ロール / チャネル / メンバー / ボイス

-   `role info` (Discord): `--guild-id`
-   `role add` / `role remove` (Discord): `--guild-id`, `--user-id`, `--role-id`
-   `channel info` (Discord): `--target`
-   `channel list` (Discord): `--guild-id`
-   `member info` (Discord/Slack): `--user-id` (+ Discord の場合は `--guild-id`)
-   `voice status` (Discord): `--guild-id`, `--user-id`

### イベント

-   `event list` (Discord): `--guild-id`
-   `event create` (Discord): `--guild-id`, `--event-name`, `--start-time`
    -   オプション: `--end-time`, `--desc`, `--channel-id`, `--location`, `--event-type`

### モデレーション (Discord)

-   `timeout`: `--guild-id`, `--user-id` (オプション `--duration-min` または `--until`; 両方省略でタイムアウト解除)
-   `kick`: `--guild-id`, `--user-id` (+ `--reason`)
-   `ban`: `--guild-id`, `--user-id` (+ `--delete-days`, `--reason`)
    -   `timeout` も `--reason` をサポートします

### ブロードキャスト

-   `broadcast`
    -   チャネル: 設定済みの任意のチャネル; `--channel all` を使用してすべてのプロバイダーを対象にできます
    -   必須: `--targets` (繰り返し)
    -   オプション: `--message`, `--media`, `--dry-run`

## 例

Discord で返信を送信:

```bash
openclaw message send --channel discord \
  --target channel:123 --message "hi" --reply-to 456
```

コンポーネント付きの Discord メッセージを送信:

```bash
openclaw message send --channel discord \
  --target channel:123 --message "Choose:" \
  --components '{"text":"Choose a path","blocks":[{"type":"actions","buttons":[{"label":"Approve","style":"success"},{"label":"Decline","style":"danger"}]}]}'
```

完全なスキーマについては [Discord コンポーネント](../channels/discord.md#interactive-components) を参照してください。Discord 投票を作成:

```bash
openclaw message poll --channel discord \
  --target channel:123 \
  --poll-question "Snack?" \
  --poll-option Pizza --poll-option Sushi \
  --poll-multi --poll-duration-hours 48
```

Telegram 投票を作成 (2分後に自動終了):

```bash
openclaw message poll --channel telegram \
  --target @mychat \
  --poll-question "Lunch?" \
  --poll-option Pizza --poll-option Sushi \
  --poll-duration-seconds 120 --silent
```

Teams プロアクティブメッセージを送信:

```bash
openclaw message send --channel msteams \
  --target conversation:19:abc@thread.tacv2 --message "hi"
```

Teams 投票を作成:

```bash
openclaw message poll --channel msteams \
  --target conversation:19:abc@thread.tacv2 \
  --poll-question "Lunch?" \
  --poll-option Pizza --poll-option Sushi
```

Slack でリアクション:

```bash
openclaw message react --channel slack \
  --target C123 --message-id 456 --emoji "✅"
```

Signal グループでリアクション:

```bash
openclaw message react --channel signal \
  --target signal:group:abc123 --message-id 1737630212345 \
  --emoji "✅" --target-author-uuid 123e4567-e89b-12d3-a456-426614174000
```

Telegram インラインボタンを送信:

```bash
openclaw message send --channel telegram --target @mychat --message "Choose:" \
  --buttons '[ [{"text":"Yes","callback_data":"cmd:yes"}], [{"text":"No","callback_data":"cmd:no"}] ]'
```

[memory](./memory.md)[models](./models.md)