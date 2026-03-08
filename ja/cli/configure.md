

  CLI コマンド

  
# configure

認証情報、デバイス、エージェントのデフォルトを設定するための対話型プロンプトです。注記: **Model** セクションには、`agents.defaults.models` 許可リスト（`/model` およびモデルピッカーに表示されるもの）のためのマルチセレクトが含まれるようになりました。ヒント: サブコマンドなしの `openclaw config` は同じウィザードを開きます。非対話的な編集には `openclaw config get|set|unset` を使用してください。関連項目:

-   ゲートウェイ設定リファレンス: [Configuration](../gateway/configuration.md)
-   Config CLI: [Config](./config.md)

注記:

-   ゲートウェイの実行場所を選択すると、常に `gateway.mode` が更新されます。必要なのがそれだけの場合は、他のセクションなしで「続行」を選択できます。
-   チャネル指向サービス（Slack/Discord/Matrix/Microsoft Teams）では、セットアップ中にチャネル/ルームの許可リストの入力を求めます。名前またはIDを入力できます。ウィザードは可能な場合、名前をIDに解決します。
-   デーモンインストールステップを実行する場合、トークン認証にはトークンが必要であり、`gateway.auth.token` は SecretRef で管理されます。configure は SecretRef を検証しますが、解決された平文のトークン値をスーパーバイザーサービスの環境メタデータに永続化しません。
-   トークン認証にトークンが必要で、設定されたトークン SecretRef が未解決の場合、configure はデーモンインストールをブロックし、実行可能な修正ガイダンスを表示します。
-   `gateway.auth.token` と `gateway.auth.password` の両方が設定されており、`gateway.auth.mode` が未設定の場合、configure はモードが明示的に設定されるまでデーモンインストールをブロックします。

## 例

```bash
openclaw configure
openclaw configure --section model --section channels
```

[config](./config.md)[cron](./cron.md)

---