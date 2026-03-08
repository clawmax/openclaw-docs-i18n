

  メッセージングプラットフォーム

  
# Matrix

Matrix はオープンで分散型のメッセージングプロトコルです。OpenClaw は任意のホームサーバー上の Matrix **ユーザー**として接続するため、ボット用の Matrix アカウントが必要です。ログイン後、ボットに直接DMを送信したり、ルーム（Matrix の「グループ」）に招待したりできます。Beeper も有効なクライアントオプションですが、E2EE を有効にする必要があります。ステータス: プラグイン (@vector-im/matrix-bot-sdk) 経由でサポートされています。ダイレクトメッセージ、ルーム、スレッド、メディア、リアクション、投票（送信 + 投票開始をテキストとして）、位置情報、および E2EE（暗号化サポート付き）に対応。

## プラグイン必須

Matrix はプラグインとして提供され、コアインストールにはバンドルされていません。CLI（npm レジストリ）経由でインストールします:

```bash
openclaw plugins install @openclaw/matrix
```

ローカルチェックアウト（git リポジトリから実行する場合）:

```bash
openclaw plugins install ./extensions/matrix
```

configure/オンボーディング中に Matrix を選択し、git チェックアウトが検出された場合、OpenClaw は自動的にローカルインストールパスを提供します。詳細: [プラグイン](../tools/plugin.md)

## セットアップ

1.  Matrix プラグインをインストール:
    -   npm から: `openclaw plugins install @openclaw/matrix`
    -   ローカルチェックアウトから: `openclaw plugins install ./extensions/matrix`
2.  ホームサーバー上に Matrix アカウントを作成:
    -   [https://matrix.org/ecosystem/hosting/](https://matrix.org/ecosystem/hosting/) でホスティングオプションを参照
    -   または自分でホストします。
3.  ボットアカウントのアクセストークンを取得:
    
    -   ホームサーバーで `curl` を使用して Matrix ログイン API を呼び出します:
    
    コピー
    
    ```bash
    curl --request POST \
      --url https://matrix.example.org/_matrix/client/v3/login \
      --header 'Content-Type: application/json' \
      --data '{
      "type": "m.login.password",
      "identifier": {
        "type": "m.id.user",
        "user": "your-user-name"
      },
      "password": "your-password"
    }'
    ```
    
    -   `matrix.example.org` をあなたのホームサーバーの URL に置き換えてください。
    -   または、`channels.matrix.userId` + `channels.matrix.password` を設定: OpenClaw は同じログインエンドポイントを呼び出し、アクセストークンを `~/.openclaw/credentials/matrix/credentials.json` に保存し、次回起動時に再利用します。
4.  認証情報を設定:
    -   環境変数: `MATRIX_HOMESERVER`, `MATRIX_ACCESS_TOKEN` (または `MATRIX_USER_ID` + `MATRIX_PASSWORD`)
    -   または設定ファイル: `channels.matrix.*`
    -   両方が設定されている場合、設定ファイルが優先されます。
    -   アクセストークンを使用する場合: ユーザーIDは `/whoami` 経由で自動的に取得されます。
    -   設定する場合、`channels.matrix.userId` は完全な Matrix ID である必要があります（例: `@bot:example.org`）。
5.  ゲートウェイを再起動（またはオンボーディングを完了）。
6.  任意の Matrix クライアント（Element、Beeper など; [https://matrix.org/ecosystem/clients/](https://matrix.org/ecosystem/clients/) を参照）からボットとのDMを開始するか、ルームに招待します。Beeper は E2EE を必要とするため、`channels.matrix.encryption: true` を設定し、デバイスを検証してください。

最小限の設定（アクセストークン、ユーザーIDは自動取得）:

```json
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      dm: { policy: "pairing" },
    },
  },
}
```

E2EE 設定（エンドツーエンド暗号化有効）:

```json
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      encryption: true,
      dm: { policy: "pairing" },
    },
  },
}
```

## 暗号化 (E2EE)

エンドツーエンド暗号化は Rust 暗号化 SDK 経由で**サポートされています**。`channels.matrix.encryption: true` で有効にします:

-   暗号化モジュールが読み込まれると、暗号化されたルームは自動的に復号されます。
-   送信メディアは、暗号化されたルームに送信する際に暗号化されます。
-   初回接続時、OpenClaw は他のセッションからデバイス検証を要求します。
-   別の Matrix クライアント（Element など）でデバイスを検証して鍵共有を有効にします。
-   暗号化モジュールが読み込めない場合、E2EE は無効になり、暗号化されたルームは復号されません。OpenClaw は警告をログに記録します。
-   暗号化モジュールの欠落エラー（例: `@matrix-org/matrix-sdk-crypto-nodejs-*`）が表示される場合は、`@matrix-org/matrix-sdk-crypto-nodejs` のビルドスクリプトを許可し、`pnpm rebuild @matrix-org/matrix-sdk-crypto-nodejs` を実行するか、`node node_modules/@matrix-org/matrix-sdk-crypto-nodejs/download-lib.js` でバイナリを取得してください。

暗号化状態は、アカウント + アクセストークンごとに `~/.openclaw/matrix/accounts//__/<token-hash>/crypto/`（SQLite データベース）に保存されます。同期状態は、その隣の `bot-storage.json` に保存されます。アクセストークン（デバイス）が変更されると、新しいストアが作成され、暗号化されたルーム用にボットを再検証する必要があります。**デバイス検証:** E2EE が有効な場合、ボットは起動時に他のセッションから検証を要求します。Element（または別のクライアント）を開き、検証リクエストを承認して信頼関係を確立します。一度検証されると、ボットは暗号化されたルーム内のメッセージを復号できるようになります。

## マルチアカウント

マルチアカウントサポート: `channels.matrix.accounts` を使用し、アカウントごとの認証情報とオプションの `name` を設定します。共有パターンについては [`gateway/configuration`](../gateway/configuration.md#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) を参照してください。各アカウントは、任意のホームサーバー上の別々の Matrix ユーザーとして実行されます。アカウントごとの設定は、トップレベルの `channels.matrix` 設定を継承し、任意のオプション（DMポリシー、グループ、暗号化など）をオーバーライドできます。

```json
{
  channels: {
    matrix: {
      enabled: true,
      dm: { policy: "pairing" },
      accounts: {
        assistant: {
          name: "メインアシスタント",
          homeserver: "https://matrix.example.org",
          accessToken: "syt_assistant_***",
          encryption: true,
        },
        alerts: {
          name: "アラートボット",
          homeserver: "https://matrix.example.org",
          accessToken: "syt_alerts_***",
          dm: { policy: "allowlist", allowFrom: ["@admin:example.org"] },
        },
      },
    },
  },
}
```

注意点:

-   アカウントの起動は、モジュールインポートの競合状態を避けるために直列化されます。
-   環境変数（`MATRIX_HOMESERVER`、`MATRIX_ACCESS_TOKEN` など）は**デフォルト**アカウントにのみ適用されます。
-   ベースチャネル設定（DMポリシー、グループポリシー、メンションゲーティングなど）は、アカウントごとにオーバーライドされない限り、すべてのアカウントに適用されます。
-   `bindings[].match.accountId` を使用して、各アカウントを異なるエージェントにルーティングします。
-   暗号化状態はアカウント + アクセストークンごとに保存されます（アカウントごとに別々の鍵ストア）。

## ルーティングモデル

-   返信は常に Matrix に戻ります。
-   DM はエージェントのメインセッションを共有します。ルームはグループセッションにマッピングされます。

## アクセス制御 (DM)

-   デフォルト: `channels.matrix.dm.policy = "pairing"`。未知の送信者にはペアリングコードが発行されます。
-   承認方法:
    -   `openclaw pairing list matrix`
    -   `openclaw pairing approve matrix `
-   公開 DM: `channels.matrix.dm.policy="open"` に加えて `channels.matrix.dm.allowFrom=["*"]`。
-   `channels.matrix.dm.allowFrom` は完全な Matrix ユーザーID（例: `@user:server`）を受け入れます。設定ウィザードは、ディレクトリ検索で単一の完全一致が見つかった場合、表示名をユーザーIDに解決します。
-   表示名やローカルパートのみ（例: `"Alice"` や `"alice"`）は使用しないでください。曖昧であり、許可リストのマッチングでは無視されます。完全な `@user:server` ID を使用してください。

## ルーム (グループ)

-   デフォルト: `channels.matrix.groupPolicy = "allowlist"`（メンションゲート付き）。未設定時のデフォルトをオーバーライドするには `channels.defaults.groupPolicy` を使用します。
-   実行時注意: `channels.matrix` が完全に欠落している場合、実行時はルームチェックで `groupPolicy="allowlist"` にフォールバックします（`channels.defaults.groupPolicy` が設定されていても）。
-   `channels.matrix.groups` でルームを許可リストに登録します（ルームIDまたはエイリアス。名前は、ディレクトリ検索で単一の完全一致が見つかった場合にIDに解決されます）:

```json
{
  channels: {
    matrix: {
      groupPolicy: "allowlist",
      groups: {
        "!roomId:example.org": { allow: true },
        "#alias:example.org": { allow: true },
      },
      groupAllowFrom: ["@owner:example.org"],
    },
  },
}
```

-   `requireMention: false` は、そのルームでの自動返信を有効にします。
-   `groups."*"` は、ルーム全体でのメンションゲーティングのデフォルトを設定できます。
-   `groupAllowFrom` は、ルーム内でボットをトリガーできる送信者を制限します（完全な Matrix ユーザーID）。
-   ルームごとの `users` 許可リストは、特定のルーム内の送信者をさらに制限できます（完全な Matrix ユーザーIDを使用）。
-   設定ウィザードは、ルーム許可リスト（ルームID、エイリアス、または名前）を求め、名前は正確で一意の一致があった場合のみ解決します。
-   起動時、OpenClaw は許可リスト内のルーム/ユーザー名をIDに解決し、マッピングをログに記録します。解決されなかったエントリは許可リストのマッチングでは無視されます。
-   招待はデフォルトで自動参加します。`channels.matrix.autoJoin` と `channels.matrix.autoJoinAllowlist` で制御します。
-   **ルームを許可しない**場合は、`channels.matrix.groupPolicy: "disabled"` を設定します（または空の許可リストを維持します）。
-   レガシーキー: `channels.matrix.rooms`（`groups` と同じ形状）。

## スレッド

-   返信スレッドはサポートされています。
-   `channels.matrix.threadReplies` は、返信がスレッド内に留まるかどうかを制御します:
    -   `off`, `inbound` (デフォルト), `always`
-   `channels.matrix.replyToMode` は、スレッド内で返信しない場合の返信先メタデータを制御します:
    -   `off` (デフォルト), `first`, `all`

## 機能

| 機能 | ステータス |
| --- | --- |
| ダイレクトメッセージ | ✅ サポート |
| ルーム | ✅ サポート |
| スレッド | ✅ サポート |
| メディア | ✅ サポート |
| E2EE | ✅ サポート（暗号化モジュール必須） |
| リアクション | ✅ サポート（ツール経由で送信/読み取り） |
| 投票 | ✅ 送信サポート。受信投票開始はテキストに変換されます（応答/終了は無視） |
| 位置情報 | ✅ サポート（geo URI。高度は無視） |
| ネイティブコマンド | ✅ サポート |

## トラブルシューティング

まずこのラダーを実行してください:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

次に、必要に応じて DM ペアリング状態を確認します:

```bash
openclaw pairing list matrix
```

一般的な失敗:

-   ログイン済みだがルームメッセージが無視される: `groupPolicy` またはルーム許可リストによってルームがブロックされています。
-   DM が無視される: `channels.matrix.dm.policy="pairing"` 時に送信者が承認待ちです。
-   暗号化ルームが失敗する: 暗号化サポートまたは暗号化設定の不一致。

トリアージフローについては: [/channels/troubleshooting](./troubleshooting.md)。

## 設定リファレンス (Matrix)

完全な設定: [設定](../gateway/configuration.md) プロバイダーオプション:

-   `channels.matrix.enabled`: チャネル起動の有効/無効。
-   `channels.matrix.homeserver`: ホームサーバーURL。
-   `channels.matrix.userId`: Matrix ユーザーID（アクセストークン使用時はオプション）。
-   `channels.matrix.accessToken`: アクセストークン。
-   `channels.matrix.password`: ログイン用パスワード（トークン保存）。
-   `channels.matrix.deviceName`: デバイス表示名。
-   `channels.matrix.encryption`: E2EE を有効化（デフォルト: false）。
-   `channels.matrix.initialSyncLimit`: 初期同期制限。
-   `channels.matrix.threadReplies`: `off | inbound | always`（デフォルト: inbound）。
-   `channels.matrix.textChunkLimit`: 送信テキストのチャンクサイズ（文字数）。
-   `channels.matrix.chunkMode`: `length`（デフォルト）または `newline` で、長さチャンク化の前に空白行（段落境界）で分割。
-   `channels.matrix.dm.policy`: `pairing | allowlist | open | disabled`（デフォルト: pairing）。
-   `channels.matrix.dm.allowFrom`: DM 許可リスト（完全な Matrix ユーザーID）。`open` には `"*"` が必要です。ウィザードは可能な場合、名前をIDに解決します。
-   `channels.matrix.groupPolicy`: `allowlist | open | disabled`（デフォルト: allowlist）。
-   `channels.matrix.groupAllowFrom`: グループメッセージの許可リスト送信者（完全な Matrix ユーザーID）。
-   `channels.matrix.allowlistOnly`: DM + ルームの許可リストルールを強制。
-   `channels.matrix.groups`: グループ許可リスト + ルームごとの設定マップ。
-   `channels.matrix.rooms`: レガシーグループ許可リスト/設定。
-   `channels.matrix.replyToMode`: スレッド/タグの返信先モード。
-   `channels.matrix.mediaMaxMb`: 受信/送信メディア上限（MB）。
-   `channels.matrix.autoJoin`: 招待処理（`always | allowlist | off`、デフォルト: always）。
-   `channels.matrix.autoJoinAllowlist`: 自動参加を許可するルームID/エイリアス。
-   `channels.matrix.accounts`: アカウントIDでキー付けされたマルチアカウント設定（各アカウントはトップレベル設定を継承）。
-   `channels.matrix.actions`: アクションごとのツールゲーティング（リアクション/メッセージ/ピン/メンバー情報/チャネル情報）。

[LINE](./line.md)[Mattermost](./mattermost.md)