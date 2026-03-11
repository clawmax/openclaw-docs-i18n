

  الموفرون

  
# Hugging Face (الاستدلال)

تقدم [موفرو استدلال Hugging Face](https://huggingface.co/docs/inference-providers) إكمالات دردشة متوافقة مع OpenAI من خلال واجهة برمجة تطبيقات موجه واحدة. تحصل على الوصول إلى العديد من النماذج (DeepSeek وLlama والمزيد) باستخدام رمز واحد. يستخدم OpenClaw **نقطة النهاية المتوافقة مع OpenAI** (إكمالات الدردشة فقط)؛ للتحويل من نص إلى صورة، أو التضمينات، أو الكلام استخدم [عملاء استدلال HF](https://huggingface.co/docs/api-inference/quicktour) مباشرة.

-   الموفر: `huggingface`
-   المصادقة: `HUGGINGFACE_HUB_TOKEN` أو `HF_TOKEN` (رمز دقيق مع إذن **Make calls to Inference Providers**)
-   واجهة برمجة التطبيقات: متوافقة مع OpenAI (`https://router.huggingface.co/v1`)
-   الفوترة: رمز HF واحد؛ تتبع [التسعير](https://huggingface.co/docs/inference-providers/pricing) أسعار المزود مع طبقة مجانية.

## البدء السريع

1.  أنشئ رمزًا دقيقًا في [Hugging Face → الإعدادات → الرموز](https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained) مع إذن **Make calls to Inference Providers**.
2.  قم بتشغيل عملية الإعداد واختر **Hugging Face** في القائمة المنسدلة للمزود، ثم أدخل مفتاح API الخاص بك عند المطالبة:

```bash
openclaw onboard --auth-choice huggingface-api-key
```

3.  في القائمة المنسدلة **النموذج الافتراضي لـ Hugging Face**، اختر النموذج الذي تريده (يتم تحميل القائمة من واجهة برمجة تطبيقات الاستدلال عندما يكون لديك رمز صالح؛ وإلا يتم عرض قائمة مدمجة). يتم حفظ اختيارك كنموذج افتراضي.
4.  يمكنك أيضًا تعيين النموذج الافتراضي أو تغييره لاحقًا في التكوين:

```json
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1" },
    },
  },
}
```

## مثال غير تفاعلي

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice huggingface-api-key \
  --huggingface-api-key "$HF_TOKEN"
```

سيؤدي هذا إلى تعيين `huggingface/deepseek-ai/DeepSeek-R1` كنموذج افتراضي.

## ملاحظة حول البيئة

إذا كان البوابة تعمل كخدمة خلفية (launchd/systemd)، تأكد من أن `HUGGINGFACE_HUB_TOKEN` أو `HF_TOKEN` متاح لتلك العملية (على سبيل المثال، في `~/.openclaw/.env` أو عبر `env.shellEnv`).

## اكتشاف النماذج والقائمة المنسدلة أثناء الإعداد

يكتشف OpenClaw النماذج عن طريق الاتصال **بـ نقطة نهاية الاستدلال مباشرة**:

```bash
GET https://router.huggingface.co/v1/models
```

(اختياري: أرسل `Authorization: Bearer $HUGGINGFACE_HUB_TOKEN` أو `$HF_TOKEN` للحصول على القائمة الكاملة؛ قد تعيد بعض نقاط النهاية مجموعة فرعية بدون مصادقة.) الاستجابة تكون على نمط OpenAI `{ "object": "list", "data": [ { "id": "Qwen/Qwen3-8B", "owned_by": "Qwen", ... }, ... ] }`. عند تكوين مفتاح Hugging Face API (عبر عملية الإعداد، أو `HUGGINGFACE_HUB_TOKEN`، أو `HF_TOKEN`)، يستخدم OpenClaw طلب GET هذا لاكتشاف نماذج إكمال الدردشة المتاحة. أثناء **عملية الإعداد التفاعلية**، بعد إدخال الرمز الخاص بك، سترى قائمة منسدلة **النموذج الافتراضي لـ Hugging Face** مملوءة من تلك القائمة (أو الكتالوج المدمج إذا فشل الطلب). أثناء وقت التشغيل (مثل بدء تشغيل البوابة)، عندما يكون المفتاح موجودًا، يستدعي OpenClaw مرة أخرى **GET** `https://router.huggingface.co/v1/models` لتحديث الكتالوج. يتم دمج القائمة مع كتالوج مدمج (للبيانات الوصفية مثل نافذة السياق والتكلفة). إذا فشل الطلب أو لم يتم تعيين مفتاح، يتم استخدام الكتالوج المدمج فقط.

## أسماء النماذج والخيارات القابلة للتعديل

-   **الاسم من واجهة برمجة التطبيقات:** يتم **تعبئة اسم عرض النموذج من GET /v1/models** عندما تعيد واجهة برمجة التطبيقات `name` أو `title` أو `display_name`؛ وإلا يتم اشتقاقه من معرف النموذج (مثال: `deepseek-ai/DeepSeek-R1` → "DeepSeek R1").
-   **تجاوز اسم العرض:** يمكنك تعيين تسمية مخصصة لكل نموذج في التكوين بحيث تظهر بالطريقة التي تريدها في واجهة سطر الأوامر وواجهة المستخدم:

```json
{
  agents: {
    defaults: {
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1 (fast)" },
        "huggingface/deepseek-ai/DeepSeek-R1:cheapest": { alias: "DeepSeek R1 (cheap)" },
      },
    },
  },
}
```

-   **اختيار المزود / السياسة:** ألحق لاحقة بـ **معرف النموذج** لاختيار كيفية قيام الموجه باختيار الخلفية:
    
    -   **`:fastest`** — أعلى إنتاجية (الموجه يختار؛ اختيار المزود **مقفل** — لا تظهر أداة اختيار خلفية تفاعلية).
    -   **`:cheapest`** — أقل تكلفة لكل رمز مخرج (الموجه يختار؛ اختيار المزود **مقفل**).
    -   **`:provider`** — فرض خلفية محددة (مثال: `:sambanova`، `:together`).
    
    عندما تختار **:cheapest** أو **:fastest** (مثال: في القائمة المنسدلة لنماذج الإعداد)، يتم قفل المزود: يقرر الموجه بناءً على التكلفة أو السرعة ولا تظهر خطوة اختيارية "تفضيل خلفية محددة". يمكنك إضافة هذه كإدخالات منفصلة في `models.providers.huggingface.models` أو تعيين `model.primary` مع اللاحقة. يمكنك أيضًا تعيين ترتيبك الافتراضي في [إعدادات موفر الاستدلال](https://hf.co/settings/inference-providers) (بدون لاحقة = استخدم ذلك الترتيب).
-   **دمج التكوين:** يتم الاحتفاظ بالإدخالات الموجودة في `models.providers.huggingface.models` (مثال: في `models.json`) عند دمج التكوين. لذلك أي `name` مخصص، أو `alias`، أو خيارات نموذج قمت بتعيينها هناك يتم الاحتفاظ بها.

## معرفات النماذج وأمثلة التكوين

تستخدم مراجع النماذج الصيغة `huggingface//` (معرفات على نمط Hub). القائمة أدناه من **GET** `https://router.huggingface.co/v1/models`؛ قد يتضمن كتالوجك المزيد. **أمثلة على المعرفات (من نقطة نهاية الاستدلال):**

| النموذج | المرجع (أضف البادئة `huggingface/`) |
| --- | --- |
| DeepSeek R1 | `deepseek-ai/DeepSeek-R1` |
| DeepSeek V3.2 | `deepseek-ai/DeepSeek-V3.2` |
| Qwen3 8B | `Qwen/Qwen3-8B` |
| Qwen2.5 7B Instruct | `Qwen/Qwen2.5-7B-Instruct` |
| Qwen3 32B | `Qwen/Qwen3-32B` |
| Llama 3.3 70B Instruct | `meta-llama/Llama-3.3-70B-Instruct` |
| Llama 3.1 8B Instruct | `meta-llama/Llama-3.1-8B-Instruct` |
| GPT-OSS 120B | `openai/gpt-oss-120b` |
| GLM 4.7 | `zai-org/GLM-4.7` |
| Kimi K2.5 | `moonshotai/Kimi-K2.5` |

يمكنك إلحاق `:fastest` أو `:cheapest` أو `:provider` (مثال: `:together`، `:sambanova`) بمعرف النموذج. قم بتعيين ترتيبك الافتراضي في [إعدادات موفر الاستدلال](https://hf.co/settings/inference-providers)؛ راجع [موفرو الاستدلال](https://huggingface.co/docs/inference-providers) و **GET** `https://router.huggingface.co/v1/models` للحصول على القائمة الكاملة.

### أمثلة تكوين كاملة

**DeepSeek R1 أساسي مع Qwen احتياطي:**

```json
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-R1",
        fallbacks: ["huggingface/Qwen/Qwen3-8B"],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1" },
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
      },
    },
  },
}
```

**Qwen كافتراضي، مع متغيرات :cheapest و :fastest:**

```json
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen3-8B" },
      models: {
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
        "huggingface/Qwen/Qwen3-8B:cheapest": { alias: "Qwen3 8B (cheapest)" },
        "huggingface/Qwen/Qwen3-8B:fastest": { alias: "Qwen3 8B (fastest)" },
      },
    },
  },
}
```

**DeepSeek + Llama + GPT-OSS مع أسماء مستعارة:**

```json
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-V3.2",
        fallbacks: [
          "huggingface/meta-llama/Llama-3.3-70B-Instruct",
          "huggingface/openai/gpt-oss-120b",
        ],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-V3.2": { alias: "DeepSeek V3.2" },
        "huggingface/meta-llama/Llama-3.3-70B-Instruct": { alias: "Llama 3.3 70B" },
        "huggingface/openai/gpt-oss-120b": { alias: "GPT-OSS 120B" },
      },
    },
  },
}
```

**فرض خلفية محددة باستخدام :provider:**

```json
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1:together" },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1:together": { alias: "DeepSeek R1 (Together)" },
      },
    },
  },
}
```

**نماذج متعددة من Qwen وDeepSeek مع لواحق السياسة:**

```json
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest" },
      models: {
        "huggingface/Qwen/Qwen2.5-7B-Instruct": { alias: "Qwen2.5 7B" },
        "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest": { alias: "Qwen2.5 7B (cheap)" },
        "huggingface/deepseek-ai/DeepSeek-R1:fastest": { alias: "DeepSeek R1 (fast)" },
        "huggingface/meta-llama/Llama-3.1-8B-Instruct": { alias: "Llama 3.1 8B" },
      },
    },
  },
}
```

[GitHub Copilot](./github-copilot.md)[Kilocode](./kilocode.md)