

  プロバイダー

  
# Qwen

Qwenは、Qwen CoderおよびQwen Visionモデル（1日2,000リクエスト、Qwenのレート制限対象）への無料枠OAuthフローを提供します。

## プラグインを有効化

```bash
openclaw plugins enable qwen-portal-auth
```

有効化後、Gatewayを再起動してください。

## 認証

```bash
openclaw models auth login --provider qwen-portal --set-default
```

これによりQwenのデバイスコードOAuthフローが実行され、プロバイダーエントリが`models.json`に書き込まれます（素早い切り替えのための`qwen`エイリアスも追加されます）。

## モデルID

-   `qwen-portal/coder-model`
-   `qwen-portal/vision-model`

モデルを切り替えるには:

```bash
openclaw models set qwen-portal/coder-model
```

## Qwen Code CLIログインの再利用

すでにQwen Code CLIでログイン済みの場合、OpenClawは認証ストアを読み込む際に`~/.qwen/oauth_creds.json`から認証情報を同期します。`models.providers.qwen-portal`エントリは依然として必要です（作成には上記のログインコマンドを使用してください）。

## 注意事項

-   トークンは自動更新されます。更新に失敗した場合やアクセスが取り消された場合は、ログインコマンドを再実行してください。
-   デフォルトのベースURL: `https://portal.qwen.ai/v1`（Qwenが異なるエンドポイントを提供する場合は`models.providers.qwen-portal.baseUrl`で上書き可能）。
-   プロバイダー全体のルールについては[モデルプロバイダー](../concepts/model-providers.md)を参照してください。

[Qianfan](./qianfan.md)[Synthetic](./synthetic.md)

---