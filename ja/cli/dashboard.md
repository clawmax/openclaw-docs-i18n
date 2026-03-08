

  CLIコマンド

  
# dashboard

現在の認証情報を使用してControl UIを開きます。

```bash
openclaw dashboard
openclaw dashboard --no-open
```

注意点:

-   `dashboard` は、可能な場合に設定された `gateway.auth.token` SecretRefを解決します。
-   SecretRefで管理されているトークン（解決済みまたは未解決）の場合、`dashboard` は、ターミナル出力、クリップボード履歴、またはブラウザ起動引数で外部シークレットが露出するのを避けるため、トークン化されていないURLを表示/コピー/開きます。
-   `gateway.auth.token` がSecretRefで管理されているが、このコマンドパスで未解決の場合、コマンドは無効なトークンプレースホルダーを埋め込む代わりに、トークン化されていないURLと明確な修正ガイダンスを表示します。

[daemon](./daemon.md)[devices](./devices.md)