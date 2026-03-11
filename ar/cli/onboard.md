

  أوامر CLI

  
# onboard

معالج إعداد تفاعلي (إعداد بوابة محلية أو بعيدة).

## أدلة ذات صلة

-   مركز إعداد CLI: [معالج الإعداد (CLI)](../start/wizard.md)
-   نظرة عامة على الإعداد: [نظرة عامة على الإعداد](../start/onboarding-overview.md)
-   مرجع إعداد CLI: [مرجع إعداد CLI](../start/wizard-cli-reference.md)
-   أتمتة CLI: [أتمتة CLI](../start/wizard-cli-automation.md)
-   إعداد macOS: [الإعداد (تطبيق macOS)](../start/onboarding.md)

## أمثلة

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url wss://gateway-host:18789
```

لأهداف شبكة خاصة نص عادي `ws://` (شبكات موثوقة فقط)، عيّن `OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1` في بيئة عملية الإعداد. موفر مخصص غير تفاعلي:

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --secret-input-mode plaintext \
  --custom-compatibility openai
```

`--custom-api-key` اختياري في الوضع غير التفاعلي. إذا حُذف، يتحقق الإعداد من `CUSTOM_API_KEY`. تخزين مفاتيح المزود كمراجع بدلاً من النص العادي:

```bash
openclaw onboard --non-interactive \
  --auth-choice openai-api-key \
  --secret-input-mode ref \
  --accept-risk
```

مع `--secret-input-mode ref`، يكتب الإعداد مراجع مدعومة بالبيئة بدلاً من قيم المفاتيح كنص عادي. لمزودي المصادقة المدعومين بالملف الشخصي، يكتب هذا إدخالات `keyRef`؛ للمزودين المخصصين يكتب `models.providers..apiKey` كمرجع بيئة (على سبيل المثال `{ source: "env", provider: "default", id: "CUSTOM_API_KEY" }`). عقد الوضع غير التفاعلي `ref`:

-   عيّن متغير البيئة للمزود في بيئة عملية الإعداد (على سبيل المثال `OPENAI_API_KEY`).
-   لا تمرر أعلام المفاتيح المضمنة (على سبيل المثال `--openai-api-key`) إلا إذا تم تعيين متغير البيئة هذا أيضًا.
-   إذا تم تمرير علم مفتاح مضمن بدون متغير البيئة المطلوب، يفشل الإعداد بسرعة مع توجيه.

خيارات رمز البوابة في الوضع غير التفاعلي:

-   `--gateway-auth token --gateway-token ` يخزن رمزًا كنص عادي.
-   `--gateway-auth token --gateway-token-ref-env ` يخزن `gateway.auth.token` كـ SecretRef بيئة.
-   `--gateway-token` و `--gateway-token-ref-env` متنافيان.
-   `--gateway-token-ref-env` يتطلب متغير بيئة غير فارغ في بيئة عملية الإعداد.
-   مع `--install-daemon`، عندما تتطلب مصادقة الرمز رمزًا، يتم التحقق من رموز البوابة المدارة بـ SecretRef ولكن لا يتم استمرارها كنص عادي محلول في بيانات وصفية لبيئة خدمة المشرف.
-   مع `--install-daemon`، إذا تطلب وضع الرمز رمزًا وكان مرجع السر المُهيأ غير محلول، يفشل الإعداد مغلقًا مع توجيهات العلاج.
-   مع `--install-daemon`، إذا تم تكوين كل من `gateway.auth.token` و `gateway.auth.password` و `gateway.auth.mode` غير معيّن، يمنع الإعداد التثبيت حتى يتم تعيين الوضع صراحة.

مثال:

```bash
export OPENCLAW_GATEWAY_TOKEN="your-token"
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice skip \
  --gateway-auth token \
  --gateway-token-ref-env OPENCLAW_GATEWAY_TOKEN \
  --accept-risk
```

سلوك الإعداد التفاعلي مع وضع المرجع:

-   اختر **استخدام مرجع سر** عند المطالبة.
-   ثم اختر إما:
    -   متغير بيئة
    -   مزود سر مُهيأ (`file` أو `exec`)
-   يقوم الإعداد بإجراء تحقق مسبق سريع قبل حفظ المرجع.
    -   إذا فشل التحقق، يعرض الإعداد الخطأ ويسمح لك بإعادة المحاولة.

خيارات نقطة نهاية Z.AI غير التفاعلية: ملاحظة: `--auth-choice zai-api-key` تكتشف الآن تلقائيًا أفضل نقطة نهاية Z.AI لمفتاحك (تفضل API العام مع `zai/glm-5`). إذا كنت تريد على وجه التحديد نقاط نهاية خطة GLM Coding، اختر `zai-coding-global` أو `zai-coding-cn`.

```bash
# اختيار نقطة نهاية بدون مطالبة
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# خيارات نقاط نهاية Z.AI الأخرى:
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

مثال Mistral غير تفاعلي:

```bash
openclaw onboard --non-interactive \
  --auth-choice mistral-api-key \
  --mistral-api-key "$MISTRAL_API_KEY"
```

ملاحظات التدفق:

-   `quickstart`: مطالبات دنيا، يولد تلقائيًا رمز بوابة.
-   `manual`: مطالبات كاملة للبورت/الربط/المصادقة (اسم مستعار لـ `advanced`).
-   سلوك نطاق DM للإعداد المحلي: [مرجع إعداد CLI](../start/wizard-cli-reference.md#outputs-and-internals).
-   أول محادثة أسرع: `openclaw dashboard` (واجهة التحكم، بدون إعداد قناة).
-   مزود مخصص: ربط أي نقطة نهاية متوافقة مع OpenAI أو Anthropic، بما في ذلك المزودين المستضافين غير المدرجين. استخدم Unknown للكشف التلقائي.

## أوامر متابعة شائعة

```bash
openclaw configure
openclaw agents add <name>
```

> **ℹ️** `--json` لا تعني الوضع غير التفاعلي. استخدم `--non-interactive` للنصوص البرمجية.

[nodes](./nodes.md)[pairing](./pairing.md)