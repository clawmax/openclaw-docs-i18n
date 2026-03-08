

  メッセージングプラットフォーム

  
# iMessage

> **⚠️** 新しいiMessageデプロイメントには、[BlueBubbles](./bluebubbles.md)を使用してください。`imsg`統合はレガシーであり、将来のリリースで削除される可能性があります。

 ステータス: レガシー外部CLI統合。ゲートウェイは`imsg rpc`を生成し、stdio上のJSON-RPCを介して通信します（別個のデーモン/ポートは不要）。

## クイックセットアップ

## 要件と権限 (macOS)

-   `imsg`を実行するMac上で、メッセージアプリにサインインしている必要があります。
-   OpenClaw/`imsg`を実行するプロセスコンテキストに対して、フルディスクアクセスが必要です（メッセージDBへのアクセス）。
-   Messages.app経由でメッセージを送信するには、自動化の権限が必要です。

> **💡** 権限はプロセスコンテキストごとに付与されます。ゲートウェイがヘッドレス（LaunchAgent/SSH）で実行される場合、同じコンテキストで一度だけ対話型コマンドを実行してプロンプトをトリガーします：
> 
> コピー
> 
> ```
> imsg chats --limit 1
> # または
> imsg send <handle> "test"
> ```

## アクセス制御とルーティング

`channels.imessage.dmPolicy`はダイレクトメッセージを制御します：

-   `pairing` (デフォルト)
-   `allowlist`
-   `open` (`allowFrom`に`"*"`を含む必要があります)
-   `disabled`

許可リストフィールド：`channels.imessage.allowFrom`。許可リストエントリは、ハンドルまたはチャットターゲット（`chat_id:*`, `chat_guid:*`, `chat_identifier:*`）にすることができます。

`channels.imessage.groupPolicy`はグループの扱いを制御します：

-   `allowlist` (構成時にデフォルト)
-   `open`
-   `disabled`

グループ送信者許可リスト：`channels.imessage.groupAllowFrom`。実行時フォールバック：`groupAllowFrom`が設定されていない場合、iMessageグループ送信者チェックは、利用可能な場合`allowFrom`にフォールバックします。実行時注意：`channels.imessage`が完全に欠落している場合、実行時は`groupPolicy="allowlist"`にフォールバックし、警告をログに記録します（`channels.defaults.groupPolicy`が設定されていても）。グループのメンションゲーティング：

-   iMessageにはネイティブのメンションメタデータがありません
-   メンション検出は正規表現パターンを使用します（`agents.list[].groupChat.mentionPatterns`、フォールバック`messages.groupChat.mentionPatterns`）
-   構成されたパターンがない場合、メンションゲーティングは適用できません

許可された送信者からの制御コマンドは、グループ内のメンションゲーティングをバイパスできます。

-   DMはダイレクトルーティングを使用します；グループはグループルーティングを使用します。
-   デフォルトの`session.dmScope=main`では、iMessage DMはエージェントのメインセッションに統合されます。
-   グループセッションは分離されます（`agent::imessage:group:<chat_id>`）。
-   返信は、発信元チャネル/ターゲットメタデータを使用してiMessageにルーティングされます。

グループ風スレッド動作：一部の複数参加者のiMessageスレッドは、`is_group=false`で到着することがあります。その`chat_id`が`channels.imessage.groups`の下で明示的に構成されている場合、OpenClawはそれをグループトラフィックとして扱います（グループゲーティング + グループセッション分離）。

## デプロイメントパターン

専用のApple IDとmacOSユーザーを使用して、ボットのトラフィックを個人のメッセージプロファイルから分離します。典型的なフロー：

1.  専用のmacOSユーザーを作成/サインインします。
2.  そのユーザーでボット用Apple IDを使用してメッセージアプリにサインインします。
3.  そのユーザーに`imsg`をインストールします。
4.  OpenClawがそのユーザーコンテキストで`imsg`を実行できるようにSSHラッパーを作成します。
5.  `channels.imessage.accounts..cliPath`と`.dbPath`をそのユーザープロファイルにポイントします。

初回実行では、そのボットユーザーセッションでGUI承認（自動化 + フルディスクアクセス）が必要になる場合があります。

一般的なトポロジ：

-   ゲートウェイはLinux/VM上で実行
-   iMessage + `imsg`はあなたのtailnet内のMac上で実行
-   `cliPath`ラッパーはSSHを使用して`imsg`を実行
-   `remoteHost`はSCP添付ファイル取得を有効化

例：

```json
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db",
    },
  },
}
```

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

SSHとSCPが非対話型になるようにSSHキーを使用します。まずホストキーが信頼されていることを確認します（例：`ssh bot@mac-mini.tailnet-1234.ts.net`）`known_hosts`が設定されるように。

iMessageは`channels.imessage.accounts`の下でアカウントごとの設定をサポートします。各アカウントは、`cliPath`、`dbPath`、`allowFrom`、`groupPolicy`、`mediaMaxMb`、履歴設定、添付ファイルルート許可リストなどのフィールドをオーバーライドできます。

## メディア、チャンキング、配信ターゲット

-   インバウンド添付ファイルの取り込みはオプション：`channels.imessage.includeAttachments`
-   リモート添付ファイルパスは、`remoteHost`が設定されている場合、SCP経由で取得できます
-   添付ファイルパスは許可されたルートと一致する必要があります：
    -   `channels.imessage.attachmentRoots` (ローカル)
    -   `channels.imessage.remoteAttachmentRoots` (リモートSCPモード)
    -   デフォルトルートパターン：`/Users/*/Library/Messages/Attachments`
-   SCPは厳格なホストキーチェックを使用します（`StrictHostKeyChecking=yes`）
-   アウトバウンドメディアサイズは`channels.imessage.mediaMaxMb`を使用します（デフォルト16 MB）

-   テキストチャンク制限：`channels.imessage.textChunkLimit` (デフォルト4000)
-   チャンクモード：`channels.imessage.chunkMode`
    -   `length` (デフォルト)
    -   `newline` (段落優先分割)

推奨される明示的ターゲット：

-   `chat_id:123` (安定したルーティングに推奨)
-   `chat_guid:...`
-   `chat_identifier:...`

ハンドルターゲットもサポートされています：

-   `imessage:+1555...`
-   `sms:+1555...`
-   `user@example.com`

```bash
imsg chats --limit 20
```

## 設定書き込み

iMessageはデフォルトでチャネル開始の設定書き込みを許可します（`commands.config: true`の場合の`/config set|unset`用）。無効化：

```json
{
  channels: {
    imessage: {
      configWrites: false,
    },
  },
}
```

## トラブルシューティング

バイナリとRPCサポートを検証します：

```bash
imsg rpc --help
openclaw channels status --probe
```

プローブがRPC未サポートを報告する場合、`imsg`を更新してください。

以下を確認：

-   `channels.imessage.dmPolicy`
-   `channels.imessage.allowFrom`
-   ペアリング承認（`openclaw pairing list imessage`）

以下を確認：

-   `channels.imessage.groupPolicy`
-   `channels.imessage.groupAllowFrom`
-   `channels.imessage.groups`許可リスト動作
-   メンションパターン構成（`agents.list[].groupChat.mentionPatterns`）

以下を確認：

-   `channels.imessage.remoteHost`
-   `channels.imessage.remoteAttachmentRoots`
-   ゲートウェイホストからのSSH/SCPキー認証
-   ゲートウェイホストの`~/.ssh/known_hosts`にホストキーが存在する
-   メッセージを実行するMac上のリモートパスの読み取り可能性

同じユーザー/セッションコンテキストで対話型GUIターミナルで再実行し、プロンプトを承認します：

```bash
imsg chats --limit 1
imsg send <handle> "test"
```

OpenClaw/`imsg`を実行するプロセスコンテキストに対して、フルディスクアクセス + 自動化が付与されていることを確認します。

## 設定リファレンスポインタ

-   [設定リファレンス - iMessage](../gateway/configuration-reference.md#imessage)
-   [ゲートウェイ設定](../gateway/configuration.md)
-   [ペアリング](./pairing.md)
-   [BlueBubbles](./bluebubbles.md)

[Google Chat](./googlechat.md)[IRC](./irc.md)