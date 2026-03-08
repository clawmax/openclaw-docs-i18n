title: "إعداد وتكوين موفر Cloudflare AI Gateway في OpenClaw"
description: "تعلّم كيفية تكوين OpenClaw لاستخدام Cloudflare AI Gateway لنماذج Anthropic. قم بإعداد مفاتيح API والمصادقة والنماذج الافتراضية للتحليلات والتخزين المؤقت."
keywords: ["cloudflare ai gateway", "موفر openclaw", "anthropic api", "claude sonnet", "إعداد بوابة api", "مصادقة بوابة الذكاء الاصطناعي", "تكوين openclaw", "تكامل بوابة cloudflare"]
---

  الموفرون

  
# Cloudflare AI Gateway

تقع بوابة Cloudflare AI Gateway أمام واجهات برمجة التطبيقات (APIs) للموفرين وتتيح لك إضافة تحليلات وتخزين مؤقت وضوابط. بالنسبة لـ Anthropic، يستخدم OpenClaw واجهة Anthropic Messages API عبر نقطة نهاية البوابة الخاصة بك.

-   الموفر: `cloudflare-ai-gateway`
-   عنوان URL الأساسي: `https://gateway.ai.cloudflare.com/v1/<account_id>/<gateway_id>/anthropic`
-   النموذج الافتراضي: `cloudflare-ai-gateway/claude-sonnet-4-5`
-   مفتاح API: `CLOUDFLARE_AI_GATEWAY_API_KEY` (مفتاح API الخاص بالموفر للطلبات عبر البوابة)

بالنسبة لنماذج Anthropic، استخدم مفتاح Anthropic API الخاص بك.

## البدء السريع

1.  قم بتعيين مفتاح API الخاص بالموفر وتفاصيل البوابة:

```bash
openclaw onboard --auth-choice cloudflare-ai-gateway-api-key
```

2.  قم بتعيين نموذج افتراضي:

```json
{
  agents: {
    defaults: {
      model: { primary: "cloudflare-ai-gateway/claude-sonnet-4-5" },
    },
  },
}
```

## مثال غير تفاعلي

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice cloudflare-ai-gateway-api-key \
  --cloudflare-ai-gateway-account-id "your-account-id" \
  --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
  --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY"
```

## البوابات المصادق عليها

إذا قمت بتمكين مصادقة البوابة في Cloudflare، أضف رأس `cf-aig-authorization` (هذا بالإضافة إلى مفتاح API الخاص بالموفر).

```json
{
  models: {
    providers: {
      "cloudflare-ai-gateway": {
        headers: {
          "cf-aig-authorization": "Bearer <cloudflare-ai-gateway-token>",
        },
      },
    },
  },
}
```

## ملاحظة حول البيئة

إذا كانت البوابة تعمل كخدمة خلفية (daemon) (مثل launchd/systemd)، تأكد من أن `CLOUDFLARE_AI_GATEWAY_API_KEY` متاح لتلك العملية (على سبيل المثال، في `~/.openclaw/.env` أو عبر `env.shellEnv`).

[Amazon Bedrock](./bedrock.md)[Claude Max API Proxy](./claude-max-api-proxy.md)

---