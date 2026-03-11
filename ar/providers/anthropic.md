

  الموفرون

  
# Anthropic

تقوم Anthropic ببناء عائلة النماذج **Claude** وتوفر الوصول إليها عبر واجهة برمجة التطبيقات (API). في OpenClaw يمكنك المصادقة باستخدام مفتاح API أو **رمز إعداد (setup-token)**.

## الخيار أ: مفتاح واجهة برمجة تطبيقات Anthropic

**الأفضل لـ:** الوصول القياسي لواجهة برمجة التطبيقات والفوترة القائمة على الاستخدام. أنشئ مفتاح API الخاص بك في Anthropic Console.

### الإعداد عبر سطر الأوامر (CLI)

```bash
openclaw onboard
# اختر: Anthropic API key

# أو بشكل غير تفاعلي
openclaw onboard --anthropic-api-key "$ANTHROPIC_API_KEY"
```

### مقتطف التكوين

```json
{
  env: { ANTHROPIC_API_KEY: "sk-ant-..." },
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## الإعدادات الافتراضية للتفكير (Claude 4.6)

-   نماذج Anthropic Claude 4.6 تستخدم بشكل افتراضي `adaptive` thinking في OpenClaw عندما لا يتم تعيين مستوى تفكير صريح.
-   يمكنك التجاوز لكل رسالة (`/think:`) أو في معلمات النموذج: `agents.defaults.models["anthropic/"].params.thinking`.
-   وثائق Anthropic ذات الصلة:
    -   [التفكير التكيفي](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking)
    -   [التفكير الممتد](https://platform.claude.com/docs/en/build-with-claude/extended-thinking)

## تخزين المطالبات المؤقت (واجهة برمجة تطبيقات Anthropic)

يدعم OpenClaw ميزة تخزين المطالبات المؤقت من Anthropic. هذه الميزة **لواجهة برمجة التطبيقات فقط**؛ المصادقة بالاشتراك لا تحترم إعدادات التخزين المؤقت.

### التكوين

استخدم المعلمة `cacheRetention` في تكوين النموذج الخاص بك:

| القيمة | مدة التخزين المؤقت | الوصف |
| --- | --- | --- |
| `none` | لا يوجد تخزين مؤقت | تعطيل تخزين المطالبات المؤقت |
| `short` | 5 دقائق | الإفتراضي لمصادقة مفتاح API |
| `long` | ساعة واحدة | تخزين مؤقت ممتد (يتطلب علامة بيتا) |

```json
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": {
          params: { cacheRetention: "long" },
        },
      },
    },
  },
}
```

### الإعدادات الافتراضية

عند استخدام مصادقة مفتاح واجهة برمجة تطبيقات Anthropic، يطبق OpenClaw تلقائيًا `cacheRetention: "short"` (تخزين مؤقت لمدة 5 دقائق) لجميع نماذج Anthropic. يمكنك تجاوز هذا عن طريق تعيين `cacheRetention` صراحةً في التكوين الخاص بك.

### تجاوزات cacheRetention لكل وكيل

استخدم معلمات مستوى النموذج كخط أساس، ثم تجاوز وكلاء محددين عبر `agents.list[].params`.

```json
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-opus-4-6" },
      models: {
        "anthropic/claude-opus-4-6": {
          params: { cacheRetention: "long" }, // خط الأساس لمعظم الوكلاء
        },
      },
    },
    list: [
      { id: "research", default: true },
      { id: "alerts", params: { cacheRetention: "none" } }, // تجاوز لهذا الوكيل فقط
    ],
  },
}
```

ترتيب دمج التكوين للمعلمات المتعلقة بالتخزين المؤقت:

1.  `agents.defaults.models["provider/model"].params`
2.  `agents.list[].params` (يطابق `id`، يتجاوز حسب المفتاح)

هذا يسمح لوكيل واحد بالاحتفاظ بتخزين مؤقت طويل الأمد بينما يقوم وكيل آخر على نفس النموذج بتعطيل التخزين المؤقت لتجنب تكاليف الكتابة على حركة المرور المتقطعة/منخفضة إعادة الاستخدام.

### ملاحظات حول Bedrock Claude

-   نماذج Anthropic Claude على Bedrock (`amazon-bedrock/*anthropic.claude*`) تقبل تمرير `cacheRetention` عند تكوينها.
-   نماذج Bedrock غير التابعة لـ Anthropic يتم إجبارها على `cacheRetention: "none"` أثناء وقت التشغيل.
-   الإعدادات الافتراضية الذكية لمفتاح واجهة برمجة تطبيقات Anthropic تضع أيضًا `cacheRetention: "short"` لمراجع نموذج Claude-on-Bedrock عندما لا يتم تعيين قيمة صريحة.

### المعلمة القديمة

المعلمة القديمة `cacheControlTtl` لا تزال مدعومة للتوافق مع الإصدارات السابقة:

-   `"5m"` تُرسم إلى `short`
-   `"1h"` تُرسم إلى `long`

نوصي بالانتقال إلى المعلمة الجديدة `cacheRetention`. يتضمن OpenClaw علامة البيتا `extended-cache-ttl-2025-04-11` لطلبات واجهة برمجة تطبيقات Anthropic؛ احتفظ بها إذا قمت بتجاوز رؤوس الموفر (انظر [/gateway/configuration](../gateway/configuration.md)).

## نافذة سياق 1 مليون رمز (بيتا Anthropic)

نافذة سياق 1 مليون رمز من Anthropic مقيدة بعلامة بيتا. في OpenClaw، قم بتمكينها لكل نموذج باستخدام `params.context1m: true` للنماذج المدعومة Opus/Sonnet.

```json
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": {
          params: { context1m: true },
        },
      },
    },
  },
}
```

يقوم OpenClaw بتعيين هذا إلى `anthropic-beta: context-1m-2025-08-07` على طلبات Anthropic. يتم تنشيط هذا فقط عندما يتم تعيين `params.context1m` صراحةً إلى `true` لهذا النموذج. المتطلب: يجب أن تسمح Anthropic باستخدام السياق الطويل على تلك البيانات الاعتمادية (عادةً فوترة مفتاح API، أو حساب اشتراك مع تمكين الاستخدام الإضافي). وإلا سترجع Anthropic: `HTTP 429: rate_limit_error: Extra usage is required for long context requests`. ملاحظة: ترفض Anthropic حاليًا طلبات بيتا `context-1m-*` عند استخدام رموز OAuth/الاشتراك (`sk-ant-oat-*`). يتخطى OpenClaw تلقائيًا رأس بيتا context1m لمصادقة OAuth ويحتفظ بعلامات البيتا المطلوبة لـ OAuth.

## الخيار ب: رمز إعداد Claude (setup-token)

**الأفضل لـ:** استخدام اشتراك Claude الخاص بك.

### أين تحصل على رمز الإعداد (setup-token)

يتم إنشاء رموز الإعداد بواسطة **Claude Code CLI**، وليس Anthropic Console. يمكنك تشغيل هذا على **أي جهاز**:

```bash
claude setup-token
```

الصق الرمز في OpenClaw (المعالج: **Anthropic token (paste setup-token)**)، أو قم بتشغيله على مضيف البوابة (gateway):

```bash
openclaw models auth setup-token --provider anthropic
```

إذا قمت بإنشاء الرمز على جهاز مختلف، الصقه:

```bash
openclaw models auth paste-token --provider anthropic
```

### الإعداد عبر سطر الأوامر (رمز الإعداد)

```bash
# الصق رمز إعداد (setup-token) أثناء عملية الإعداد
openclaw onboard --auth-choice setup-token
```

### مقتطف التكوين (رمز الإعداد)

```json
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## ملاحظات

-   قم بإنشاء رمز الإعداد باستخدام `claude setup-token` والصقه، أو قم بتشغيل `openclaw models auth setup-token` على مضيف البوابة.
-   إذا رأيت "OAuth token refresh failed …" على اشتراك Claude، قم بإعادة المصادقة باستخدام رمز إعداد. انظر [/gateway/troubleshooting#oauth-token-refresh-failed-anthropic-claude-subscription](../gateway/troubleshooting.md#oauth-token-refresh-failed-anthropic-claude-subscription).
-   تفاصيل المصادقة + قواعد إعادة الاستخدام موجودة في [/concepts/oauth](../concepts/oauth.md).

## استكشاف الأخطاء وإصلاحها

**أخطاء 401 / الرمز أصبح غير صالح فجأة**

-   قد تنتهي صلاحية مصادقة اشتراك Claude أو يتم إلغاؤها. أعد تشغيل `claude setup-token` والصقه في **مضيف البوابة**.
-   إذا كان تسجيل دخول Claude CLI موجودًا على جهاز مختلف، استخدم `openclaw models auth paste-token --provider anthropic` على مضيف البوابة.

**لم يتم العثور على مفتاح API للموفر "anthropic"**

-   المصادقة **لكل وكيل**. الوكلاء الجدد لا يرثون مفاتيح الوكيل الرئيسي.
-   أعد تشغيل عملية الإعداد لذلك الوكيل، أو الصق رمز إعداد / مفتاح API على مضيف البوابة، ثم تحقق باستخدام `openclaw models status`.

**لم يتم العثور على بيانات اعتماد للملف الشخصي `anthropic:default`**

-   قم بتشغيل `openclaw models status` لمعرفة أي ملف شخصي للمصادقة نشط.
-   أعد تشغيل عملية الإعداد، أو الصق رمز إعداد / مفتاح API لذلك الملف الشخصي.

**لا يوجد ملف شخصي للمصادقة متاح (جميعها في فترة تبريد/غير متاحة)**

-   تحقق من `openclaw models status --json` للحصول على `auth.unusableProfiles`.
-   أضف ملفًا شخصيًا آخر لـ Anthropic أو انتظر فترة التبريد.

المزيد: [/gateway/troubleshooting](../gateway/troubleshooting.md) و [/help/faq](../help/faq.md).

[التبديل الاحتياطي للنموذج](../concepts/model-failover.md)[Amazon Bedrock](./bedrock.md)