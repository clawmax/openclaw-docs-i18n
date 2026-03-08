

  メッセージングプラットフォーム

  
# Signal

ステータス: 外部CLI統合。ゲートウェイはHTTP JSON-RPC + SSEを介して`signal-cli`と通信します。

## 前提条件

-   サーバーにOpenClawがインストールされていること（以下のLinuxフローはUbuntu 24でテスト済み）。
-   ゲートウェイが動作するホストで`signal-cli`が利用可能であること。
-   1回の検証SMSを受信できる電話番号（SMS登録パスの場合）。
-   登録中のSignalキャプチャ（`signalcaptchas.org`）へのブラウザアクセス。

## クイックセットアップ（初心者向け）

1.  ボットには**別のSignal番号**を使用することを推奨します。
2.  `signal-cli`をインストールします（JVMビルドを使用する場合はJavaが必要）。
3.  セットアップパスを1つ選択します：
    -   **パスA (QRリンク):** `signal-cli link -n "OpenClaw"`を実行し、SignalでQRコードをスキャンします。
    -   **パスB (SMS登録):** キャプチャ + SMS検証で専用番号を登録します。
4.  OpenClawを設定し、ゲートウェイを再起動します。
5.  最初のDMを送信し、ペアリングを承認します（`openclaw pairing approve signal <コード>`）。

最小構成：

```json
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

フィールドリファレンス：

| フィールド | 説明 |
| --- | --- |
| `account` | E.164形式（`+15551234567`）のボット電話番号 |
| `cliPath` | `signal-cli`へのパス（`PATH`にある場合は`signal-cli`） |
| `dmPolicy` | DMアクセスポリシー（`pairing`を推奨） |
| `allowFrom` | DMを許可する電話番号または`uuid:`値 |

## 概要

-   `signal-cli`を介したSignalチャネル（組み込みlibsignalではありません）。
-   決定論的ルーティング：返信は常にSignalに戻ります。
-   DMはエージェントのメインセッションを共有します。グループは分離されています（`agent::signal:group:`）。

## 設定書き込み

デフォルトでは、Signalは`/config set|unset`によってトリガーされる設定更新の書き込みを許可されています（`commands.config: true`が必要）。無効にするには：

```json
{
  channels: { signal: { configWrites: false } },
}
```

## 番号モデル（重要）

-   ゲートウェイは**Signalデバイス**（`signal-cli`アカウント）に接続します。
-   **個人のSignalアカウント**でボットを実行すると、自身のメッセージは無視されます（ループ保護）。
-   「ボットにメッセージを送ると返信が来る」には、**別のボット番号**を使用してください。

## セットアップパスA: 既存のSignalアカウントをリンク（QR）

1.  `signal-cli`をインストールします（JVMまたはネイティブビルド）。
2.  ボットアカウントをリンクします：
    -   `signal-cli link -n "OpenClaw"`を実行し、SignalでQRコードをスキャンします。
3.  Signalを設定し、ゲートウェイを起動します。

例：

```json
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

マルチアカウントサポート：`channels.signal.accounts`を使用し、アカウントごとの設定とオプションの`name`を指定します。共有パターンについては[`gateway/configuration`](../gateway/configuration.md#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts)を参照してください。

## セットアップパスB: 専用ボット番号を登録（SMS、Linux）

既存のSignalアプリアカウントをリンクする代わりに、専用のボット番号が必要な場合に使用します。

1.  SMS（または固定電話の場合は音声検証）を受信できる番号を取得します。
    -   アカウント/セッションの競合を避けるために、専用のボット番号を使用してください。
2.  ゲートウェイホストに`signal-cli`をインストールします：

```
VERSION=$(curl -Ls -o /dev/null -w %{url_effective} https://github.com/AsamK/signal-cli/releases/latest | sed -e 's/^.*\/v//')
curl -L -O "https://github.com/AsamK/signal-cli/releases/download/v${VERSION}/signal-cli-${VERSION}-Linux-native.tar.gz"
sudo tar xf "signal-cli-${VERSION}-Linux-native.tar.gz" -C /opt
sudo ln -sf /opt/signal-cli /usr/local/bin/
signal-cli --version
```

JVMビルド（`signal-cli-${VERSION}.tar.gz`）を使用する場合は、まずJRE 25+をインストールしてください。`signal-cli`を最新の状態に保ってください。アップストリームでは、古いリリースはSignalサーバーAPIの変更により動作しなくなる可能性があると記載されています。

3.  番号を登録および検証します：

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register
```

キャプチャが必要な場合：

1.  `https://signalcaptchas.org/registration/generate.html`を開きます。
2.  キャプチャを完了し、「Open Signal」から`signalcaptcha://...`リンクターゲットをコピーします。
3.  可能であれば、ブラウザセッションと同じ外部IPから実行します。
4.  すぐに登録を再実行します（キャプチャトークンはすぐに期限切れになります）：

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register --captcha '<SIGNALCAPTCHA_URL>'
signal-cli -a +<BOT_PHONE_NUMBER> verify <VERIFICATION_CODE>
```

4.  OpenClawを設定し、ゲートウェイを再起動し、チャネルを確認します：

```bash
# ユーザーsystemdサービスとしてゲートウェイを実行している場合：
systemctl --user restart openclaw-gateway

# その後確認：
openclaw doctor
openclaw channels status --probe
```

5.  DM送信者をペアリングします：
    -   ボット番号に任意のメッセージを送信します。
    -   サーバーでコードを承認します：`openclaw pairing approve signal <ペアリングコード>`。
    -   「不明な連絡先」を避けるために、ボット番号を電話の連絡先として保存します。

重要：`signal-cli`で電話番号アカウントを登録すると、その番号のメインSignalアプリセッションの認証が解除される可能性があります。既存の電話アプリ設定を維持する必要がある場合は、専用のボット番号を使用するか、QRリンクモードを使用してください。アップストリームリファレンス：

-   `signal-cli` README: `https://github.com/AsamK/signal-cli`
-   キャプチャフロー: `https://github.com/AsamK/signal-cli/wiki/Registration-with-captcha`
-   リンクフロー: `https://github.com/AsamK/signal-cli/wiki/Linking-other-devices-(Provisioning)`

## 外部デーモンモード（httpUrl）

`signal-cli`を自分で管理したい場合（遅いJVMコールドスタート、コンテナ初期化、または共有CPU）、デーモンを別々に実行し、OpenClawをそれに向けます：

```json
{
  channels: {
    signal: {
      httpUrl: "http://127.0.0.1:8080",
      autoStart: false,
    },
  },
}
```

これにより、OpenClaw内部での自動起動と起動待機がスキップされます。自動起動時の起動が遅い場合は、`channels.signal.startupTimeoutMs`を設定します。

## アクセス制御（DM + グループ）

DM：

-   デフォルト：`channels.signal.dmPolicy = "pairing"`。
-   不明な送信者はペアリングコードを受け取り、承認されるまでメッセージは無視されます（コードは1時間後に期限切れ）。
-   承認方法：
    -   `openclaw pairing list signal`
    -   `openclaw pairing approve signal <コード>`
-   ペアリングはSignal DMのデフォルトのトークン交換です。詳細：[ペアリング](./pairing.md)
-   UUIDのみの送信者（`sourceUuid`から）は、`channels.signal.allowFrom`に`uuid:`として保存されます。

グループ：

-   `channels.signal.groupPolicy = open | allowlist | disabled`。
-   `channels.signal.groupAllowFrom`は、`allowlist`が設定されている場合にグループ内でトリガーできる送信者を制御します。
-   実行時注意：`channels.signal`が完全に欠落している場合、実行時はグループチェックのために`groupPolicy="allowlist"`にフォールバックします（`channels.defaults.groupPolicy`が設定されていても）。

## 動作の仕組み

-   `signal-cli`はデーモンとして実行され、ゲートウェイはSSEを介してイベントを読み取ります。
-   受信メッセージは共有チャネルエンベロープに正規化されます。
-   返信は常に同じ番号またはグループにルーティングされます。

## メディア + 制限

-   送信テキストは`channels.signal.textChunkLimit`（デフォルト4000）でチャンク化されます。
-   オプションの改行チャンク化：`channels.signal.chunkMode="newline"`を設定すると、長さによるチャンク化の前に空白行（段落境界）で分割します。
-   添付ファイルをサポート（`signal-cli`からbase64で取得）。
-   デフォルトのメディア上限：`channels.signal.mediaMaxMb`（デフォルト8）。
-   `channels.signal.ignoreAttachments`を使用してメディアのダウンロードをスキップします。
-   グループ履歴コンテキストは`channels.signal.historyLimit`（または`channels.signal.accounts.*.historyLimit`）を使用し、`messages.groupChat.historyLimit`にフォールバックします。無効にするには`0`を設定します（デフォルト50）。

## 入力中 + 既読確認

-   **入力インジケーター**: OpenClawは`signal-cli sendTyping`を介して入力中信号を送信し、返信中に更新します。
-   **既読確認**: `channels.signal.sendReadReceipts`がtrueの場合、OpenClawは許可されたDMの既読確認を転送します。
-   Signal-cliはグループの既読確認を公開しません。

## リアクション（メッセージツール）

-   `message action=react`を`channel=signal`で使用します。
-   ターゲット：送信者E.164またはUUID（ペアリング出力からの`uuid:`を使用。UUIDのみでも動作します）。
-   `messageId`はリアクション対象のメッセージのSignalタイムスタンプです。
-   グループリアクションには`targetAuthor`または`targetAuthorUuid`が必要です。

例：

```bash
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅
```

設定：

-   `channels.signal.actions.reactions`: リアクションアクションを有効/無効にします（デフォルトtrue）。
-   `channels.signal.reactionLevel`: `off | ack | minimal | extensive`。
    -   `off`/`ack`はエージェントリアクションを無効にします（メッセージツール`react`はエラーになります）。
    -   `minimal`/`extensive`はエージェントリアクションを有効にし、ガイダンスレベルを設定します。
-   アカウントごとのオーバーライド：`channels.signal.accounts..actions.reactions`、`channels.signal.accounts..reactionLevel`。

## 配信ターゲット（CLI/cron）

-   DM: `signal:+15551234567`（またはプレーンなE.164）。
-   UUID DM: `uuid:`（またはUUIDのみ）。
-   グループ: `signal:group:`。
-   ユーザー名: `username:`（Signalアカウントでサポートされている場合）。

## トラブルシューティング

まずこのラダーを実行します：

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

必要に応じてDMペアリング状態を確認します：

```bash
openclaw pairing list signal
```

一般的な障害：

-   デーモンに到達可能だが返信がない：アカウント/デーモン設定（`httpUrl`、`account`）と受信モードを確認します。
-   DMが無視される：送信者がペアリング承認待ちです。
-   グループメッセージが無視される：グループ送信者/メンションゲーティングが配信をブロックしています。
-   編集後の設定検証エラー：`openclaw doctor --fix`を実行します。
-   診断にSignalが表示されない：`channels.signal.enabled: true`を確認します。

追加チェック：

```bash
openclaw pairing list signal
pgrep -af signal-cli
grep -i "signal" "/tmp/openclaw/openclaw-$(date +%Y-%m-%d).log" | tail -20
```

トリアージフローについては：[/channels/troubleshooting](./troubleshooting.md)。

## セキュリティに関する注意

-   `signal-cli`はアカウントキーをローカルに保存します（通常`~/.local/share/signal-cli/data/`）。
-   サーバー移行または再構築の前にSignalアカウント状態をバックアップしてください。
-   より広範なDMアクセスを明示的に希望しない限り、`channels.signal.dmPolicy: "pairing"`を維持してください。
-   SMS検証は登録または回復フローでのみ必要ですが、番号/アカウントの制御を失うと再登録が複雑になる可能性があります。

## 設定リファレンス（Signal）

完全な設定：[設定](../gateway/configuration.md) プロバイダーオプション：

-   `channels.signal.enabled`: チャネル起動を有効/無効にします。
-   `channels.signal.account`: ボットアカウントのE.164。
-   `channels.signal.cliPath`: `signal-cli`へのパス。
-   `channels.signal.httpUrl`: 完全なデーモンURL（ホスト/ポートを上書き）。
-   `channels.signal.httpHost`、`channels.signal.httpPort`: デーモンバインド（デフォルト127.0.0.1:8080）。
-   `channels.signal.autoStart`: デーモンを自動起動します（`httpUrl`が未設定の場合はデフォルトtrue）。
-   `channels.signal.startupTimeoutMs`: 起動待機タイムアウト（ミリ秒、上限120000）。
-   `channels.signal.receiveMode`: `on-start | manual`。
-   `channels.signal.ignoreAttachments`: 添付ファイルのダウンロードをスキップします。
-   `channels.signal.ignoreStories`: デーモンからのストーリーを無視します。
-   `channels.signal.sendReadReceipts`: 既読確認を転送します。
-   `channels.signal.dmPolicy`: `pairing | allowlist | open | disabled`（デフォルト: pairing）。
-   `channels.signal.allowFrom`: DM許可リスト（E.164または`uuid:`）。`open`には`"*"`が必要です。Signalにはユーザー名がありません。電話/UUID IDを使用してください。
-   `channels.signal.groupPolicy`: `open | allowlist | disabled`（デフォルト: allowlist）。
-   `channels.signal.groupAllowFrom`: グループ送信者許可リスト。
-   `channels.signal.historyLimit`: コンテキストとして含める最大グループメッセージ数（0で無効）。
-   `channels.signal.dmHistoryLimit`: ユーザーターン単位のDM履歴制限。ユーザーごとのオーバーライド：`channels.signal.dms["<phone_or_uuid>"].historyLimit`。
-   `channels.signal.textChunkLimit`: 送信チャンクサイズ（文字）。
-   `channels.signal.chunkMode`: `length`（デフォルト）または`newline`（長さによるチャンク化の前に空白行（段落境界）で分割）。
-   `channels.signal.mediaMaxMb`: 受信/送信メディア上限（MB）。

関連するグローバルオプション：

-   `agents.list[].groupChat.mentionPatterns`（Signalはネイティブメンションをサポートしません）。
-   `messages.groupChat.mentionPatterns`（グローバルフォールバック）。
-   `messages.responsePrefix`。

[Nostr](./nostr.md)[Synology Chat](./synology-chat.md)