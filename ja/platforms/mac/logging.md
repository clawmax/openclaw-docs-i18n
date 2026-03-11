

  macOSコンパニオンアプリ

  
# macOSロギング

## ローリング診断ファイルログ (デバッグペイン)

OpenClawはmacOSアプリのログをswift-log（デフォルトでは統一ロギング）経由でルーティングし、耐久性のあるキャプチャが必要な場合にローカルのローテーションするファイルログをディスクに書き込むことができます。

-   詳細度: **デバッグペイン → ログ → アプリロギング → 詳細度**
-   有効化: **デバッグペイン → ログ → アプリロギング → 「ローリング診断ログを書き込む (JSONL)」**
-   保存場所: `~/Library/Logs/OpenClaw/diagnostics.jsonl` (自動的にローテーション; 古いファイルは `.1`, `.2`, … というサフィックスが付きます)
-   クリア: **デバッグペイン → ログ → アプリロギング → 「クリア」**

注意:

-   これは**デフォルトではオフ**です。アクティブにデバッグしている間のみ有効にしてください。
-   ファイルは機密として扱い、レビューなしで共有しないでください。

## macOSでの統一ロギングのプライベートデータ

統一ロギングは、サブシステムが `privacy -off` をオプトインしない限り、ほとんどのペイロードを編集します。PeterによるmacOSの[ロギングプライバシーの小技](https://steipete.me/posts/2025/logging-privacy-shenanigans) (2025) に関する記事によると、これは `/Library/Preferences/Logging/Subsystems/` 内のplistファイルによってサブシステム名をキーとして制御されています。新しいログエントリのみがこのフラグを取得するため、問題を再現する前に有効にしてください。

## OpenClaw (ai.openclaw) で有効にする

-   まずplistを一時ファイルに書き込み、rootとしてアトミックにインストールします:

```bash
cat <<'EOF' >/tmp/ai.openclaw.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>DEFAULT-OPTIONS</key>
    <dict>
        <key>Enable-Private-Data</key>
        <true/>
    </dict>
</dict>
</plist>
EOF
sudo install -m 644 -o root -g wheel /tmp/ai.openclaw.plist /Library/Preferences/Logging/Subsystems/ai.openclaw.plist
```

-   再起動は不要です。logdはファイルを素早く認識しますが、新しいログ行のみがプライベートペイロードを含むようになります。
-   既存のヘルパーを使用してより詳細な出力を表示します。例: `./scripts/clawlog.sh --category WebChat --last 5m`。

## デバッグ後の無効化

-   オーバーライドを削除: `sudo rm /Library/Preferences/Logging/Subsystems/ai.openclaw.plist`。
-   必要に応じて `sudo log config --reload` を実行し、logdにオーバーライドを即座に破棄させます。
-   この設定には電話番号やメッセージ本文が含まれる可能性があることを忘れないでください。追加の詳細が必要な間だけplistを配置したままにしてください。

[メニューバーアイコン](./icon.md)[macOS権限](./permissions.md)