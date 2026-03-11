

  الموفرون

  
# بوابة Vercel الذكاء الاصطناعي

توفر [بوابة Vercel الذكاء الاصطناعي](https://vercel.com/ai-gateway) واجهة برمجة تطبيقات موحدة للوصول إلى مئات النماذج من خلال نقطة نهاية واحدة.

-   الموفر: `vercel-ai-gateway`
-   المصادقة: `AI_GATEWAY_API_KEY`
-   واجهة برمجة التطبيقات: متوافقة مع Anthropic Messages

## البدء السريع

1.  قم بتعيين مفتاح API (مستحسن: تخزينه للبوابة):

```bash
openclaw onboard --auth-choice ai-gateway-api-key
```

2.  قم بتعيين نموذج افتراضي:

```json
{
  agents: {
    defaults: {
      model: { primary: "vercel-ai-gateway/anthropic/claude-opus-4.6" },
    },
  },
}
```

## مثال غير تفاعلي

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY"
```

## ملاحظة حول البيئة

إذا كانت البوابة تعمل كخدمة خلفية (launchd/systemd)، تأكد من أن `AI_GATEWAY_API_KEY` متاح لتلك العملية (على سبيل المثال، في `~/.openclaw/.env` أو عبر `env.shellEnv`).

## اختصار معرف النموذج

يقبل OpenClaw المراجع المختصرة لنماذج Vercel Claude ويعيد تطبيعها أثناء وقت التشغيل:

-   `vercel-ai-gateway/claude-opus-4.6` -> `vercel-ai-gateway/anthropic/claude-opus-4.6`
-   `vercel-ai-gateway/opus-4.6` -> `vercel-ai-gateway/anthropic/claude-opus-4-6`

[Together](./together.md)[Venice AI](./venice.md)

---