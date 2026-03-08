

  CLI コマンド

  
# tui

Gateway に接続されたターミナル UI を開きます。関連項目:

-   TUI ガイド: [TUI](../web/tui.md)

注記:

-   `tui` は、可能な場合 (`env`/`file`/`exec` プロバイダー)、トークン/パスワード認証用に設定されたゲートウェイ認証 SecretRefs を解決します。

## 例

```bash
openclaw tui
openclaw tui --url ws://127.0.0.1:18789 --token <token>
openclaw tui --session main --deliver
```

[system](./system.md)[uninstall](./uninstall.md)