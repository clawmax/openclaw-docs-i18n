title: "Z.AIプロバイダー経由でOpenClawでGLMモデルを使用する方法"
description: "Z.AIプロバイダーを使用してOpenClawでglm-5などのGLMモデルをセットアップおよび設定する方法を学びます。CLIセットアップと設定例を含みます。"
keywords: ["glmモデル", "openclaw zaiプロバイダー", "glm-5セットアップ", "zai apiキー", "openclaw設定", "glmモデルファミリー", "zaiプラットフォーム", "openclawプロバイダー"]
---

  プロバイダー

  
# GLMモデル

GLMは、Z.AIプラットフォームを通じて利用可能な**モデルファミリー**（企業ではありません）です。OpenClawでは、GLMモデルは`zai`プロバイダーと`zai/glm-5`のようなモデルIDを介してアクセスされます。

## CLIセットアップ

```bash
openclaw onboard --auth-choice zai-api-key
```

## 設定スニペット

```json
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-5" } } },
}
```

## 注意事項

-   GLMのバージョンと利用可能性は変更される可能性があります。最新情報はZ.AIのドキュメントを確認してください。
-   モデルIDの例には`glm-5`、`glm-4.7`、`glm-4.6`などがあります。
-   プロバイダーの詳細については、[/providers/zai](./zai.md)を参照してください。

[Litellm](./litellm.md)[MiniMax](./minimax.md)

---