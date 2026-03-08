

  プロバイダー

  
# Qianfan

Qianfanは、BaiduのMaaSプラットフォームであり、単一のエンドポイントとAPIキーの背後にある多くのモデルにリクエストをルーティングする**統一API**を提供します。OpenAI互換であるため、ベースURLを切り替えるだけでほとんどのOpenAI SDKが動作します。

## 前提条件

1.  Qianfan APIアクセス権を持つBaidu Cloudアカウント
2.  QianfanコンソールからのAPIキー
3.  システムにインストールされたOpenClaw

## APIキーの取得

1.  [Qianfanコンソール](https://console.bce.baidu.com/qianfan/ais/console/apiKey)にアクセスします
2.  新しいアプリケーションを作成するか、既存のものを選択します
3.  APIキーを生成します（形式: `bce-v3/ALTAK-...`）
4.  OpenClawで使用するためにAPIキーをコピーします

## CLIセットアップ

```bash
openclaw onboard --auth-choice qianfan-api-key
```

## 関連ドキュメント

-   [OpenClaw設定](../gateway/configuration.md)
-   [モデルプロバイダー](../concepts/model-providers.md)
-   [エージェントセットアップ](../concepts/agent.md)
-   [Qianfan APIドキュメント](https://cloud.baidu.com/doc/qianfan-api/s/3m7of64lb)

[OpenRouter](./openrouter.md)[Qwen](./qwen.md)

---