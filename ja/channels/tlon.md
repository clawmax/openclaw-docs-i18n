

  メッセージングプラットフォーム

  
# Tlon

Tlon は Urbit 上に構築された分散型メッセンジャーです。OpenClaw はあなたの Urbit シップに接続し、DM やグループチャットメッセージに応答できます。グループへの返信はデフォルトで @メンションを必要とし、許可リストを通じてさらに制限することができます。ステータス: プラグイン経由でサポートされています。DM、グループメンション、スレッド返信、リッチテキストフォーマット、画像アップロードがサポートされています。リアクションと投票はまだサポートされていません。

## プラグイン必須

Tlon はプラグインとして提供され、コアインストールにはバンドルされていません。CLI (npm レジストリ) 経由でインストールします:

```bash
openclaw plugins install @openclaw/tlon
```

ローカルチェックアウト (git リポジトリから実行している場合):

```bash
openclaw plugins install ./extensions/tlon
```

詳細: [プラグイン](../tools/plugin.md)

## セットアップ

1.  Tlon プラグインをインストールします。
2.  シップの URL とログインコードを準備します。
3.  `channels.tlon` を設定します。
4.  ゲートウェイを再起動します。
5.  ボットに DM を送るか、グループチャンネルでメンションします。

最小限の設定 (単一アカウント):

```json
{
  channels: {
    tlon: {
      enabled: true,
      ship: "~sampel-palnet",
      url: "https://your-ship-host",
      code: "lidlut-tabwed-pillex-ridrup",
      ownerShip: "~your-main-ship", // 推奨: あなたのシップ、常に許可されます
    },
  },
}
```

## プライベート/LAN シップ

デフォルトでは、OpenClaw は SSRF 保護のため、プライベート/内部ホスト名と IP 範囲をブロックします。シップがプライベートネットワーク (localhost、LAN IP、内部ホスト名) で実行されている場合は、明示的にオプトインする必要があります:

```json
{
  channels: {
    tlon: {
      url: "http://localhost:8080",
      allowPrivateNetwork: true,
    },
  },
}
```

これは以下のような URL に適用されます:

-   `http://localhost:8080`
-   `http://192.168.x.x:8080`
-   `http://my-ship.local:8080`

⚠️ ローカルネットワークを信頼する場合にのみ有効にしてください。この設定は、シップ URL へのリクエストに対する SSRF 保護を無効にします。

## グループチャンネル

自動検出はデフォルトで有効です。チャンネルを手動でピン留めすることもできます:

```json
{
  channels: {
    tlon: {
      groupChannels: ["chat/~host-ship/general", "chat/~host-ship/support"],
    },
  },
}
```

自動検出を無効化:

```json
{
  channels: {
    tlon: {
      autoDiscoverChannels: false,
    },
  },
}
```

## アクセス制御

DM 許可リスト (空 = DM は許可されません、承認フローには `ownerShip` を使用):

```json
{
  channels: {
    tlon: {
      dmAllowlist: ["~zod", "~nec"],
    },
  },
}
```

グループ認可 (デフォルトで制限あり):

```json
{
  channels: {
    tlon: {
      defaultAuthorizedShips: ["~zod"],
      authorization: {
        channelRules: {
          "chat/~host-ship/general": {
            mode: "restricted",
            allowedShips: ["~zod", "~nec"],
          },
          "chat/~host-ship/announcements": {
            mode: "open",
          },
        },
      },
    },
  },
}
```

## オーナーと承認システム

オーナーシップを設定すると、未承認ユーザーが操作を試みた際に承認リクエストを受け取ります:

```json
{
  channels: {
    tlon: {
      ownerShip: "~your-main-ship",
    },
  },
}
```

オーナーシップは **すべての場所で自動的に認可されます** — DM 招待は自動承認され、チャンネルメッセージは常に許可されます。オーナーを `dmAllowlist` や `defaultAuthorizedShips` に追加する必要はありません。設定すると、オーナーは以下の通知を DM で受け取ります:

-   許可リストにないシップからの DM リクエスト
-   認可のないチャンネルでのメンション
-   グループ招待リクエスト

## 自動承認設定

DM 招待を自動承認 (dmAllowlist 内のシップに対して):

```json
{
  channels: {
    tlon: {
      autoAcceptDmInvites: true,
    },
  },
}
```

グループ招待を自動承認:

```json
{
  channels: {
    tlon: {
      autoAcceptGroupInvites: true,
    },
  },
}
```

## 配信ターゲット (CLI/cron)

`openclaw message send` または cron 配信で使用します:

-   DM: `~sampel-palnet` または `dm/~sampel-palnet`
-   グループ: `chat/~host-ship/channel` または `group:~host-ship/channel`

## バンドルされたスキル

Tlon プラグインには、Tlon 操作への CLI アクセスを提供するバンドルされたスキル ([`@tloncorp/tlon-skill`](https://github.com/tloncorp/tlon-skill)) が含まれています:

-   **連絡先**: プロフィールの取得/更新、連絡先の一覧表示
-   **チャンネル**: 一覧表示、作成、メッセージ投稿、履歴の取得
-   **グループ**: 一覧表示、作成、メンバー管理
-   **DM**: メッセージ送信、メッセージへのリアクション
-   **リアクション**: 投稿や DM への絵文字リアクションの追加/削除
-   **設定**: スラッシュコマンドによるプラグイン権限の管理

このスキルは、プラグインがインストールされると自動的に利用可能になります。

## 機能

| 機能 | ステータス |
| --- | --- |
| ダイレクトメッセージ | ✅ サポート済み |
| グループ/チャンネル | ✅ サポート済み (デフォルトでメンションゲートあり) |
| スレッド | ✅ サポート済み (スレッド内で自動返信) |
| リッチテキスト | ✅ Markdown を Tlon 形式に変換 |
| 画像 | ✅ Tlon ストレージにアップロード |
| リアクション | ✅ [バンドルされたスキル](#bundled-skill) 経由 |
| 投票 | ❌ まだサポートされていません |
| ネイティブコマンド | ✅ サポート済み (デフォルトでオーナーのみ) |

## トラブルシューティング

まずこのラダーを実行してください:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
```

一般的な失敗:

-   **DM が無視される**: 送信者が `dmAllowlist` に含まれておらず、承認フローのための `ownerShip` が設定されていない。
-   **グループメッセージが無視される**: チャンネルが検出されていないか、送信者が認可されていない。
-   **接続エラー**: シップ URL が到達可能か確認する。ローカルシップの場合は `allowPrivateNetwork` を有効にする。
-   **認証エラー**: ログインコードが最新であることを確認する (コードはローテーションします)。

## 設定リファレンス

完全な設定: [設定](../gateway/configuration.md) プロバイダーオプション:

-   `channels.tlon.enabled`: チャンネルの起動を有効/無効化。
-   `channels.tlon.ship`: ボットの Urbit シップ名 (例: `~sampel-palnet`)。
-   `channels.tlon.url`: シップ URL (例: `https://sampel-palnet.tlon.network`)。
-   `channels.tlon.code`: シップログインコード。
-   `channels.tlon.allowPrivateNetwork`: localhost/LAN URL を許可 (SSRF バイパス)。
-   `channels.tlon.ownerShip`: 承認システムのためのオーナーシップ (常に認可)。
-   `channels.tlon.dmAllowlist`: DM が許可されるシップ (空 = なし)。
-   `channels.tlon.autoAcceptDmInvites`: 許可リスト内のシップからの DM を自動承認。
-   `channels.tlon.autoAcceptGroupInvites`: すべてのグループ招待を自動承認。
-   `channels.tlon.autoDiscoverChannels`: グループチャンネルを自動検出 (デフォルト: true)。
-   `channels.tlon.groupChannels`: 手動でピン留めされたチャンネルネスト。
-   `channels.tlon.defaultAuthorizedShips`: すべてのチャンネルで認可されるシップ。
-   `channels.tlon.authorization.channelRules`: チャンネルごとの認可ルール。
-   `channels.tlon.showModelSignature`: メッセージにモデル名を追加。

## 注意事項

-   グループへの返信は、応答するためにメンション (例: `~your-bot-ship`) が必要です。
-   スレッド返信: 受信メッセージがスレッド内にある場合、OpenClaw はスレッド内で返信します。
-   リッチテキスト: Markdown フォーマット (太字、斜体、コード、見出し、リスト) は Tlon のネイティブ形式に変換されます。
-   画像: URL は Tlon ストレージにアップロードされ、画像ブロックとして埋め込まれます。

[Telegram](./telegram.md)[Twitch](./twitch.md)