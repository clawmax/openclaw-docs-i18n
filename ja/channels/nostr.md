

  メッセージングプラットフォーム

  
# Nostr

**ステータス:** オプションプラグイン（デフォルトで無効）。Nostrは分散型ソーシャルネットワーキングプロトコルです。このチャネルは、OpenClawがNIP-04経由で暗号化ダイレクトメッセージ（DM）を受信および応答できるようにします。

## インストール（オンデマンド）

### オンボーディング（推奨）

-   オンボーディングウィザード（`openclaw onboard`）および`openclaw channels add`は、オプションのチャネルプラグインをリスト表示します。
-   Nostrを選択すると、オンデマンドでプラグインをインストールするよう促されます。

インストールのデフォルト:

-   **開発チャネル + gitチェックアウト利用時:** ローカルのプラグインパスを使用します。
-   **安定版/ベータ版:** npmからダウンロードします。

プロンプトでの選択は常に上書きできます。

### 手動インストール

```bash
openclaw plugins install @openclaw/nostr
```

ローカルチェックアウトを使用する（開発ワークフロー）:

```bash
openclaw plugins install --link <path-to-openclaw>/extensions/nostr
```

プラグインのインストールまたは有効化後は、Gatewayを再起動してください。

## クイックセットアップ

1.  Nostrキーペアを生成する（必要な場合）:

```bash
# nakを使用
nak key generate
```

2.  設定に追加する:

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}"
    }
  }
}
```

3.  キーをエクスポートする:

```bash
export NOSTR_PRIVATE_KEY="nsec1..."
```

4.  Gatewayを再起動する。

## 設定リファレンス

| キー | タイプ | デフォルト | 説明 |
| --- | --- | --- | --- |
| `privateKey` | string | 必須 | `nsec`または16進数形式の秘密鍵 |
| `relays` | string\[\] | `['wss://relay.damus.io', 'wss://nos.lol']` | リレーURL（WebSocket） |
| `dmPolicy` | string | `pairing` | DMアクセスポリシー |
| `allowFrom` | string\[\] | `[]` | 許可された送信者の公開鍵 |
| `enabled` | boolean | `true` | チャネルの有効/無効 |
| `name` | string | \- | 表示名 |
| `profile` | object | \- | NIP-01プロファイルメタデータ |

## プロファイルメタデータ

プロファイルデータはNIP-01の`kind:0`イベントとして公開されます。Control UI（Channels -> Nostr -> Profile）から管理するか、設定で直接設定できます。例:

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "profile": {
        "name": "openclaw",
        "displayName": "OpenClaw",
        "about": "パーソナルアシスタントDMボット",
        "picture": "https://example.com/avatar.png",
        "banner": "https://example.com/banner.png",
        "website": "https://example.com",
        "nip05": "openclaw@example.com",
        "lud16": "openclaw@example.com"
      }
    }
  }
}
```

注意:

-   プロファイルURLは`https://`を使用する必要があります。
-   リレーからのインポートはフィールドをマージし、ローカルの上書きを保持します。

## アクセス制御

### DMポリシー

-   **pairing** (デフォルト): 未知の送信者にはペアリングコードが送られます。
-   **allowlist**: `allowFrom`に含まれる公開鍵のみがDMできます。
-   **open**: 公開インバウンドDM（`allowFrom: ["*"]`が必要）。
-   **disabled**: インバウンドDMを無視します。

### 許可リストの例

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "dmPolicy": "allowlist",
      "allowFrom": ["npub1abc...", "npub1xyz..."]
    }
  }
}
```

## キー形式

受け入れられる形式:

-   **秘密鍵:** `nsec...` または64文字の16進数
-   **公開鍵 (`allowFrom`):** `npub...` または16進数

## リレー

デフォルト: `relay.damus.io` および `nos.lol`.

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "relays": ["wss://relay.damus.io", "wss://relay.primal.net", "wss://nostr.wine"]
    }
  }
}
```

ヒント:

-   冗長性のために2-3つのリレーを使用します。
-   リレーが多すぎないようにします（遅延、重複）。
-   有料リレーは信頼性を向上させることがあります。
-   ローカルリレーはテストに適しています（`ws://localhost:7777`）。

## プロトコルサポート

| NIP | ステータス | 説明 |
| --- | --- | --- |
| NIP-01 | サポート済み | 基本イベント形式 + プロファイルメタデータ |
| NIP-04 | サポート済み | 暗号化DM（`kind:4`） |
| NIP-17 | 計画中 | ギフトラップされたDM |
| NIP-44 | 計画中 | バージョン付き暗号化 |

## テスト

### ローカルリレー

```bash
# strfryを起動
docker run -p 7777:7777 ghcr.io/hoytech/strfry
```

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "relays": ["ws://localhost:7777"]
    }
  }
}
```

### 手動テスト

1.  ボットの公開鍵（npub）をログから確認します。
2.  Nostrクライアント（Damus、Amethystなど）を開きます。
3.  ボットの公開鍵にDMを送信します。
4.  応答を確認します。

## トラブルシューティング

### メッセージを受信しない

-   秘密鍵が有効であることを確認してください。
-   リレーURLが到達可能であり、`wss://`（またはローカルの場合は`ws://`）を使用していることを確認してください。
-   `enabled`が`false`になっていないことを確認してください。
-   Gatewayログでリレー接続エラーを確認してください。

### 応答を送信しない

-   リレーが書き込みを受け入れているか確認してください。
-   アウトバウンド接続性を確認してください。
-   リレーのレート制限に注意してください。

### 重複した応答

-   複数のリレーを使用する場合は予期される動作です。
-   メッセージはイベントIDによって重複排除され、最初の配信のみが応答をトリガーします。

## セキュリティ

-   秘密鍵をコミットしないでください。
-   キーには環境変数を使用してください。
-   本番環境のボットには`allowlist`の使用を検討してください。

## 制限事項（MVP）

-   ダイレクトメッセージのみ（グループチャットなし）。
-   メディア添付ファイルなし。
-   NIP-04のみ（NIP-17ギフトラップは計画中）。

[Nextcloud Talk](./nextcloud-talk.md)[Signal](./signal.md)