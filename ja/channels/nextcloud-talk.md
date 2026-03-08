

  メッセージングプラットフォーム

  
# Nextcloud Talk

ステータス: プラグイン (ウェブフックボット) 経由でサポートされています。ダイレクトメッセージ、ルーム、リアクション、マークダウンメッセージがサポートされています。

## 必要なプラグイン

Nextcloud Talk はプラグインとして提供され、コアインストールにはバンドルされていません。CLI (npm レジストリ) 経由でインストールします:

```bash
openclaw plugins install @openclaw/nextcloud-talk
```

ローカルチェックアウト (git リポジトリから実行する場合):

```bash
openclaw plugins install ./extensions/nextcloud-talk
```

configure/オンボーディング中に Nextcloud Talk を選択し、git チェックアウトが検出された場合、OpenClaw は自動的にローカルインストールパスを提供します。詳細: [プラグイン](../tools/plugin.md)

## クイックセットアップ (初心者向け)

1.  Nextcloud Talk プラグインをインストールします。
2.  あなたの Nextcloud サーバー上で、ボットを作成します:
    
    コピー
    
    ```bash
    ./occ talk:bot:install "OpenClaw" "<shared-secret>" "<webhook-url>" --feature reaction
    ```
    
3.  対象ルームの設定でボットを有効にします。
4.  OpenClaw を設定します:
    -   設定: `channels.nextcloud-talk.baseUrl` + `channels.nextcloud-talk.botSecret`
    -   または環境変数: `NEXTCLOUD_TALK_BOT_SECRET` (デフォルトアカウントのみ)
5.  ゲートウェイを再起動します (またはオンボーディングを完了します)。

最小限の設定:

```json
{
  channels: {
    "nextcloud-talk": {
      enabled: true,
      baseUrl: "https://cloud.example.com",
      botSecret: "shared-secret",
      dmPolicy: "pairing",
    },
  },
}
```

## 注意事項

-   ボットは DM を開始できません。ユーザーが最初にボットにメッセージを送る必要があります。
-   ウェブフック URL はゲートウェイから到達可能でなければなりません。プロキシの背後にある場合は `webhookPublicUrl` を設定してください。
-   メディアアップロードはボット API でサポートされていません。メディアは URL として送信されます。
-   ウェブフックペイロードは DM とルームを区別しません。ルームタイプの検索を有効にするには `apiUser` + `apiPassword` を設定してください (設定しない場合、DM はルームとして扱われます)。

## アクセス制御 (DM)

-   デフォルト: `channels.nextcloud-talk.dmPolicy = "pairing"`。未知の送信者にはペアリングコードが送られます。
-   承認方法:
    -   `openclaw pairing list nextcloud-talk`
    -   `openclaw pairing approve nextcloud-talk `
-   公開 DM: `channels.nextcloud-talk.dmPolicy="open"` に加えて `channels.nextcloud-talk.allowFrom=["*"]`。
-   `allowFrom` は Nextcloud ユーザー ID のみと一致します。表示名は無視されます。

## ルーム (グループ)

-   デフォルト: `channels.nextcloud-talk.groupPolicy = "allowlist"` (メンションゲート)。
-   `channels.nextcloud-talk.rooms` でルームを許可リストに追加します:

```json
{
  channels: {
    "nextcloud-talk": {
      rooms: {
        "room-token": { requireMention: true },
      },
    },
  },
}
```

-   ルームを一切許可しない場合は、許可リストを空のままにするか、`channels.nextcloud-talk.groupPolicy="disabled"` を設定します。

## 機能

| 機能 | ステータス |
| --- | --- |
| ダイレクトメッセージ | サポート済み |
| ルーム | サポート済み |
| スレッド | 未サポート |
| メディア | URL のみ |
| リアクション | サポート済み |
| ネイティブコマンド | 未サポート |

## 設定リファレンス (Nextcloud Talk)

完全な設定: [設定](../gateway/configuration.md) プロバイダーオプション:

-   `channels.nextcloud-talk.enabled`: チャネルの起動を有効/無効にします。
-   `channels.nextcloud-talk.baseUrl`: Nextcloud インスタンスの URL。
-   `channels.nextcloud-talk.botSecret`: ボット共有シークレット。
-   `channels.nextcloud-talk.botSecretFile`: シークレットファイルのパス。
-   `channels.nextcloud-talk.apiUser`: ルーム検索 (DM 検出) 用の API ユーザー。
-   `channels.nextcloud-talk.apiPassword`: ルーム検索用の API/アプリパスワード。
-   `channels.nextcloud-talk.apiPasswordFile`: API パスワードファイルのパス。
-   `channels.nextcloud-talk.webhookPort`: ウェブフックリスナーポート (デフォルト: 8788)。
-   `channels.nextcloud-talk.webhookHost`: ウェブフックホスト (デフォルト: 0.0.0.0)。
-   `channels.nextcloud-talk.webhookPath`: ウェブフックパス (デフォルト: /nextcloud-talk-webhook)。
-   `channels.nextcloud-talk.webhookPublicUrl`: 外部から到達可能なウェブフック URL。
-   `channels.nextcloud-talk.dmPolicy`: `pairing | allowlist | open | disabled`。
-   `channels.nextcloud-talk.allowFrom`: DM 許可リスト (ユーザー ID)。`open` には `"*"` が必要です。
-   `channels.nextcloud-talk.groupPolicy`: `allowlist | open | disabled`。
-   `channels.nextcloud-talk.groupAllowFrom`: グループ許可リスト (ユーザー ID)。
-   `channels.nextcloud-talk.rooms`: ルームごとの設定と許可リスト。
-   `channels.nextcloud-talk.historyLimit`: グループ履歴制限 (0 で無効)。
-   `channels.nextcloud-talk.dmHistoryLimit`: DM 履歴制限 (0 で無効)。
-   `channels.nextcloud-talk.dms`: DM ごとのオーバーライド (historyLimit)。
-   `channels.nextcloud-talk.textChunkLimit`: 送信テキストのチャンクサイズ (文字数)。
-   `channels.nextcloud-talk.chunkMode`: `length` (デフォルト) または `newline` (長さチャンク化の前に空白行 (段落境界) で分割)。
-   `channels.nextcloud-talk.blockStreaming`: このチャネルのブロックストリーミングを無効にします。
-   `channels.nextcloud-talk.blockStreamingCoalesce`: ブロックストリーミングの結合調整。
-   `channels.nextcloud-talk.mediaMaxMb`: 受信メディアの上限 (MB)。

[Microsoft Teams](./msteams.md)[Nostr](./nostr.md)