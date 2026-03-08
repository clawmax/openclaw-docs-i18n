

  メッセージングプラットフォーム

  
# WhatsApp

ステータス: WhatsApp Web (Baileys) 経由で本番環境対応。ゲートウェイがリンクされたセッションを所有します。

## クイックセットアップ

### ステップ 1: WhatsAppアクセスポリシーを構成する

```json
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing",
      allowFrom: ["+15551234567"],
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
  },
}
```

### ステップ 2: WhatsAppをリンクする (QR)

```bash
openclaw channels login --channel whatsapp
```

特定のアカウントの場合:

```bash
openclaw channels login --channel whatsapp --account work
```

### ステップ 3: ゲートウェイを起動する

```bash
openclaw gateway
```

### ステップ 4: 最初のペアリングリクエストを承認する (ペアリングモード使用時)

```bash
openclaw pairing list whatsapp
openclaw pairing approve whatsapp <CODE>
```

ペアリングリクエストは1時間後に期限切れになります。保留中のリクエストはチャネルごとに最大3件に制限されます。

 

> **ℹ️** OpenClawでは可能な限り、WhatsAppを別の番号で実行することを推奨しています。(チャネルのメタデータとオンボーディングフローはそのセットアップに最適化されていますが、個人番号のセットアップもサポートされています。)

## デプロイメントパターン

これは最もクリーンな運用モードです:

-   OpenClaw用の別個のWhatsAppアイデンティティ
-   より明確なDM許可リストとルーティング境界
-   自己チャットの混乱が起こりにくい

最小限のポリシーパターン:

```json
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"],
    },
  },
}
```

オンボーディングは個人番号モードをサポートし、自己チャットに適したベースラインを書き込みます:

-   `dmPolicy: "allowlist"`
-   `allowFrom` に個人番号を含む
-   `selfChatMode: true`

ランタイムでは、自己チャット保護はリンクされた自己番号と `allowFrom` を基準に動作します。

メッセージングプラットフォームチャネルは、現在のOpenClawチャネルアーキテクチャではWhatsApp Webベース (`Baileys`) です。組み込みのチャットチャネルレジストリには、別個のTwilio WhatsAppメッセージングチャネルは存在しません。

## ランタイムモデル

-   ゲートウェイがWhatsAppソケットと再接続ループを所有します。
-   送信には、対象アカウントのアクティブなWhatsAppリスナーが必要です。
-   ステータスおよびブロードキャストチャットは無視されます (`@status`, `@broadcast`)。
-   ダイレクトチャットはDMセッションルールを使用します (`session.dmScope`; デフォルト `main` はDMをエージェントのメインセッションに統合します)。
-   グループセッションは分離されます (`agent::whatsapp:group:`)。

## アクセス制御とアクティベーション

`channels.whatsapp.dmPolicy` はダイレクトチャットへのアクセスを制御します:

-   `pairing` (デフォルト)
-   `allowlist`
-   `open` (`allowFrom` に `"*"` を含む必要があります)
-   `disabled`

`allowFrom` はE.164形式の番号を受け入れます (内部的に正規化されます)。マルチアカウントのオーバーライド: `channels.whatsapp.accounts..dmPolicy` (および `allowFrom`) は、そのアカウントに対してチャネルレベルのデフォルト設定よりも優先されます。ランタイム動作の詳細:

-   ペアリングはチャネルの許可ストアに永続化され、設定された `allowFrom` とマージされます
-   許可リストが設定されていない場合、リンクされた自己番号がデフォルトで許可されます
-   送信側 `fromMe` のDMは自動的にペアリングされることはありません

グループアクセスには2つの層があります:

1.  **グループメンバーシップ許可リスト** (`channels.whatsapp.groups`)
    -   `groups` が省略されている場合、すべてのグループが対象となります
    -   `groups` が存在する場合、それはグループ許可リストとして機能します (`"*"` が許可されます)
2.  **グループ送信者ポリシー** (`channels.whatsapp.groupPolicy` + `groupAllowFrom`)
    -   `open`: 送信者許可リストがバイパスされます
    -   `allowlist`: 送信者は `groupAllowFrom` (または `*`) と一致する必要があります
    -   `disabled`: すべてのグループ受信をブロックします

送信者許可リストのフォールバック:

-   `groupAllowFrom` が設定されていない場合、ランタイムは利用可能な場合に `allowFrom` にフォールバックします
-   送信者許可リストは、メンション/返信アクティベーションの前に評価されます

注: `channels.whatsapp` ブロックがまったく存在しない場合、`channels.defaults.groupPolicy` が設定されていても、ランタイムのグループポリシーフォールバックは `allowlist` になります (警告ログ付き)。

グループ返信はデフォルトでメンションを必要とします。メンション検出には以下が含まれます:

-   ボットアイデンティティの明示的なWhatsAppメンション
-   設定されたメンション正規表現パターン (`agents.list[].groupChat.mentionPatterns`, フォールバック `messages.groupChat.mentionPatterns`)
-   暗黙的なボットへの返信検出 (返信送信者がボットアイデンティティと一致)

セキュリティ上の注意:

-   引用/返信はメンションゲートを満たすのみです。送信者の認可を**付与しません**
-   `groupPolicy: "allowlist"` の場合、許可リストにない送信者は、許可リストユーザーのメッセージに返信したとしてもブロックされます

セッションレベルのアクティベーションコマンド:

-   `/activation mention`
-   `/activation always`

`activation` はセッション状態を更新します (グローバル設定ではありません)。オーナーによるゲートがかかっています。

## 個人番号と自己チャットの動作

リンクされた自己番号が `allowFrom` にも存在する場合、WhatsApp自己チャット保護が有効になります:

-   自己チャットターンの既読レシートをスキップします
-   そうでなければ自分自身にpingを送信する可能性のあるメンションJID自動トリガー動作を無視します
-   `messages.responsePrefix` が設定されていない場合、自己チャット返信はデフォルトで `[{identity.name}]` または `[openclaw]` になります

## メッセージ正規化とコンテキスト

受信WhatsAppメッセージは共有の受信エンベロープでラップされます。引用返信が存在する場合、コンテキストは以下の形式で追加されます:

```json
[<sender> への返信 id:<stanzaId>]
<引用本文またはメディアプレースホルダー>
[/返信]
```

返信メタデータフィールドも、利用可能な場合に設定されます (`ReplyToId`, `ReplyToBody`, `ReplyToSender`, 送信者JID/E.164)。

メディアのみの受信メッセージは、以下のようなプレースホルダーで正規化されます:

-   `<media:image>`
-   `<media:video>`
-   `<media:audio>`
-   `<media:document>`
-   `<media:sticker>`

位置情報と連絡先のペイロードは、ルーティング前にテキストコンテキストに正規化されます。

グループの場合、未処理のメッセージはバッファリングされ、ボットが最終的にトリガーされたときにコンテキストとして注入されることがあります。

-   デフォルト制限: `50`
-   設定: `channels.whatsapp.historyLimit`
-   フォールバック: `messages.groupChat.historyLimit`
-   `0` で無効化

インジェクションマーカー:

-   `[前回の返信以降のチャットメッセージ - コンテキスト用]`
-   `[現在のメッセージ - これに返信してください]`

既読レシートは、受信したWhatsAppメッセージに対してデフォルトで有効になっています。グローバルに無効化:

```json
{
  channels: {
    whatsapp: {
      sendReadReceipts: false,
    },
  },
}
```

アカウントごとのオーバーライド:

```json
{
  channels: {
    whatsapp: {
      accounts: {
        work: {
          sendReadReceipts: false,
        },
      },
    },
  },
}
```

自己チャットターンは、グローバルに有効になっている場合でも既読レシートをスキップします。

## 配信、チャンキング、メディア

-   デフォルトチャンク制限: `channels.whatsapp.textChunkLimit = 4000`
-   `channels.whatsapp.chunkMode = "length" | "newline"`
-   `newline` モードは段落境界 (空行) を優先し、その後長さセーフなチャンキングにフォールバックします

-   画像、動画、音声 (PTTボイスノート)、ドキュメントペイロードをサポートします
-   `audio/ogg` はボイスノート互換性のために `audio/ogg; codecs=opus` に書き換えられます
-   アニメーションGIF再生は、動画送信時に `gifPlayback: true` を設定することでサポートされます
-   キャプションは、マルチメディア返信ペイロードを送信する際に最初のメディアアイテムに適用されます
-   メディアソースは HTTP(S)、`file://`、またはローカルパスです

-   受信メディア保存上限: `channels.whatsapp.mediaMaxMb` (デフォルト `50`)
-   送信メディア送信上限: `channels.whatsapp.mediaMaxMb` (デフォルト `50`)
-   アカウントごとのオーバーライドは `channels.whatsapp.accounts..mediaMaxMb` を使用します
-   画像は制限に収まるように自動最適化 (リサイズ/品質調整) されます
-   メディア送信失敗時、最初のアイテムのフォールバックは、応答を黙ってドロップする代わりにテキスト警告を送信します

## 受信確認リアクション

WhatsAppは、`channels.whatsapp.ackReaction` を介して受信確認時に即座にリアクションをサポートします。

```json
{
  channels: {
    whatsapp: {
      ackReaction: {
        emoji: "👀",
        direct: true,
        group: "mentions", // always | mentions | never
      },
    },
  },
}
```

動作に関する注意:

-   受信が承認された直後 (返信前) に送信されます
-   失敗はログに記録されますが、通常の返信配信をブロックしません
-   グループモード `mentions` はメンションでトリガーされたターンにリアクションします。グループアクティベーション `always` はこのチェックのバイパスとして機能します
-   WhatsAppは `channels.whatsapp.ackReaction` を使用します (従来の `messages.ackReaction` はここでは使用されません)

## マルチアカウントと認証情報

-   アカウントIDは `channels.whatsapp.accounts` から取得されます
-   デフォルトアカウント選択: `default` が存在する場合はそれ、それ以外は最初に設定されたアカウントID (ソート済み)
-   アカウントIDは内部的にルックアップ用に正規化されます

-   現在の認証パス: `~/.openclaw/credentials/whatsapp//creds.json`
-   バックアップファイル: `creds.json.bak`
-   レガシーデフォルト認証 (`~/.openclaw/credentials/` 内) は、デフォルトアカウントフローに対して認識/移行されます

`openclaw channels logout --channel whatsapp [--account ]` は、そのアカウントのWhatsApp認証状態をクリアします。レガシー認証ディレクトリでは、`oauth.json` は保持され、Baileys認証ファイルは削除されます。

## ツール、アクション、設定書き込み

-   エージェントツールサポートにはWhatsAppリアクションアクション (`react`) が含まれます。
-   アクションゲート:
    -   `channels.whatsapp.actions.reactions`
    -   `channels.whatsapp.actions.polls`
-   チャネル開始の設定書き込みはデフォルトで有効です (`channels.whatsapp.configWrites=false` で無効化可能)。

## トラブルシューティング

症状: チャネルステータスがリンクされていないと報告します。修正:

```bash
openclaw channels login --channel whatsapp
openclaw channels status
```

症状: リンクされたアカウントが繰り返し切断または再接続試行を行います。修正:

```bash
openclaw doctor
openclaw logs --follow
```

必要に応じて、`channels login` で再リンクします。

送信は、対象アカウントのアクティブなゲートウェイリスナーが存在しない場合、迅速に失敗します。ゲートウェイが実行中で、アカウントがリンクされていることを確認してください。

以下の順序で確認してください:

-   `groupPolicy`
-   `groupAllowFrom` / `allowFrom`
-   `groups` 許可リストエントリ
-   メンションゲート (`requireMention` + メンションパターン)
-   `openclaw.json` (JSON5) 内の重複キー: 後のエントリが前のエントリを上書きするため、スコープごとに単一の `groupPolicy` を保持してください

WhatsAppゲートウェイランタイムはNodeを使用する必要があります。Bunは安定したWhatsApp/Telegramゲートウェイ操作に対して互換性がないとフラグが立てられています。

## 設定リファレンスポインタ

主なリファレンス:

-   [設定リファレンス - WhatsApp](../gateway/configuration-reference.md#whatsapp)

重要なWhatsAppフィールド:

-   アクセス: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`
-   配信: `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `sendReadReceipts`, `ackReaction`
-   マルチアカウント: `accounts..enabled`, `accounts..authDir`, アカウントレベルのオーバーライド
-   運用: `configWrites`, `debounceMs`, `web.enabled`, `web.heartbeatSeconds`, `web.reconnect.*`
-   セッション動作: `session.dmScope`, `historyLimit`, `dmHistoryLimit`, `dms..historyLimit`

## 関連項目

-   [ペアリング](./pairing.md)
-   [チャネルルーティング](./channel-routing.md)
-   [マルチエージェントルーティング](../concepts/multi-agent.md)
-   [トラブルシューティング](./troubleshooting.md)

[Twitch](./twitch.md)[Zalo](./zalo.md)