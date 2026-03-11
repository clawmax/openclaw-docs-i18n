

  CLI コマンド

  
# doctor

ゲートウェイとチャネルのヘルスチェックとクイックフィックス。関連情報:

-   トラブルシューティング: [トラブルシューティング](../gateway/troubleshooting.md)
-   セキュリティ監査: [セキュリティ](../gateway/security.md)

## 例

```bash
openclaw doctor
openclaw doctor --repair
openclaw doctor --deep
```

注記:

-   インタラクティブなプロンプト（キーチェーン/OAuth の修正など）は、標準入力が TTY であり、かつ `--non-interactive` が**設定されていない**場合にのみ実行されます。ヘッドレス実行（cron、Telegram、端末なし）ではプロンプトはスキップされます。
-   `--fix` (`--repair` のエイリアス) は、バックアップを `~/.openclaw/openclaw.json.bak` に書き込み、不明な設定キーを削除し、各削除をリスト表示します。
-   状態整合性チェックは、セッションディレクトリ内の孤立したトランスクリプトファイルを検出し、それらを `.deleted.<タイムスタンプ>` としてアーカイブして、安全にスペースを回収できるようになりました。
-   Doctor にはメモリ検索の準備状況チェックが含まれており、埋め込み認証情報が欠落している場合に `openclaw configure --section model` を推奨することがあります。
-   サンドボックスモードが有効になっているが Docker が利用できない場合、doctor は修正方法 (`Docker をインストール` または `openclaw config set agents.defaults.sandbox.mode off`) を含む高シグナルの警告を報告します。

## macOS: launchctl 環境変数オーバーライド

以前に `launchctl setenv OPENCLAW_GATEWAY_TOKEN ...` (または `...PASSWORD`) を実行した場合、その値は設定ファイルを上書きし、持続的な「認証されていません」エラーの原因となる可能性があります。

```bash
launchctl getenv OPENCLAW_GATEWAY_TOKEN
launchctl getenv OPENCLAW_GATEWAY_PASSWORD

launchctl unsetenv OPENCLAW_GATEWAY_TOKEN
launchctl unsetenv OPENCLAW_GATEWAY_PASSWORD
```

[ドキュメント](./docs.md)[ゲートウェイ](./gateway.md)