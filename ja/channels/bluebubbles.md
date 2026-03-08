

  メッセージングプラットフォーム

  
# BlueBubbles

ステータス: BlueBubbles macOS サーバーと HTTP 経由で通信するバンドルプラグイン。レガシーな imsg チャネルと比較して、より豊富な API と簡単なセットアップのため、**iMessage 統合に推奨**されます。

## 概要

-   BlueBubbles ヘルパーアプリ ([bluebubbles.app](https://bluebubbles.app)) を介して macOS 上で動作します。
-   推奨/テスト済み: macOS Sequoia (15)。macOS Tahoe (26) でも動作しますが、Tahoe では編集機能が現在壊れており、グループアイコンの更新は成功を報告しても同期されない場合があります。
-   OpenClaw は REST API (`GET /api/v1/ping`, `POST /message/text`, `POST /chat/:id/*`) を介して通信します。
-   受信メッセージは Webhook 経由で到着します。送信返信、入力インジケーター、既読確認、タップバックは REST 呼び出しです。
-   添付ファイルとステッカーはインバウンドメディアとして取り込まれ（可能な場合はエージェントに表示されます）。
-   ペアリング/許可リストは他のチャネル (`/channels/pairing` など) と同じように動作し、`channels.bluebubbles.allowFrom` + ペアリングコードを使用します。
-   リアクションは Slack/Telegram と同様にシステムイベントとして表示されるため、エージェントは返信する前にそれらを「言及」できます。
-   高度な機能: 編集、送信取り消し、返信スレッド、メッセージエフェクト、グループ管理。

## クイックスタート

1.  Mac に BlueBubbles サーバーをインストールします ([bluebubbles.app/install](https://bluebubbles.app/install) の指示に従ってください)。
2.  BlueBubbles 設定で Web API を有効にし、パスワードを設定します。
3.  `openclaw onboard` を実行し、BlueBubbles を選択するか、手動で設定します:
    
    コピー
    
    ```json
    {
      channels: {
        bluebubbles: {
          enabled: true,
          serverUrl: "http://192.168.1.100:1234",
          password: "example-password",
          webhookPath: "/bluebubbles-webhook",
        },
      },
    }
    ```
    
4.  BlueBubbles の Webhook をゲートウェイに向けます (例: `https://your-gateway-host:3000/bluebubbles-webhook?password=`)。
5.  ゲートウェイを起動します。Webhook ハンドラーが登録され、ペアリングが開始されます。

セキュリティ上の注意:

-   常に Webhook パスワードを設定してください。
-   Webhook 認証は常に必要です。OpenClaw は、`channels.bluebubbles.password` と一致するパスワード/guid (`?password=` や `x-password` など) を含まない BlueBubbles Webhook リクエストを拒否します。これはループバック/プロキシトポロジに関係なく適用されます。
-   パスワード認証は、Webhook ボディ全体を読み取り/解析する前にチェックされます。

## Messages.app を生存させ続ける (VM / ヘッドレス設定)

一部の macOS VM / 常時オン設定では、Messages.app が「アイドル」状態になることがあります (アプリが開かれる/フォアグラウンドになるまで受信イベントが停止します)。簡単な回避策は、AppleScript + LaunchAgent を使用して **5分ごとに Messages をポークする**ことです。

### 1) AppleScript を保存する

これを以下の名前で保存します:

-   `~/Scripts/poke-messages.scpt`

例のスクリプト (非対話型; フォーカスを奪いません):

```
try
  tell application "Messages"
    if not running then
      launch
    end if

    -- スクリプティングインターフェースに触れてプロセスを応答性のある状態に保つ。
    set _chatCount to (count of chats)
  end tell
on error
  -- 一時的な失敗 (初回実行プロンプト、ロックされたセッションなど) は無視する。
end try
```

### 2) LaunchAgent をインストールする

これを以下の名前で保存します:

-   `~/Library/LaunchAgents/com.user.poke-messages.plist`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>com.user.poke-messages</string>

    <key>ProgramArguments</key>
    <array>
      <string>/bin/bash</string>
      <string>-lc</string>
      <string>/usr/bin/osascript &quot;$HOME/Scripts/poke-messages.scpt&quot;</string>
    </array>

    <key>RunAtLoad</key>
    <true/>

    <key>StartInterval</key>
    <integer>300</integer>

    <key>StandardOutPath</key>
    <string>/tmp/poke-messages.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/poke-messages.err</string>
  </dict>
</plist>
```

注意点:

-   これは **300秒ごと** および **ログイン時** に実行されます。
-   初回実行時、macOS の **自動化** プロンプト (`osascript` → Messages) がトリガーされる場合があります。LaunchAgent を実行するのと同じユーザーセッションで承認してください。

ロードします:

```bash
launchctl unload ~/Library/LaunchAgents/com.user.poke-messages.plist 2>/dev/null || true
launchctl load ~/Library/LaunchAgents/com.user.poke-messages.plist
```

## オンボーディング

BlueBubbles は対話型セットアップウィザードで利用可能です:

```bash
openclaw onboard
```

ウィザードは以下をプロンプトします:

-   **サーバー URL** (必須): BlueBubbles サーバーアドレス (例: `http://192.168.1.100:1234`)
-   **パスワード** (必須): BlueBubbles サーバー設定の API パスワード
-   **Webhook パス** (オプション): デフォルトは `/bluebubbles-webhook`
-   **DM ポリシー**: ペアリング、許可リスト、オープン、または無効化
-   **許可リスト**: 電話番号、メールアドレス、またはチャットターゲット

CLI 経由で BlueBubbles を追加することもできます:

```bash
openclaw channels add bluebubbles --http-url http://192.168.1.100:1234 --password <password>
```

## アクセス制御 (DM + グループ)

DM:

-   デフォルト: `channels.bluebubbles.dmPolicy = "pairing"`。
-   未知の送信者はペアリングコードを受け取り、承認されるまでメッセージは無視されます (コードは1時間で期限切れ)。
-   以下で承認:
    -   `openclaw pairing list bluebubbles`
    -   `openclaw pairing approve bluebubbles `
-   ペアリングはデフォルトのトークン交換です。詳細: [ペアリング](./pairing.md)

グループ:

-   `channels.bluebubbles.groupPolicy = open | allowlist | disabled` (デフォルト: `allowlist`)。
-   `channels.bluebubbles.groupAllowFrom` は、`allowlist` が設定されている場合にグループ内で誰がトリガーできるかを制御します。

### メンションゲーティング (グループ)

BlueBubbles はグループチャット用のメンションゲーティングをサポートし、iMessage/WhatsApp の動作と一致します:

-   `agents.list[].groupChat.mentionPatterns` (または `messages.groupChat.mentionPatterns`) を使用してメンションを検出します。
-   グループで `requireMention` が有効な場合、エージェントはメンションされたときのみ応答します。
-   承認された送信者からの制御コマンドは、メンションゲーティングをバイパスします。

グループごとの設定:

```json
{
  channels: {
    bluebubbles: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true }, // すべてのグループのデフォルト
        "iMessage;-;chat123": { requireMention: false }, // 特定のグループのオーバーライド
      },
    },
  },
}
```

### コマンドゲーティング

-   制御コマンド (例: `/config`, `/model`) には承認が必要です。
-   `allowFrom` と `groupAllowFrom` を使用してコマンド承認を決定します。
-   承認された送信者は、グループ内でメンションされなくても制御コマンドを実行できます。

## 入力中 + 既読確認

-   **入力中インジケーター**: 応答生成の前および最中に自動的に送信されます。
-   **既読確認**: `channels.bluebubbles.sendReadReceipts` で制御されます (デフォルト: `true`)。
-   **入力中インジケーター**: OpenClaw は入力開始イベントを送信します。BlueBubbles は送信時またはタイムアウト時に自動的に入力をクリアします (DELETE による手動停止は信頼性が低い)。

```json
{
  channels: {
    bluebubbles: {
      sendReadReceipts: false, // 既読確認を無効化
    },
  },
}
```

## 高度なアクション

BlueBubbles は設定で有効にすると高度なメッセージアクションをサポートします:

```json
{
  channels: {
    bluebubbles: {
      actions: {
        reactions: true, // タップバック (デフォルト: true)
        edit: true, // 送信済みメッセージの編集 (macOS 13+, macOS 26 Tahoe では壊れています)
        unsend: true, // メッセージの送信取り消し (macOS 13+)
        reply: true, // メッセージ GUID による返信スレッド
        sendWithEffect: true, // メッセージエフェクト (slam, loud など)
        renameGroup: true, // グループチャットの名前変更
        setGroupIcon: true, // グループチャットのアイコン/写真設定 (macOS 26 Tahoe では不安定)
        addParticipant: true, // グループへの参加者追加
        removeParticipant: true, // グループからの参加者削除
        leaveGroup: true, // グループチャットからの退出
        sendAttachment: true, // 添付ファイル/メディアの送信
      },
    },
  },
}
```

利用可能なアクション:

-   **react**: タップバックリアクションの追加/削除 (`messageId`, `emoji`, `remove`)
-   **edit**: 送信済みメッセージの編集 (`messageId`, `text`)
-   **unsend**: メッセージの送信取り消し (`messageId`)
-   **reply**: 特定のメッセージへの返信 (`messageId`, `text`, `to`)
-   **sendWithEffect**: iMessage エフェクト付きで送信 (`text`, `to`, `effectId`)
-   **renameGroup**: グループチャットの名前変更 (`chatGuid`, `displayName`)
-   **setGroupIcon**: グループチャットのアイコン/写真設定 (`chatGuid`, `media`) — macOS 26 Tahoe では不安定 (API は成功を返してもアイコンが同期しない場合があります)。
-   **addParticipant**: グループへの参加者追加 (`chatGuid`, `address`)
-   **removeParticipant**: グループからの参加者削除 (`chatGuid`, `address`)
-   **leaveGroup**: グループチャットからの退出 (`chatGuid`)
-   **sendAttachment**: メディア/ファイルの送信 (`to`, `buffer`, `filename`, `asVoice`)
    -   ボイスメモ: `asVoice: true` を設定し、**MP3** または **CAF** オーディオを使用して iMessage ボイスメッセージとして送信します。BlueBubbles はボイスメモ送信時に MP3 → CAF に変換します。

### メッセージID (短縮 vs 完全)

OpenClaw はトークンを節約するために*短縮*メッセージID (例: `1`, `2`) を表示する場合があります。

-   `MessageSid` / `ReplyToId` は短縮 ID の場合があります。
-   `MessageSidFull` / `ReplyToIdFull` にはプロバイダーの完全な ID が含まれます。
-   短縮 ID はメモリ内にあり、再起動時またはキャッシュ削除時に期限切れになる可能性があります。
-   アクションは短縮または完全な `messageId` を受け入れますが、短縮 ID が利用できなくなるとエラーになります。

永続的な自動化と保存には完全な ID を使用してください:

-   テンプレート: `{{MessageSidFull}}`, `{{ReplyToIdFull}}`
-   コンテキスト: インバウンドペイロード内の `MessageSidFull` / `ReplyToIdFull`

テンプレート変数については [設定](../gateway/configuration.md) を参照してください。

## ブロックストリーミング

応答を単一のメッセージとして送信するか、ブロック単位でストリーミングするかを制御します:

```json
{
  channels: {
    bluebubbles: {
      blockStreaming: true, // ブロックストリーミングを有効化 (デフォルトはオフ)
    },
  },
}
```

## メディア + 制限

-   インバウンド添付ファイルはダウンロードされ、メディアキャッシュに保存されます。
-   メディアキャップは `channels.bluebubbles.mediaMaxMb` でインバウンドおよびアウトバウンドメディアに対して設定されます (デフォルト: 8 MB)。
-   アウトバウンドテキストは `channels.bluebubbles.textChunkLimit` でチャンク化されます (デフォルト: 4000 文字)。

## 設定リファレンス

完全な設定: [設定](../gateway/configuration.md) プロバイダーオプション:

-   `channels.bluebubbles.enabled`: チャネルの有効/無効。
-   `channels.bluebubbles.serverUrl`: BlueBubbles REST API ベース URL。
-   `channels.bluebubbles.password`: API パスワード。
-   `channels.bluebubbles.webhookPath`: Webhook エンドポイントパス (デフォルト: `/bluebubbles-webhook`)。
-   `channels.bluebubbles.dmPolicy`: `pairing | allowlist | open | disabled` (デフォルト: `pairing`)。
-   `channels.bluebubbles.allowFrom`: DM 許可リスト (ハンドル、メール、E.164 番号、`chat_id:*`, `chat_guid:*`)。
-   `channels.bluebubbles.groupPolicy`: `open | allowlist | disabled` (デフォルト: `allowlist`)。
-   `channels.bluebubbles.groupAllowFrom`: グループ送信者許可リスト。
-   `channels.bluebubbles.groups`: グループごとの設定 (`requireMention` など)。
-   `channels.bluebubbles.sendReadReceipts`: 既読確認を送信 (デフォルト: `true`)。
-   `channels.bluebubbles.blockStreaming`: ブロックストリーミングを有効化 (デフォルト: `false`; ストリーミング返信に必要)。
-   `channels.bluebubbles.textChunkLimit`: アウトバウンドチャンクサイズ (文字数) (デフォルト: 4000)。
-   `channels.bluebubbles.chunkMode`: `length` (デフォルト) は `textChunkLimit` を超えた場合のみ分割します; `newline` は長さチャンク化の前に空白行 (段落境界) で分割します。
-   `channels.bluebubbles.mediaMaxMb`: インバウンド/アウトバウンドメディアキャップ (MB) (デフォルト: 8)。
-   `channels.bluebubbles.mediaLocalRoots`: アウトバウンドローカルメディアパスに許可される絶対ローカルディレクトリの明示的な許可リスト。ローカルパス送信は、これが設定されていない限りデフォルトで拒否されます。アカウントごとのオーバーライド: `channels.bluebubbles.accounts..mediaLocalRoots`。
-   `channels.bluebubbles.historyLimit`: コンテキスト用のグループメッセージ最大数 (0 で無効化)。
-   `channels.bluebubbles.dmHistoryLimit`: DM 履歴制限。
-   `channels.bluebubbles.actions`: 特定のアクションの有効/無効。
-   `channels.bluebubbles.accounts`: マルチアカウント設定。

関連するグローバルオプション:

-   `agents.list[].groupChat.mentionPatterns` (または `messages.groupChat.mentionPatterns`)。
-   `messages.responsePrefix`。

## アドレッシング / 配信ターゲット

安定したルーティングには `chat_guid` を推奨します:

-   `chat_guid:iMessage;-;+15555550123` (グループに推奨)
-   `chat_id:123`
-   `chat_identifier:...`
-   直接ハンドル: `+15555550123`, `user@example.com`
    -   直接ハンドルに既存の DM チャットがない場合、OpenClaw は `POST /api/v1/chat/new` 経由で作成します。これには BlueBubbles プライベート API が有効になっている必要があります。

## セキュリティ

-   Webhook リクエストは、`guid`/`password` クエリパラメータまたはヘッダーを `channels.bluebubbles.password` と比較して認証されます。`localhost` からのリクエストも受け入れられます。
-   API パスワードと Webhook エンドポイントは秘密にしてください (認証情報のように扱ってください)。
-   Localhost 信頼は、同じホストのリバースプロキシが意図せずパスワードをバイパスする可能性があることを意味します。ゲートウェイをプロキシする場合は、プロキシで認証を要求し、`gateway.trustedProxies` を設定してください。詳細は [ゲートウェイセキュリティ](../gateway/security.md#reverse-proxy-configuration) を参照してください。
-   LAN 外に公開する場合は、BlueBubbles サーバーで HTTPS + ファイアウォールルールを有効にしてください。

## トラブルシューティング

-   入力中/既読イベントが動作しなくなった場合は、BlueBubbles Webhook ログを確認し、ゲートウェイパスが `channels.bluebubbles.webhookPath` と一致していることを確認してください。
-   ペアリングコードは1時間で期限切れになります。`openclaw pairing list bluebubbles` と `openclaw pairing approve bluebubbles ` を使用してください。
-   リアクションには BlueBubbles プライベート API (`POST /api/v1/message/react`) が必要です。サーバーバージョンがそれを公開していることを確認してください。
-   編集/送信取り消しには macOS 13+ および互換性のある BlueBubbles サーバーバージョンが必要です。macOS 26 (Tahoe) では、プライベート API の変更により編集機能が現在壊れています。
-   グループアイコンの更新は macOS 26 (Tahoe) で不安定な場合があります: API は成功を返しても新しいアイコンが同期しないことがあります。
-   OpenClaw は BlueBubbles サーバーの macOS バージョンに基づいて既知の壊れたアクションを自動的に非表示にします。macOS 26 (Tahoe) で編集がまだ表示される場合は、`channels.bluebubbles.actions.edit=false` で手動で無効にしてください。
-   ステータス/ヘルス情報については: `openclaw status --all` または `openclaw status --deep`。

一般的なチャネルワークフローのリファレンスについては、[チャネル](../channels.md) と [プラグイン](../tools/plugin.md) ガイドを参照してください。

[チャットチャネル](../channels.md)[Discord](./discord.md)