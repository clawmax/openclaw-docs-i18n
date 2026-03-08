

  提供商

  
# 千帆

千帆是百度的模型即服务平台，提供了一个**统一 API**，可将请求通过单一端点和 API 密钥路由到众多模型。它与 OpenAI 兼容，因此大多数 OpenAI SDK 只需切换基础 URL 即可使用。

## 先决条件

1.  一个具有千帆 API 访问权限的百度云账户
2.  来自千帆控制台的 API 密钥
3.  系统上已安装 OpenClaw

## 获取您的 API 密钥

1.  访问 [千帆控制台](https://console.bce.baidu.com/qianfan/ais/console/apiKey)
2.  创建新应用或选择现有应用
3.  生成 API 密钥（格式：`bce-v3/ALTAK-...`）
4.  复制 API 密钥以供 OpenClaw 使用

## CLI 设置

```bash
openclaw onboard --auth-choice qianfan-api-key
```

## 相关文档

-   [OpenClaw 配置](../gateway/configuration.md)
-   [模型提供商](../concepts/model-providers.md)
-   [智能体设置](../concepts/agent.md)
-   [千帆 API 文档](https://cloud.baidu.com/doc/qianfan-api/s/3m7of64lb)

[OpenRouter](./openrouter.md)[Qwen](./qwen.md)

---