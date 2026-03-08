

  メッセージングプラットフォーム

  
# LINE

LINEはLINE Messaging APIを介してOpenClawに接続します。このプラグインはゲートウェイ上でウェブフックレシーバーとして動作し、認証にはチャネルアクセストークンとチャネルシークレットを使用します。ステータス: プラグイン経由でサポートされています。ダイレクトメッセージ、グループチャット、メディア、位置情報、Flexメッセージ、テンプレートメッセージ、クイックリプライがサポートされています。リアクションとスレッドはサポートされていません。

## 必要なプラグイン

LINEプラグインをインストールします:

```bash
openclaw plugins install @openclaw/line
```

ローカルチェックアウト（gitリポジトリから実行する場合）:

```bash
openclaw plugins install ./extensions/line
```

## セットアップ

1.  LINE Developersアカウントを作成し、コンソールを開きます: [https://developers.line.biz/console/](https://developers.line.biz/console/)
2.  プロバイダーを作成（または選択）し、**Messaging API**チャネルを追加します。
3.  チャネル設定から**チャネルアクセストークン**と**チャネルシークレット**をコピーします。
4.  Messaging API設定で**ウェブフックを利用する**を有効にします。
5.  ウェブフックURLをゲートウェイエンドポイントに設定します（HTTPS必須）:

```
https://gateway-host/line/webhook
```

ゲートウェイは、LINEのウェブフック検証（GET）と受信イベント（POST）に応答します。カスタムパスが必要な場合は、`channels.line.webhookPath` または `channels.line.accounts..webhookPath` を設定し、URLをそれに合わせて更新してください。セキュリティ上の注意:

-   LINEの署名検証はボディに依存します（生ボディに対するHMAC）。そのため、OpenClawは検証前に厳格な事前認証ボディ制限とタイムアウトを適用します。

## 設定

最小限の設定:

```json
{
  channels: {
    line: {
      enabled: true,
      channelAccessToken: "LINE_CHANNEL_ACCESS_TOKEN",
      channelSecret: "LINE_CHANNEL_SECRET",
      dmPolicy: "pairing",
    },
  },
}
```

環境変数（デフォルトアカウントのみ）:

-   `LINE_CHANNEL_ACCESS_TOKEN`
-   `LINE_CHANNEL_SECRET`

トークン/シークレットファイル:

```json
{
  channels: {
    line: {
      tokenFile: "/path/to/line-token.txt",
      secretFile: "/path/to/line-secret.txt",
    },
  },
}
```

複数アカウント:

```json
{
  channels: {
    line: {
      accounts: {
        marketing: {
          channelAccessToken: "...",
          channelSecret: "...",
          webhookPath: "/line/marketing",
        },
      },
    },
  },
}
```

## アクセス制御

ダイレクトメッセージはデフォルトでペアリングです。未知の送信者にはペアリングコードが送られ、承認されるまでメッセージは無視されます。

```bash
openclaw pairing list line
openclaw pairing approve line <CODE>
```

許可リストとポリシー:

-   `channels.line.dmPolicy`: `pairing | allowlist | open | disabled`
-   `channels.line.allowFrom`: DM用に許可されたLINEユーザーID
-   `channels.line.groupPolicy`: `allowlist | open | disabled`
-   `channels.line.groupAllowFrom`: グループ用に許可されたLINEユーザーID
-   グループごとのオーバーライド: `channels.line.groups..allowFrom`
-   実行時注意: `channels.line` が完全に欠落している場合、ランタイムはグループチェックで `groupPolicy="allowlist"` にフォールバックします（`channels.defaults.groupPolicy` が設定されていても）。

LINE IDは大文字小文字を区別します。有効なIDの例:

-   ユーザー: `U` + 32桁の16進文字
-   グループ: `C` + 32桁の16進文字
-   ルーム: `R` + 32桁の16進文字

## メッセージの動作

-   テキストは5000文字で分割されます。
-   マークダウン書式は除去されます。コードブロックやテーブルは可能な限りFlexカードに変換されます。
-   ストリーミング応答はバッファリングされます。エージェントが処理中、LINEはローディングアニメーション付きの完全なチャンクを受け取ります。
-   メディアダウンロードは `channels.line.mediaMaxMb`（デフォルト10）で制限されます。

## チャネルデータ（リッチメッセージ）

`channelData.line` を使用して、クイックリプライ、位置情報、Flexカード、またはテンプレートメッセージを送信します。

```json
{
  text: "どうぞ",
  channelData: {
    line: {
      quickReplies: ["ステータス", "ヘルプ"],
      location: {
        title: "オフィス",
        address: "123 Main St",
        latitude: 35.681236,
        longitude: 139.767125,
      },
      flexMessage: {
        altText: "ステータスカード",
        contents: {
          /* Flexペイロード */
        },
      },
      templateMessage: {
        type: "confirm",
        text: "続行しますか？",
        confirmLabel: "はい",
        confirmData: "yes",
        cancelLabel: "いいえ",
        cancelData: "no",
      },
    },
  },
}
```

LINEプラグインには、Flexメッセージプリセット用の `/card` コマンドも同梱されています:

```bash
/card info "ようこそ" "ご参加ありがとうございます！"
```

## トラブルシューティング

-   **ウェブフック検証が失敗する:** ウェブフックURLがHTTPSであり、`channelSecret`がLINEコンソールと一致していることを確認してください。
-   **受信イベントがない:** ウェブフックパスが `channels.line.webhookPath` と一致し、ゲートウェイがLINEから到達可能であることを確認してください。
-   **メディアダウンロードエラー:** メディアがデフォルト制限を超える場合は、`channels.line.mediaMaxMb` を引き上げてください。

[IRC](./irc.md)[Matrix](./matrix.md)

---