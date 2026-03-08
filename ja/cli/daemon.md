

  CLI コマンド

  
# daemon

ゲートウェイサービス管理コマンドのレガシーエイリアスです。`openclaw daemon ...` は、`openclaw gateway ...` サービスコマンドと同じサービス制御インターフェースにマッピングされます。

## 使用方法

```bash
openclaw daemon status
openclaw daemon install
openclaw daemon start
openclaw daemon stop
openclaw daemon restart
openclaw daemon uninstall
```

## サブコマンド

-   `status`: サービスのインストール状態を表示し、ゲートウェイのヘルスをプローブします
-   `install`: サービスをインストールします (`launchd`/`systemd`/`schtasks`)
-   `uninstall`: サービスを削除します
-   `start`: サービスを起動します
-   `stop`: サービスを停止します
-   `restart`: サービスを再起動します

## 共通オプション

-   `status`: `--url`, `--token`, `--password`, `--timeout`, `--no-probe`, `--deep`, `--json`
-   `install`: `--port`, `--runtime <node|bun>`, `--token`, `--force`, `--json`
-   ライフサイクル (`uninstall|start|stop|restart`): `--json`

注記:

-   `status` は、可能な場合、プローブ認証のために設定された認証 SecretRef を解決します。
-   トークン認証にトークンが必要で、`gateway.auth.token` が SecretRef で管理されている場合、`install` は SecretRef が解決可能であることを検証しますが、解決されたトークンをサービス環境メタデータに永続化しません。
-   トークン認証にトークンが必要で、設定されたトークン SecretRef が未解決の場合、インストールは失敗します（フェイルクローズ）。
-   `gateway.auth.token` と `gateway.auth.password` の両方が設定されており、`gateway.auth.mode` が未設定の場合、モードが明示的に設定されるまでインストールはブロックされます。

## 推奨

現在のドキュメントと例については [`openclaw gateway`](./gateway.md) を使用してください。

[cron](./cron.md)[dashboard](./dashboard.md)