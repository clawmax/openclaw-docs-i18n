

  メッセージングプラットフォーム

  
# Zalo

ステータス: 実験的。DMはサポートされています。グループ処理は明示的なグループポリシー制御で利用可能です。

## プラグイン必須

Zaloはプラグインとして提供され、コアインストールにはバンドルされていません。

-   CLI経由でインストール: `openclaw plugins install @openclaw/zalo`
-   または、オンボーディング中に**Zalo**を選択し、インストールプロンプトを確認します
-   詳細: [プラグイン](../tools/plugin.md)

## クイックセットアップ (初心者向け)

1.  Zaloプラグインをインストール:
    -   ソースチェックアウトから: `openclaw plugins install ./extensions/zalo`
    -   npmから (公開されている場合): `openclaw plugins install @openclaw/zalo`
    -   または、オンボーディングで**Zalo**を選択し、インストールプロンプトを確認します
2.  トークンを設定:
    -   環境変数: `ZALO_BOT_TOKEN=...`
    -   または設定: `channels.zalo.botToken: "..."`.
3.  ゲートウェイを再起動 (またはオンボーディングを完了)。
4.  DMアクセスはデフォルトでペアリングです。最初のコンタクト時にペアリングコードを承認します。

最小構成:

```json
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "12345689:abc-xyz",
      dmPolicy: "pairing",
    },
  },
}
```

## 概要

Zaloはベトナムに焦点を当てたメッセージングアプリです。そのBot APIにより、Gatewayは1対1の会話用のボットを実行できます。Zaloへの確定的なルーティングを望むサポートや通知に適しています。

-   Gatewayが所有するZalo Bot APIチャネル。
-   確定的なルーティング: 返信はZaloに戻ります。モデルがチャネルを選択することはありません。
-   DMはエージェントのメインセッションを共有します。
-   グループはポリシー制御 (`groupPolicy` + `groupAllowFrom`) でサポートされ、デフォルトはフェイルクローズの許可リスト動作です。

## セットアップ (高速パス)

### 1) ボットトークンの作成 (Zalo Bot Platform)

1.  [https://bot.zaloplatforms.com](https://bot.zaloplatforms.com) にアクセスし、サインインします。
2.  新しいボットを作成し、設定を構成します。
3.  ボットトークンをコピーします (形式: `12345689:abc-xyz`)。

### 2) トークンの構成 (環境変数または設定)

例:

```json
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "12345689:abc-xyz",
      dmPolicy: "pairing",
    },
  },
}
```

環境変数オプション: `ZALO_BOT_TOKEN=...` (デフォルトアカウントのみ有効)。マルチアカウントサポート: `channels.zalo.accounts` を使用し、アカウントごとのトークンとオプションの `name` を設定します。

3.  ゲートウェイを再起動します。Zaloはトークンが解決されると起動します (環境変数または設定)。
4.  DMアクセスはデフォルトでペアリングです。ボットが最初に連絡されたときにコードを承認します。

## 動作の仕組み

-   受信メッセージは、メディアプレースホルダーとともに共有チャネルエンベロープに正規化されます。
-   返信は常に同じZaloチャットにルーティングされます。
-   デフォルトはロングポーリングです。ウェブフックモードは `channels.zalo.webhookUrl` で利用可能です。

## 制限

-   送信テキストは2000文字に分割されます (Zalo API制限)。
-   メディアのダウンロード/アップロードは `channels.zalo.mediaMaxMb` (デフォルト5) で制限されます。
-   ストリーミングは、2000文字制限により有用性が低いため、デフォルトでブロックされます。

## アクセス制御 (DM)

### DMアクセス

-   デフォルト: `channels.zalo.dmPolicy = "pairing"`。未知の送信者はペアリングコードを受け取り、承認されるまでメッセージは無視されます (コードは1時間で期限切れ)。
-   承認方法:
    -   `openclaw pairing list zalo`
    -   `openclaw pairing approve zalo `
-   ペアリングはデフォルトのトークン交換です。詳細: [ペアリング](./pairing.md)
-   `channels.zalo.allowFrom` は数値のユーザーIDを受け入れます (ユーザー名ルックアップは利用不可)。

## アクセス制御 (グループ)

-   `channels.zalo.groupPolicy` はグループ受信処理を制御します: `open | allowlist | disabled`。
-   デフォルト動作はフェイルクローズ: `allowlist`。
-   `channels.zalo.groupAllowFrom` は、グループ内でボットをトリガーできる送信者IDを制限します。
-   `groupAllowFrom` が設定されていない場合、Zaloは送信者チェックに `allowFrom` にフォールバックします。
-   `groupPolicy: "disabled"` はすべてのグループメッセージをブロックします。
-   `groupPolicy: "open"` は任意のグループメンバーを許可します (メンションゲート付き)。
-   実行時注意: `channels.zalo` が完全に欠落している場合、安全性のため、実行時は `groupPolicy="allowlist"` にフォールバックします。

## ロングポーリング vs ウェブフック

-   デフォルト: ロングポーリング (公開URL不要)。
-   ウェブフックモード: `channels.zalo.webhookUrl` と `channels.zalo.webhookSecret` を設定します。
    -   ウェブフックシークレットは8〜256文字である必要があります。
    -   ウェブフックURLはHTTPSを使用する必要があります。
    -   Zaloは `X-Bot-Api-Secret-Token` ヘッダー付きでイベントを送信し、検証します。
    -   Gateway HTTPは `channels.zalo.webhookPath` (デフォルトはウェブフックURLのパス) でウェブフックリクエストを処理します。
    -   リクエストは `Content-Type: application/json` (または `+json` メディアタイプ) を使用する必要があります。
    -   重複イベント (`event_name + message_id`) は短いリプレイウィンドウで無視されます。
    -   バーストトラフィックはパス/ソースごとにレート制限され、HTTP 429を返す可能性があります。

**注意:** getUpdates (ポーリング) とウェブフックは、Zalo APIドキュメントによると相互排他的です。

## サポートされるメッセージタイプ

-   **テキストメッセージ**: 2000文字の分割による完全サポート。
-   **画像メッセージ**: 受信画像のダウンロードと処理。`sendPhoto` 経由で画像を送信。
-   **ステッカー**: ログ記録されますが、完全には処理されません (エージェント応答なし)。
-   **サポートされていないタイプ**: ログ記録されます (例: 保護されたユーザーからのメッセージ)。

## 機能

| 機能 | ステータス |
| --- | --- |
| ダイレクトメッセージ | ✅ サポート |
| グループ | ⚠️ ポリシー制御付きでサポート (デフォルトは許可リスト) |
| メディア (画像) | ✅ サポート |
| リアクション | ❌ 未サポート |
| スレッド | ❌ 未サポート |
| 投票 | ❌ 未サポート |
| ネイティブコマンド | ❌ 未サポート |
| ストリーミング | ⚠️ ブロック (2000文字制限) |

## 配信ターゲット (CLI/クローン)

-   チャットIDをターゲットとして使用します。
-   例: `openclaw message send --channel zalo --target 123456789 --message "hi"`.

## トラブルシューティング

**ボットが応答しない:**

-   トークンが有効か確認: `openclaw channels status --probe`
-   送信者が承認されているか確認 (ペアリングまたはallowFrom)
-   ゲートウェイログを確認: `openclaw logs --follow`

**ウェブフックがイベントを受信しない:**

-   ウェブフックURLがHTTPSを使用していることを確認
-   シークレットトークンが8〜256文字であることを確認
-   ゲートウェイHTTPエンドポイントが構成されたパスで到達可能であることを確認
-   getUpdatesポーリングが実行されていないことを確認 (相互排他的)

## 構成リファレンス (Zalo)

完全な構成: [構成](../gateway/configuration.md) プロバイダーオプション:

-   `channels.zalo.enabled`: チャネル起動の有効/無効。
-   `channels.zalo.botToken`: Zalo Bot Platformからのボットトークン。
-   `channels.zalo.tokenFile`: ファイルパスからトークンを読み込み。
-   `channels.zalo.dmPolicy`: `pairing | allowlist | open | disabled` (デフォルト: pairing)。
-   `channels.zalo.allowFrom`: DM許可リスト (ユーザーID)。`open` には `"*"` が必要です。ウィザードは数値IDを要求します。
-   `channels.zalo.groupPolicy`: `open | allowlist | disabled` (デフォルト: allowlist)。
-   `channels.zalo.groupAllowFrom`: グループ送信者許可リスト (ユーザーID)。未設定時は `allowFrom` にフォールバック。
-   `channels.zalo.mediaMaxMb`: 受信/送信メディア上限 (MB、デフォルト5)。
-   `channels.zalo.webhookUrl`: ウェブフックモードを有効化 (HTTPS必須)。
-   `channels.zalo.webhookSecret`: ウェブフックシークレット (8-256文字)。
-   `channels.zalo.webhookPath`: ゲートウェイHTTPサーバー上のウェブフックパス。
-   `channels.zalo.proxy`: APIリクエスト用プロキシURL。

マルチアカウントオプション:

-   `channels.zalo.accounts..botToken`: アカウントごとのトークン。
-   `channels.zalo.accounts..tokenFile`: アカウントごとのトークンファイル。
-   `channels.zalo.accounts..name`: 表示名。
-   `channels.zalo.accounts..enabled`: アカウントの有効/無効。
-   `channels.zalo.accounts..dmPolicy`: アカウントごとのDMポリシー。
-   `channels.zalo.accounts..allowFrom`: アカウントごとの許可リスト。
-   `channels.zalo.accounts..groupPolicy`: アカウントごとのグループポリシー。
-   `channels.zalo.accounts..groupAllowFrom`: アカウントごとのグループ送信者許可リスト。
-   `channels.zalo.accounts..webhookUrl`: アカウントごとのウェブフックURL。
-   `channels.zalo.accounts..webhookSecret`: アカウントごとのウェブフックシークレット。
-   `channels.zalo.accounts..webhookPath`: アカウントごとのウェブフックパス。
-   `channels.zalo.accounts..proxy`: アカウントごとのプロキシURL。

[WhatsApp](./whatsapp.md)[Zalo Personal](./zalouser.md)