

  نظرة عامة

  
# موفرو النماذج

يمكن لـ OpenClaw استخدام العديد من موفري LLM. اختر مزودًا، وقم بالمصادقة، ثم عيّن النموذج الافتراضي كـ `provider/model`. هل تبحث عن وثائق قنوات الدردشة (WhatsApp/Telegram/Discord/Slack/Mattermost (plugin)/إلخ.)؟ راجع [القنوات](./channels.md).

## البدء السريع

1.  قم بالمصادقة مع المزود (عادةً عبر `openclaw onboard`).
2.  عيّن النموذج الافتراضي:

```json
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## وثائق المزودين

-   [Amazon Bedrock](./providers/bedrock.md)
-   [Anthropic (API + Claude Code CLI)](./providers/anthropic.md)
-   [Cloudflare AI Gateway](./providers/cloudflare-ai-gateway.md)
-   [نماذج GLM](./providers/glm.md)
-   [Hugging Face (Inference)](./providers/huggingface.md)
-   [Kilocode](./providers/kilocode.md)
-   [LiteLLM (بوابة موحدة)](./providers/litellm.md)
-   [MiniMax](./providers/minimax.md)
-   [Mistral](./providers/mistral.md)
-   [Moonshot AI (Kimi + Kimi Coding)](./providers/moonshot.md)
-   [NVIDIA](./providers/nvidia.md)
-   [Ollama (نماذج محلية)](./providers/ollama.md)
-   [OpenAI (API + Codex)](./providers/openai.md)
-   [OpenCode Zen](./providers/opencode.md)
-   [OpenRouter](./providers/openrouter.md)
-   [Qianfan](./providers/qianfan.md)
-   [Qwen (OAuth)](./providers/qwen.md)
-   [Together AI](./providers/together.md)
-   [Vercel AI Gateway](./providers/vercel-ai-gateway.md)
-   [Venice (Venice AI، يركز على الخصوصية)](./providers/venice.md)
-   [vLLM (نماذج محلية)](./providers/vllm.md)
-   [Xiaomi](./providers/xiaomi.md)
-   [Z.AI](./providers/zai.md)

## موفرو النصوص (النسخ)

-   [Deepgram (نسخ الصوت)](./providers/deepgram.md)

## أدوات المجتمع

-   [Claude Max API Proxy](./providers/claude-max-api-proxy.md) - وكيل مجتمعي لبيانات اعتماد اشتراك Claude (تحقق من سياسة/شروط Anthropic قبل الاستخدام)

للحصول على كتالوج المزودين الكامل (xAI، Groq، Mistral، إلخ.) والتكوين المتقدم، راجع [موفرو النماذج](./concepts/model-providers.md).

[بدء سريع مع موفري النماذج](./providers/models.md)

---