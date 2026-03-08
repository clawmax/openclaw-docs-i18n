

  CLIコマンド

  
# directory

サポートしているチャネルでのディレクトリ検索（連絡先/ピア、グループ、および「自分」）。

## 共通フラグ

-   `--channel `: チャネルID/エイリアス（複数のチャネルが設定されている場合は必須。1つだけ設定されている場合は自動）
-   `--account `: アカウントID（デフォルト: チャネルのデフォルト）
-   `--json`: JSONで出力

## 注意点

-   `directory` は、他のコマンド（特に `openclaw message send --target ...`）に貼り付けることができるIDを見つけるのに役立ちます。
-   多くのチャネルでは、結果はライブプロバイダーのディレクトリではなく、設定ベース（許可リスト / 設定済みグループ）です。
-   デフォルトの出力は `id`（および場合によっては `name`）がタブで区切られた形式です。スクリプト用には `--json` を使用してください。

## 結果をmessage sendで使用する

```bash
openclaw directory peers list --channel slack --query "U0"
openclaw message send --channel slack --target user:U012ABCDEF --message "hello"
```

## IDフォーマット（チャネル別）

-   WhatsApp: `+15551234567` (DM), `1234567890-1234567890@g.us` (グループ)
-   Telegram: `@username` または数値チャットID。グループは数値ID
-   Slack: `user:U…` および `channel:C…`
-   Discord: `user:` および `channel:`
-   Matrix (プラグイン): `user:@user:server`, `room:!roomId:server`, または `#alias:server`
-   Microsoft Teams (プラグイン): `user:` および `conversation:`
-   Zalo (プラグイン): ユーザーID (Bot API)
-   Zalo Personal / `zalouser` (プラグイン): `zca` (`me`, `friend list`, `group list`) からのスレッドID (DM/グループ)

## 自分自身（"me"）

```bash
openclaw directory self --channel zalouser
```

## ピア（連絡先/ユーザー）

```bash
openclaw directory peers list --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory peers list --channel zalouser --limit 50
```

## グループ

```bash
openclaw directory groups list --channel zalouser
openclaw directory groups list --channel zalouser --query "work"
openclaw directory groups members --channel zalouser --group-id <id>
```

[devices](./devices.md)[dns](./dns.md)