

  メッセージングプラットフォーム

  
# Slack

ステータス: Slackアプリ連携によるDMおよびチャンネル対応で本番環境対応済み。デフォルトモードはソケットモード。HTTP Events APIモードもサポートされています。

## クイックセットアップ

## トークンモデル

-   `botToken` + `appToken` はソケットモードで必須です。
-   HTTPモードでは `botToken` + `signingSecret` が必要です。
-   設定ファイルのトークンは環境変数のフォールバックを上書きします。
-   `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` 環境変数のフォールバックはデフォルトアカウントにのみ適用されます。
-   `userToken` (`xoxp-...`) は設定ファイル専用（環境変数フォールバックなし）で、デフォルトは読み取り専用動作です (`userTokenReadOnly: true`)。
-   オプション: 送信メッセージがアクティブなエージェントのアイデンティティ（カスタム `username` とアイコン）を使用する場合は、`chat:write.customize` を追加します。`icon_emoji` は `:emoji_name:` 構文を使用します。

> **💡** アクション/ディレクトリ読み取りでは、設定されている場合、ユーザートークンが優先されることがあります。書き込みでは、ボットトークンが引き続き優先されます。ユーザートークンによる書き込みは、`userTokenReadOnly: false` かつボットトークンが利用できない場合にのみ許可されます。

## アクセス制御とルーティング

`channels.slack.dmPolicy` はDMアクセスを制御します（レガシー: `channels.slack.dm.policy`）:

-   `pairing` (デフォルト)
-   `allowlist`
-   `open` (`channels.slack.allowFrom` に `"*"` を含む必要があります。レガシー: `channels.slack.dm.allowFrom`)
-   `disabled`

DMフラグ:

-   `dm.enabled` (デフォルト true)
-   `channels.slack.allowFrom` (推奨)
-   `dm.allowFrom` (レガシー)
-   `dm.groupEnabled` (グループDMはデフォルト false)
-   `dm.groupChannels` (オプションのMPIM許可リスト)

マルチアカウントの優先順位:

-   `channels.slack.accounts.default.allowFrom` は `default` アカウントにのみ適用されます。
-   名前付きアカウントは、自身の `allowFrom` が設定されていない場合、`channels.slack.allowFrom` を継承します。
-   名前付きアカウントは `channels.slack.accounts.default.allowFrom` を継承しません。

DMでのペアリングは `openclaw pairing approve slack ` を使用します。

`channels.slack.groupPolicy` はチャンネル処理を制御します:

-   `open`
-   `allowlist`
-   `disabled`

チャンネル許可リストは `channels.slack.channels` の下にあります。実行時注意: `channels.slack` が完全に欠落している場合（環境変数のみのセットアップ）、ランタイムは `groupPolicy="allowlist"` にフォールバックし、警告をログに記録します（`channels.defaults.groupPolicy` が設定されていても）。名前/ID解決:

-   チャンネル許可リストエントリとDM許可リストエントリは、トークンアクセスが許可されている場合、起動時に解決されます。
-   解決されなかったエントリは設定されたまま保持されます。
-   インバウンド認証マッチングはデフォルトでID優先です。直接のユーザー名/スラッグマッチングには `channels.slack.dangerouslyAllowNameMatching: true` が必要です。

チャンネルメッセージはデフォルトでメンションゲートされています。メンションソース:

-   明示的なアプリメンション (`<@botId>`)
-   メンション正規表現パターン (`agents.list[].groupChat.mentionPatterns`, フォールバック `messages.groupChat.mentionPatterns`)
-   暗黙的なボットへのスレッド返信動作

チャンネルごとの制御 (`channels.slack.channels.<id|name>`):

-   `requireMention`
-   `users` (許可リスト)
-   `allowBots`
-   `skills`
-   `systemPrompt`
-   `tools`, `toolsBySender`
-   `toolsBySender` キー形式: `id:`, `e164:`, `username:`, `name:`, または `"*"` ワイルドカード（レガシーの接頭辞なしキーは `id:` のみにマッピングされます）

## コマンドとスラッシュ動作

-   ネイティブコマンド自動モードはSlackでは**オフ**です (`commands.native: "auto"` はSlackネイティブコマンドを有効にしません)。
-   `channels.slack.commands.native: true` (またはグローバル `commands.native: true`) でネイティブSlackコマンドハンドラーを有効にします。
-   ネイティブコマンドが有効な場合、Slackで一致するスラッシュコマンドを登録します (`/` 名)。ただし1つの例外があります:
    -   ステータスコマンドには `/agentstatus` を登録します (Slackは `/status` を予約しています)。
-   ネイティブコマンドが有効でない場合、`channels.slack.slashCommand` を介して単一の設定済みスラッシュコマンドを実行できます。
-   ネイティブ引数メニューはレンダリング戦略を適応させます:
    -   最大5オプション: ボタンブロック
    -   6-100オプション: 静的選択メニュー
    -   100を超えるオプション: インタラクティビティオプションハンドラーが利用可能な場合、非同期オプションフィルタリング付き外部選択
    -   エンコードされたオプション値がSlack制限を超える場合、フローはボタンにフォールバックします。
-   長いオプションペイロードの場合、スラッシュコマンド引数メニューは選択された値をディスパッチする前に確認ダイアログを使用します。

デフォルトのスラッシュコマンド設定:

-   `enabled: false`
-   `name: "openclaw"`
-   `sessionPrefix: "slack:slash"`
-   `ephemeral: true`

スラッシュセッションは分離されたキーを使用します:

-   `agent::slack:slash:`

また、コマンド実行はターゲット会話セッション (`CommandTargetSessionKey`) に対してルーティングされます。

## スレッド、セッション、返信タグ

-   DMは `direct` としてルーティングされます。チャンネルは `channel`、MPIMは `group` としてルーティングされます。
-   デフォルトの `session.dmScope=main` では、Slack DMはエージェントのメインセッションに折りたたまれます。
-   チャンネルセッション: `agent::slack:channel:`。
-   スレッド返信は、該当する場合、スレッドセッションサフィックス (`:thread:`) を作成できます。
-   `channels.slack.thread.historyScope` のデフォルトは `thread` です。`thread.inheritParent` のデフォルトは `false` です。
-   `channels.slack.thread.initialHistoryLimit` は、新しいスレッドセッションが開始されたときに取得される既存のスレッドメッセージの数を制御します（デフォルト `20`。無効にするには `0` を設定）。

返信スレッド制御:

-   `channels.slack.replyToMode`: `off|first|all` (デフォルト `off`)
-   `channels.slack.replyToModeByChatType`: `direct|group|channel` ごと
-   ダイレクトチャット用レガシーフォールバック: `channels.slack.dm.replyToMode`

手動返信タグがサポートされています:

-   `[[reply_to_current]]`
-   `[[reply_to:]]`

注意: `replyToMode="off"` は、明示的な `[[reply_to_*]]` タグを含む、Slackでの**すべての**返信スレッド化を無効にします。これは、明示的なタグが `"off"` モードでも尊重されるTelegramとは異なります。この違いはプラットフォームのスレッドモデルを反映しています。Slackスレッドはメッセージをチャンネルから隠しますが、Telegramの返信はメインチャットフローに表示されたままです。

## メディア、チャンク分割、配信

Slackファイル添付ファイルは、SlackがホストするプライベートURL（トークン認証済みリクエストフロー）からダウンロードされ、フェッチが成功しサイズ制限が許容される場合、メディアストアに書き込まれます。ランタイムインバウンドサイズ上限は、`channels.slack.mediaMaxMb` で上書きされない限り、デフォルトで `20MB` です。

-   テキストチャンクは `channels.slack.textChunkLimit` を使用します（デフォルト 4000）
-   `channels.slack.chunkMode="newline"` は段落優先分割を有効にします
-   ファイル送信はSlackアップロードAPIを使用し、スレッド返信 (`thread_ts`) を含めることができます
-   アウトバウンドメディア上限は設定されている場合 `channels.slack.mediaMaxMb` に従います。それ以外の場合、チャンネル送信はメディアパイプラインからのMIME種別デフォルトを使用します

推奨される明示的ターゲット:

-   DM用 `user:`
-   チャンネル用 `channel:`

Slack DMは、ユーザーターゲットに送信する際にSlack会話APIを介して開かれます。

## アクションとゲート

Slackアクションは `channels.slack.actions.*` で制御されます。現在のSlackツーリングで利用可能なアクショングループ:

| グループ | デフォルト |
| --- | --- |
| messages | enabled |
| reactions | enabled |
| pins | enabled |
| memberInfo | enabled |
| emojiList | enabled |

## イベントと運用動作

-   メッセージ編集/削除/スレッドブロードキャストはシステムイベントにマッピングされます。
-   リアクション追加/削除イベントはシステムイベントにマッピングされます。
-   メンバー参加/退出、チャンネル作成/名前変更、ピン追加/削除イベントはシステムイベントにマッピングされます。
-   アシスタントスレッドステータス更新（スレッド内の「入力中…」インジケーター用）は `assistant.threads.setStatus` を使用し、ボットスコープ `assistant:write` が必要です。
-   `channel_id_changed` は、`configWrites` が有効な場合、チャンネル設定キーを移行できます。
-   チャンネルトピック/目的メタデータは信頼できないコンテキストとして扱われ、ルーティングコンテキストに注入される可能性があります。
-   ブロックアクションとモーダルインタラクションは、豊富なペイロードフィールドを持つ構造化された `Slack interaction: ...` システムイベントを発行します:
    -   ブロックアクション: 選択された値、ラベル、ピッカー値、`workflow_*` メタデータ
    -   ルーティングされたチャンネルメタデータとフォーム入力を持つモーダル `view_submission` および `view_closed` イベント

## Ackリアクション

`ackReaction` は、OpenClawがインバウンドメッセージを処理している間に確認用の絵文字を送信します。解決順序:

-   `channels.slack.accounts..ackReaction`
-   `channels.slack.ackReaction`
-   `messages.ackReaction`
-   エージェントアイデンティティ絵文字フォールバック (`agents.list[].identity.emoji`, それ以外は ”👀”)

注意:

-   Slackはショートコードを期待します（例: `"eyes"`）。
-   Slackアカウントまたはグローバルにリアクションを無効にするには `""` を使用します。

## タイピングリアクションフォールバック

`typingReaction` は、OpenClawが返信を処理している間、インバウンドSlackメッセージに一時的なリアクションを追加し、実行が終了すると削除します。これは、特にDMで、Slackネイティブのアシスタントタイピングが利用できない場合に便利なフォールバックです。解決順序:

-   `channels.slack.accounts..typingReaction`
-   `channels.slack.typingReaction`

注意:

-   Slackはショートコードを期待します（例: `"hourglass_flowing_sand"`）。
-   リアクションはベストエフォートであり、返信または失敗パス完了後に自動的にクリーンアップが試みられます。

## マニフェストとスコープチェックリスト

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "Slack connector for OpenClaw"
  },
  "features": {
    "bot_user": {
      "display_name": "OpenClaw",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "Send a message to OpenClaw",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "im:history",
        "im:read",
        "im:write",
        "mpim:history",
        "mpim:read",
        "mpim:write",
        "users:read",
        "app_mentions:read",
        "assistant:write",
        "reactions:read",
        "reactions:write",
        "pins:read",
        "pins:write",
        "emoji:read",
        "commands",
        "files:read",
        "files:write"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "reaction_added",
        "reaction_removed",
        "member_joined_channel",
        "member_left_channel",
        "channel_rename",
        "pin_added",
        "pin_removed"
      ]
    }
  }
}
```

`channels.slack.userToken` を設定する場合、典型的な読み取りスコープは:

-   `channels:history`, `groups:history`, `im:history`, `mpim:history`
-   `channels:read`, `groups:read`, `im:read`, `mpim:read`
-   `users:read`
-   `reactions:read`
-   `pins:read`
-   `emoji:read`
-   `search:read` (Slack検索読み取りに依存する場合)

## トラブルシューティング

順番に確認:

-   `groupPolicy`
-   チャンネル許可リスト (`channels.slack.channels`)
-   `requireMention`
-   チャンネルごとの `users` 許可リスト

便利なコマンド:

```bash
openclaw channels status --probe
openclaw logs --follow
openclaw doctor
```

確認:

-   `channels.slack.dm.enabled`
-   `channels.slack.dmPolicy` (またはレガシー `channels.slack.dm.policy`)
-   ペアリング承認 / 許可リストエントリ

```bash
openclaw pairing list slack
```

Slackアプリ設定でボットトークン + アプリトークンとソケットモード有効化を検証します。

検証:

-   署名シークレット
-   Webhookパス
-   SlackリクエストURL（イベント + インタラクティビティ + スラッシュコマンド）
-   HTTPアカウントごとの一意の `webhookPath`

意図したものを確認:

-   ネイティブコマンドモード (`channels.slack.commands.native: true`) とSlackに登録された一致するスラッシュコマンド
-   または単一スラッシュコマンドモード (`channels.slack.slashCommand.enabled: true`)

また、`commands.useAccessGroups` とチャンネル/ユーザー許可リストも確認してください。

## テキストストリーミング

OpenClawは、Agents and AI Apps APIを介したSlackネイティブテキストストリーミングをサポートしています。`channels.slack.streaming` はライブプレビュー動作を制御します:

-   `off`: ライブプレビューストリーミングを無効にします。
-   `partial` (デフォルト): プレビューテキストを最新の部分出力で置き換えます。
-   `block`: チャンク化されたプレビュー更新を追加します。
-   `progress`: 生成中に進行状況ステータステキストを表示し、最終テキストを送信します。

`channels.slack.nativeStreaming` は、`streaming` が `partial` の場合にSlackのネイティブストリーミングAPI (`chat.startStream` / `chat.appendStream` / `chat.stopStream`) を制御します（デフォルト: `true`）。ネイティブSlackストリーミングを無効にする（ドラフトプレビュー動作を維持）:

```yaml
channels:
  slack:
    streaming: partial
    nativeStreaming: false
```

レガシーキー:

-   `channels.slack.streamMode` (`replace | status_final | append`) は自動的に `channels.slack.streaming` に移行されます。
-   ブール値 `channels.slack.streaming` は自動的に `channels.slack.nativeStreaming` に移行されます。

### 要件

1.  Slackアプリ設定で **Agents and AI Apps** を有効にします。
2.  アプリに `assistant:write` スコープがあることを確認します。
3.  そのメッセージに対して返信スレッドが利用可能である必要があります。スレッド選択は引き続き `replyToMode` に従います。

### 動作

-   最初のテキストチャンクでストリームを開始します (`chat.startStream`)。
-   後のテキストチャンクは同じストリームに追加されます (`chat.appendStream`)。
-   返信終了でストリームを確定します (`chat.stopStream`)。
-   メディアおよび非テキストペイロードは通常の配信にフォールバックします。
-   ストリーミングが返信中に失敗した場合、OpenClawは残りのペイロードに対して通常の配信にフォールバックします。

## 設定リファレンスポインタ

主なリファレンス:

-   [設定リファレンス - Slack](../gateway/configuration-reference.md#slack) 高シグナルのSlackフィールド:
    -   モード/認証: `mode`, `botToken`, `appToken`, `signingSecret`, `webhookPath`, `accounts.*`
    -   DMアクセス: `dm.enabled`, `dmPolicy`, `allowFrom` (レガシー: `dm.policy`, `dm.allowFrom`), `dm.groupEnabled`, `dm.groupChannels`
    -   互換性トグル: `dangerouslyAllowNameMatching` (非常用; 必要な場合を除きオフのまま)
    -   チャンネルアクセス: `groupPolicy`, `channels.*`, `channels.*.users`, `channels.*.requireMention`
    -   スレッド/履歴: `replyToMode`, `replyToModeByChatType`, `thread.*`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
    -   配信: `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `streaming`, `nativeStreaming`
    -   運用/機能: `configWrites`, `commands.native`, `slashCommand.*`, `actions.*`, `userToken`, `userTokenReadOnly`

## 関連項目

-   [ペアリング](./pairing.md)
-   [チャンネルルーティング](./channel-routing.md)
-   [トラブルシューティング](./troubleshooting.md)
-   [設定](../gateway/configuration.md)
-   [スラッシュコマンド](../tools/slash-commands.md)

[Synology Chat](./synology-chat.md)[Telegram](./telegram.md)