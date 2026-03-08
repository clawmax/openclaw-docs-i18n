

  設定

  
# ペアリング

「ペアリング」は、OpenClawの明示的な**所有者承認**ステップです。以下の2つの場面で使用されます：

1.  **DMペアリング** (ボットと通信を許可するユーザー)
2.  **ノードペアリング** (ゲートウェイネットワークへの参加を許可するデバイス/ノード)

セキュリティの背景: [セキュリティ](../gateway/security.md)

## 1) DMペアリング (受信チャットアクセス)

チャネルがDMポリシー `pairing` で設定されている場合、未知の送信者には短いコードが送られ、そのメッセージは**承認されるまで処理されません**。デフォルトのDMポリシーは以下に記載されています: [セキュリティ](../gateway/security.md) ペアリングコード:

-   8文字、大文字、曖昧な文字 (`0O1I`) は含まれません。
-   **1時間で期限切れ**。ボットは新しいリクエストが作成されたときのみペアリングメッセージを送信します (送信者ごとにおおよそ1時間に1回)。
-   保留中のDMペアリングリクエストは、デフォルトで**チャネルごとに3件**に制限されています。それ以上のリクエストは、1件が期限切れになるか承認されるまで無視されます。

### 送信者を承認する

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

対応チャネル: `telegram`, `whatsapp`, `signal`, `imessage`, `discord`, `slack`, `feishu`.

### 状態の保存場所

`~/.openclaw/credentials/` 以下に保存されます:

-   保留中のリクエスト: `-pairing.json`
-   承認済み許可リストストア:
    -   デフォルトアカウント: `-allowFrom.json`
    -   非デフォルトアカウント: `--allowFrom.json`

アカウントスコープの動作:

-   非デフォルトアカウントは、自身のスコープが設定された許可リストファイルのみを読み書きします。
-   デフォルトアカウントは、チャネルスコープの、スコープが設定されていない許可リストファイルを使用します。

これらは機密として扱ってください (これらはあなたのアシスタントへのアクセスを制御します)。

## 2) ノードデバイスペアリング (iOS/Android/macOS/ヘッドレスノード)

ノードは `role: node` を持つ**デバイス**としてゲートウェイに接続します。ゲートウェイは承認が必要なデバイスペアリングリクエストを作成します。

### Telegram経由でペアリング (iOS推奨)

`device-pair` プラグインを使用している場合、初回のデバイスペアリングをTelegramから完全に行うことができます:

1.  Telegramで、ボットにメッセージを送信: `/pair`
2.  ボットは2つのメッセージで返信します: 説明メッセージと、別の**セットアップコード**メッセージ (Telegram内でコピー/ペーストしやすい) です。
3.  スマートフォンで、OpenClaw iOSアプリを開く → 設定 → ゲートウェイ。
4.  セットアップコードを貼り付けて接続します。
5.  Telegramに戻る: `/pair approve`

セットアップコードは、以下を含むbase64エンコードされたJSONペイロードです:

-   `url`: ゲートウェイWebSocket URL (`ws://...` または `wss://...`)
-   `token`: 短命のペアリングトークン

セットアップコードは有効な間はパスワードのように扱ってください。

### ノードデバイスを承認する

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
```

### ノードペアリング状態の保存

`~/.openclaw/devices/` 以下に保存されます:

-   `pending.json` (短命; 保留中のリクエストは期限切れになります)
-   `paired.json` (ペアリング済みデバイス + トークン)

### 注意点

-   従来の `node.pair.*` API (CLI: `openclaw nodes pending/approve`) は、別のゲートウェイ所有のペアリングストアです。WSノードは依然としてデバイスペアリングが必要です。

## 関連ドキュメント

-   セキュリティモデル + プロンプトインジェクション: [セキュリティ](../gateway/security.md)
-   安全な更新 (doctorの実行): [更新](../install/updating.md)
-   チャネル設定:
    -   Telegram: [Telegram](./telegram.md)
    -   WhatsApp: [WhatsApp](./whatsapp.md)
    -   Signal: [Signal](./signal.md)
    -   BlueBubbles (iMessage): [BlueBubbles](./bluebubbles.md)
    -   iMessage (レガシー): [iMessage](./imessage.md)
    -   Discord: [Discord](./discord.md)
    -   Slack: [Slack](./slack.md)

[Zalo Personal](./zalouser.md)[グループメッセージ](./group-messages.md)

---