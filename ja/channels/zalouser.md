

  メッセージングプラットフォーム

  
# Zalo 個人アカウント

ステータス: 実験的。この連携は、OpenClaw内のネイティブな`zca-js`を使用して**個人Zaloアカウント**を自動化します。

> **警告:** これは非公式の連携であり、アカウントの停止/禁止につながる可能性があります。自己責任でご利用ください。

## 必要なプラグイン

Zalo Personalはプラグインとして提供され、コアインストールにはバンドルされていません。

-   CLI経由でインストール: `openclaw plugins install @openclaw/zalouser`
-   またはソースチェックアウトから: `openclaw plugins install ./extensions/zalouser`
-   詳細: [プラグイン](../tools/plugin.md)

外部の`zca`/`openzca` CLIバイナリは必要ありません。

## クイックセットアップ (初心者向け)

1.  プラグインをインストール (上記参照)。
2.  ログイン (QRコード、Gatewayマシン上で):
    -   `openclaw channels login --channel zalouser`
    -   ZaloモバイルアプリでQRコードをスキャンします。
3.  チャンネルを有効化:

```json
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing",
    },
  },
}
```

4.  Gatewayを再起動 (またはオンボーディングを完了)。
5.  DMアクセスはデフォルトでペアリング; 最初のコンタクトでペアリングコードを承認します。

## 概要

-   `zca-js`を介して完全にプロセス内で実行されます。
-   ネイティブなイベントリスナーを使用して受信メッセージを受信します。
-   JS APIを直接介して返信を送信します (テキスト/メディア/リンク)。
-   Zalo Bot APIが利用できない「個人アカウント」ユースケース向けに設計されています。

## 命名規則

チャンネルIDは`zalouser`とし、これが**個人Zaloユーザーアカウント** (非公式) を自動化することを明確にします。将来の公式Zalo API連携の可能性のために、`zalo`は予約済みです。

## IDの検索 (ディレクトリ)

ディレクトリCLIを使用して、ピア/グループとそのIDを検出します:

```bash
openclaw directory self --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory groups list --channel zalouser --query "work"
```

## 制限事項

-   送信テキストは約2000文字に分割されます (Zaloクライアントの制限)。
-   ストリーミングはデフォルトでブロックされています。

## アクセス制御 (DM)

`channels.zalouser.dmPolicy` は以下をサポート: `pairing | allowlist | open | disabled` (デフォルト: `pairing`)。`channels.zalouser.allowFrom` はユーザーIDまたは名前を受け付けます。オンボーディング中、名前はプラグインのプロセス内連絡先検索を使用してIDに解決されます。以下のコマンドで承認します:

-   `openclaw pairing list zalouser`
-   `openclaw pairing approve zalouser `

## グループアクセス (オプション)

-   デフォルト: `channels.zalouser.groupPolicy = "open"` (グループ許可)。未設定時のデフォルトを上書きするには `channels.defaults.groupPolicy` を使用します。
-   許可リストに制限するには:
    -   `channels.zalouser.groupPolicy = "allowlist"`
    -   `channels.zalouser.groups` (キーはグループIDまたは名前)
-   すべてのグループをブロック: `channels.zalouser.groupPolicy = "disabled"`。
-   設定ウィザードはグループ許可リストの入力を促すことができます。
-   起動時、OpenClawは許可リスト内のグループ/ユーザー名をIDに解決し、マッピングをログに記録します。解決できないエントリは入力されたまま保持されます。

例:

```json
{
  channels: {
    zalouser: {
      groupPolicy: "allowlist",
      groups: {
        "123456789": { allow: true },
        "Work Chat": { allow: true },
      },
    },
  },
}
```

### グループメンションゲーティング

-   `channels.zalouser.groups..requireMention` は、グループへの返信にメンションが必要かどうかを制御します。
-   解決順序: 正確なグループID/名前 -> 正規化されたグループスラッグ -> `*` -> デフォルト (`true`)。
-   これは許可リストされたグループとオープングループモードの両方に適用されます。

例:

```json
{
  channels: {
    zalouser: {
      groupPolicy: "allowlist",
      groups: {
        "*": { allow: true, requireMention: true },
        "Work Chat": { allow: true, requireMention: false },
      },
    },
  },
}
```

## マルチアカウント

アカウントはOpenClaw状態内の`zalouser`プロファイルにマッピングされます。例:

```json
{
  channels: {
    zalouser: {
      enabled: true,
      defaultAccount: "default",
      accounts: {
        work: { enabled: true, profile: "work" },
      },
    },
  },
}
```

## 入力中表示、リアクション、配信確認

-   OpenClawは返信を送信する前に (ベストエフォートで) 入力中イベントを送信します。
-   メッセージリアクションアクション`react`は、チャンネルアクション内の`zalouser`でサポートされています。
    -   特定のリアクション絵文字をメッセージから削除するには `remove: true` を使用します。
    -   リアクションのセマンティクス: [リアクション](../tools/reactions.md)
-   イベントメタデータを含む受信メッセージに対して、OpenClawは配信済み + 既読確認を送信します (ベストエフォート)。

## トラブルシューティング

**ログインが保持されない:**

-   `openclaw channels status --probe`
-   再ログイン: `openclaw channels logout --channel zalouser && openclaw channels login --channel zalouser`

**許可リスト/グループ名が解決されなかった:**

-   `allowFrom`/`groups`で数値ID、または正確なフレンド/グループ名を使用してください。

**古いCLIベースのセットアップからアップグレードした場合:**

-   古い外部`zca`プロセスの前提条件をすべて削除してください。
-   チャンネルは現在、外部CLIバイナリなしでOpenClaw内で完全に実行されます。

[Zalo](./zalo.md)[ペアリング](./pairing.md)