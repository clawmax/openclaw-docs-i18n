title: "دليل أتمتة واجهة سطر أوامر OpenClaw للإعداد التلقائي غير التفاعلي"
description: "تعلّم كيفية أتمتة معالج إعداد OpenClaw باستخدام علم `--non-interactive`. احصل على أمثلة لأنظمة Anthropic و OpenAI و Gemini ومزودين مخصصين."
keywords: ["واجهة سطر أوامر openclaw", "الإعداد غير التفاعلي", "أتمتة openclaw", "أتمتة واجهة سطر الأوامر", "أمر onboard", "إعداد مفتاح API", "أتمتة الوكيل", "واجهة سطر الأوامر"]
---

  الدلائل

  
# أتمتة واجهة سطر الأوامر

استخدم `--non-interactive` لأتمتة أمر `openclaw onboard`.

> **ℹ️** `--json` لا تعني الوضع غير التفاعلي. استخدم `--non-interactive` (و `--workspace`) للنصوص البرمجية.

## مثال أساسي غير تفاعلي

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --secret-input-mode plaintext \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

أضف `--json` للحصول على ملخص قابل للقراءة آليًا. استخدم `--secret-input-mode ref` لتخزين مراجع مدعومة بمتغيرات بيئة في ملفات تعريف المصادقة بدلاً من القيم النصية العادية. يتوفر اختيار تفاعلي بين مراجع المتغيرات البيئية ومراجع المزودين المُهيئين (`file` أو `exec`) في تدفق معالج الإعداد. في الوضع غير التفاعلي `ref`، يجب تعيين متغيرات بيئة المزود في بيئة العملية. تمرير أعلام المفاتيح المضمنة دون متغير البيئة المطابق سيفشل الآن بسرعة. مثال:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice openai-api-key \
  --secret-input-mode ref \
  --accept-risk
```

## أمثلة خاصة بالمزود

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice gemini-api-key \
  --gemini-api-key "$GEMINI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice zai-api-key \
  --zai-api-key "$ZAI_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice cloudflare-ai-gateway-api-key \
  --cloudflare-ai-gateway-account-id "your-account-id" \
  --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
  --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice moonshot-api-key \
  --moonshot-api-key "$MOONSHOT_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice mistral-api-key \
  --mistral-api-key "$MISTRAL_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice synthetic-api-key \
  --synthetic-api-key "$SYNTHETIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice opencode-zen \
  --opencode-zen-api-key "$OPENCODE_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback
```

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --custom-provider-id "my-custom" \
  --custom-compatibility anthropic \
  --gateway-port 18789 \
  --gateway-bind loopback
```

`--custom-api-key` اختياري. إذا حُذف، يتحقق الإعداد من `CUSTOM_API_KEY`.نموذج وضع المرجع (`ref`):

```bash
export CUSTOM_API_KEY="your-key"
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --secret-input-mode ref \
  --custom-provider-id "my-custom" \
  --custom-compatibility anthropic \
  --gateway-port 18789 \
  --gateway-bind loopback
```

في هذا الوضع، يخزن الإعداد `apiKey` كـ `{ source: "env", provider: "default", id: "CUSTOM_API_KEY" }`.

## إضافة وكيل آخر

استخدم `openclaw agents add ` لإنشاء وكيل منفصل بمساحة عمله وجلساته وملفات تعريف المصادقة الخاصة به. التشغيل دون `--workspace` سيطلق المعالج.

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

ما يضبطه:

-   `agents.list[].name`
-   `agents.list[].workspace`
-   `agents.list[].agentDir`

ملاحظات:

-   تتبع مساحات العمل الافتراضية النمط `~/.openclaw/workspace-`.
-   أضف `bindings` لتوجيه الرسائل الواردة (يمكن للمعالج القيام بذلك).
-   أعلام غير تفاعلية: `--model`, `--agent-dir`, `--bind`, `--non-interactive`.

## وثائق ذات صلة

-   مركز الإعداد: [معالج الإعداد (واجهة سطر الأوامر)](./wizard.md)
-   المرجع الكامل: [مرجع إعداد واجهة سطر الأوامر](./wizard-cli-reference.md)
-   مرجع الأوامر: [أمر `openclaw onboard`](../cli/onboard.md)

[مرجع واجهة سطر الأوامر](./wizard-cli-reference.md)

---