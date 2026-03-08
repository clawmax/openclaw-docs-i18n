title: "دليل إعداد وتكوين موفري النماذج في OpenClaw"
description: "تعلم كيفية المصادقة وإعداد موفري نماذج اللغة الكبيرة مثل OpenAI و Anthropic و Mistral في OpenClaw. دليل البدء السريع والقائمة الكاملة للموفرين المدعومين."
keywords: ["نماذج openclaw", "موفرو llm", "تكوين النموذج", "anthropic claude", "openai api", "إعداد النموذج", "openrouter", "bedrock"]
---

  نظرة عامة

  
# البدء السريع مع موفر النموذج

يمكن لـ OpenClaw استخدام العديد من موفري نماذج اللغة الكبيرة. اختر واحدًا، قم بالمصادقة، ثم عيّن النموذج الافتراضي كـ `provider/model`.

## البدء السريع (خطوتان)

1.  قم بالمصادقة مع المزود (عادةً عبر `openclaw onboard`).
2.  عيّن النموذج الافتراضي:

```json
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## الموفرون المدعومون (مجموعة البداية)

-   [OpenAI (API + Codex)](./openai.md)
-   [Anthropic (API + Claude Code CLI)](./anthropic.md)
-   [OpenRouter](./openrouter.md)
-   [Vercel AI Gateway](./vercel-ai-gateway.md)
-   [Cloudflare AI Gateway](./cloudflare-ai-gateway.md)
-   [Moonshot AI (Kimi + Kimi Coding)](./moonshot.md)
-   [Mistral](./mistral.md)
-   [Synthetic](./synthetic.md)
-   [OpenCode Zen](./opencode.md)
-   [Z.AI](./zai.md)
-   [GLM models](./glm.md)
-   [MiniMax](./minimax.md)
-   [Venice (Venice AI)](./venice.md)
-   [Amazon Bedrock](./bedrock.md)
-   [Qianfan](./qianfan.md)

للحصول على كتالوج المزودين الكامل (xAI, Groq, Mistral, إلخ) والتكوين المتقدم، راجع [موفرو النماذج](../concepts/model-providers.md).

[موفرو النماذج](../providers.md)[واجهة سطر أوامر النماذج](../concepts/models.md)

---