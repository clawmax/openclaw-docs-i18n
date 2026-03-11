

  設定

  
# チャネルトラブルシューティング

チャネルは接続しているが動作がおかしい場合は、このページを使用してください。

## コマンドラダー

まず以下の順序で実行してください:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

正常なベースライン:

-   `Runtime: running`
-   `RPC probe: ok`
-   チャネルプローブが connected/ready を示す

## WhatsApp

### WhatsApp 障害の特徴

| 症状 | 最速の確認方法 | 修正方法 |
| --- | --- | --- |
| 接続済みだがDM応答がない | `openclaw pairing list whatsapp` | 送信者を承認するか、DMポリシー/許可リストを切り替えます。 |
| グループメッセージが無視される | 設定内の `requireMention` とメンションパターンを確認 | ボットにメンションするか、そのグループのメンションポリシーを緩和します。 |
| ランダムな切断/再ログインループ | `openclaw channels status --probe` + ログ | 再ログインし、認証情報ディレクトリが正常であることを確認します。 |

完全なトラブルシューティング: [/channels/whatsapp#troubleshooting-quick](./whatsapp.md#troubleshooting-quick)

## Telegram

### Telegram 障害の特徴

| 症状 | 最速の確認方法 | 修正方法 |
| --- | --- | --- |
| `/start` は実行できるが、使用可能な応答フローがない | `openclaw pairing list telegram` | ペアリングを承認するか、DMポリシーを変更します。 |
| ボットはオンラインだがグループが沈黙したまま | メンション要件とボットのプライバシーモードを確認 | グループ可視性のためにプライバシーモードを無効にするか、ボットにメンションします。 |
| ネットワークエラーによる送信失敗 | ログを検査しTelegram API呼び出しの失敗を確認 | `api.telegram.org` へのDNS/IPv6/プロキシルーティングを修正します。 |
| アップグレード後、許可リストにブロックされる | `openclaw security audit` と設定の許可リスト | `openclaw doctor --fix` を実行するか、`@username` を数値の送信者IDに置き換えます。 |

完全なトラブルシューティング: [/channels/telegram#troubleshooting](./telegram.md#troubleshooting)

## Discord

### Discord 障害の特徴

| 症状 | 最速の確認方法 | 修正方法 |
| --- | --- | --- |
| ボットはオンラインだがギルド応答がない | `openclaw channels status --probe` | ギルド/チャネルを許可し、メッセージコンテンツインテントを確認します。 |
| グループメッセージが無視される | メンションゲーティングによるドロップのログを確認 | ボットにメンションするか、ギルド/チャネルの `requireMention: false` を設定します。 |
| DM応答がない | `openclaw pairing list discord` | DMペアリングを承認するか、DMポリシーを調整します。 |

完全なトラブルシューティング: [/channels/discord#troubleshooting](./discord.md#troubleshooting)

## Slack

### Slack 障害の特徴

| 症状 | 最速の確認方法 | 修正方法 |
| --- | --- | --- |
| ソケットモードは接続済みだが応答がない | `openclaw channels status --probe` | アプリトークン + ボットトークンと必要なスコープを確認します。 |
| DMがブロックされる | `openclaw pairing list slack` | ペアリングを承認するか、DMポリシーを緩和します。 |
| チャネルメッセージが無視される | `groupPolicy` とチャネル許可リストを確認 | チャネルを許可するか、ポリシーを `open` に切り替えます。 |

完全なトラブルシューティング: [/channels/slack#troubleshooting](./slack.md#troubleshooting)

## iMessage と BlueBubbles

### iMessage と BlueBubbles 障害の特徴

| 症状 | 最速の確認方法 | 修正方法 |
| --- | --- | --- |
| 受信イベントがない | Webhook/サーバーの到達性とアプリの権限を確認 | Webhook URLまたはBlueBubblesサーバーの状態を修正します。 |
| 送信はできるがmacOSで受信できない | Messages自動化のためのmacOSプライバシー権限を確認 | TCC権限を再付与し、チャネルプロセスを再起動します。 |
| DM送信者がブロックされる | `openclaw pairing list imessage` または `openclaw pairing list bluebubbles` | ペアリングを承認するか、許可リストを更新します。 |

完全なトラブルシューティング:

-   [/channels/imessage#troubleshooting-macos-privacy-and-security-tcc](./imessage.md#troubleshooting-macos-privacy-and-security-tcc)
-   [/channels/bluebubbles#troubleshooting](./bluebubbles.md#troubleshooting)

## Signal

### Signal 障害の特徴

| 症状 | 最速の確認方法 | 修正方法 |
| --- | --- | --- |
| デーモンは到達可能だがボットが沈黙 | `openclaw channels status --probe` | `signal-cli` デーモンのURL/アカウントと受信モードを確認します。 |
| DMがブロックされる | `openclaw pairing list signal` | 送信者を承認するか、DMポリシーを調整します。 |
| グループ応答がトリガーされない | グループ許可リストとメンションパターンを確認 | 送信者/グループを追加するか、ゲーティングを緩和します。 |

完全なトラブルシューティング: [/channels/signal#troubleshooting](./signal.md#troubleshooting)

## Matrix

### Matrix 障害の特徴

| 症状 | 最速の確認方法 | 修正方法 |
| --- | --- | --- |
| ログイン済みだがルームメッセージを無視する | `openclaw channels status --probe` | `groupPolicy` とルーム許可リストを確認します。 |
| DMが処理されない | `openclaw pairing list matrix` | 送信者を承認するか、DMポリシーを調整します。 |
| 暗号化ルームが失敗する | 暗号化モジュールと暗号化設定を確認 | 暗号化サポートを有効にし、ルームに再参加/同期します。 |

完全なトラブルシューティング: [/channels/matrix#troubleshooting](./matrix.md#troubleshooting)

[チャネルロケーションパーシング](./location.md)

---