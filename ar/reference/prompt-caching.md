

  مرجع تقني

  
# التخزين المؤقت للطلبات

التخزين المؤقت للطلبات يعني أن مزود النموذج يمكنه إعادة استخدام بادئات الطلبات غير المتغيرة (عادة تعليمات النظام/المطور والسياق الثابت الآخر) عبر الدورات بدلاً من معالجتها مرة أخرى في كل مرة. يكتب الطلب الأول المطابق رموز التخزين المؤقت (`cacheWrite`)، ويمكن للطلبات المطابقة اللاحقة قراءتها مرة أخرى (`cacheRead`). أهمية ذلك: تكلفة أقل للرموز المميزة، استجابات أسرع، وأداء أكثر قابلية للتنبؤ للجلسات طويلة الأمد. بدون التخزين المؤقت، تدفع الطلبات المتكررة تكلفة الطلب الكاملة في كل دورة حتى عندما لا يتغير معظم الإدخال. تغطي هذه الصفحة جميع المقابض المتعلقة بالتخزين المؤقت التي تؤثر على إعادة استخدام الطلبات وتكلفة الرموز المميزة. للحصول على تفاصيل تسعير أنثروبيك، انظر: [https://docs.anthropic.com/docs/build-with-claude/prompt-caching](https://docs.anthropic.com/docs/build-with-claude/prompt-caching)

## المقابض الأساسية

### cacheRetention (على مستوى النموذج والوكيل)

عيّن احتفاظ التخزين المؤقت في معلمات النموذج:

```yaml
agents:
  defaults:
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "short" # none | short | long
```

تجاوز لكل وكيل:

```yaml
agents:
  list:
    - id: "alerts"
      params:
        cacheRetention: "none"
```

ترتيب دمج التكوين:

1.  `agents.defaults.models["provider/model"].params`
2.  `agents.list[].params` (مطابقة معرف الوكيل؛ يتجاوز بالمفتاح)

### cacheControlTtl القديم

لا تزال القيم القديمة مقبولة ويتم تعيينها:

-   `5m` -> `short`
-   `1h` -> `long`

يفضل استخدام `cacheRetention` للتكوين الجديد.

### contextPruning.mode: "cache-ttl"

يقوم بتقليم سياق نتائج الأدوات القديمة بعد نوافذ مهلة التخزين المؤقت (TTL) حتى لا تعيد الطلبات بعد فترات الخمول تخزين السجل كبير الحجم مؤقتًا.

```yaml
agents:
  defaults:
    contextPruning:
      mode: "cache-ttl"
      ttl: "1h"
```

انظر [تقليم الجلسة](../concepts/session-pruning.md) للحصول على السلوك الكامل.

### نبض القلب للحفاظ على الدفء

يمكن أن يحافظ نبض القلب على دفء نوافذ التخزين المؤقت ويقلل من عمليات كتابة التخزين المؤقت المتكررة بعد فجوات الخمول.

```yaml
agents:
  defaults:
    heartbeat:
      every: "55m"
```

يتم دعم نبض القلب لكل وكيل في `agents.list[].heartbeat`.

## سلوك المزود

### أنثروبيك (API مباشر)

-   يتم دعم `cacheRetention`.
-   مع ملفات تعريف مصادقة مفتاح واجهة برمجة تطبيقات أنثروبيك، يقوم OpenClaw بتهيئة `cacheRetention: "short"` لمراجع نموذج أنثروبيك عندما لا تكون محددة.

### Amazon Bedrock

-   تدعم مراجع نموذج أنثروبيك كلود (`amazon-bedrock/*anthropic.claude*`) تمرير `cacheRetention` الصريح.
-   يتم إجبار نماذج Bedrock غير الأنثروبيكية على `cacheRetention: "none"` أثناء وقت التشغيل.

### نماذج أنثروبيك على OpenRouter

لمراجع النموذج `openrouter/anthropic/*`، يقوم OpenClaw بحقن `cache_control` الخاص بأنثروبيك في كتل طلبات النظام/المطور لتحسين إعادة استخدام تخزين الطلبات المؤقت.

### مزودون آخرون

إذا كان المزود لا يدعم وضع التخزين المؤقت هذا، فلن يكون لـ `cacheRetention` أي تأثير.

## أنماط الضبط

### حركة مرور مختلطة (الافتراضي الموصى به)

احتفظ بخط أساس طويل الأمد على وكيلك الرئيسي، وعطّل التخزين المؤقت على وكلاء الإشعارات الانفجارية:

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-6"
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "long"
  list:
    - id: "research"
      default: true
      heartbeat:
        every: "55m"
    - id: "alerts"
      params:
        cacheRetention: "none"
```

### خط أساس يعطي الأولوية للتكلفة

-   عيّن خط الأساس `cacheRetention: "short"`.
-   فعّل `contextPruning.mode: "cache-ttl"`.
-   احتفظ بنبض القلب أقل من مهلة التخزين المؤقت (TTL) فقط للوكلاء الذين يستفيدون من التخزين المؤقت الدافئ.

## تشخيص التخزين المؤقت

يكشف OpenClaw عن تشخيصات مخصصة لتتبع التخزين المؤقت لتشغيلات الوكيل المضمنة.

### تكوين diagnostics.cacheTrace

```yaml
diagnostics:
  cacheTrace:
    enabled: true
    filePath: "~/.openclaw/logs/cache-trace.jsonl" # اختياري
    includeMessages: false # الافتراضي true
    includePrompt: false # الافتراضي true
    includeSystem: false # الافتراضي true
```

القيم الافتراضية:

-   `filePath`: `$OPENCLAW_STATE_DIR/logs/cache-trace.jsonl`
-   `includeMessages`: `true`
-   `includePrompt`: `true`
-   `includeSystem`: `true`

### مفاتيح تبديل البيئة (لتصحيح الأخطاء لمرة واحدة)

-   `OPENCLAW_CACHE_TRACE=1` يفعل تتبع التخزين المؤقت.
-   `OPENCLAW_CACHE_TRACE_FILE=/path/to/cache-trace.jsonl` يتجاوز مسار الإخراج.
-   `OPENCLAW_CACHE_TRACE_MESSAGES=0|1` يبدل التقاط حمولة الرسالة الكاملة.
-   `OPENCLAW_CACHE_TRACE_PROMPT=0|1` يبدل التقاط نص الطلب.
-   `OPENCLAW_CACHE_TRACE_SYSTEM=0|1` يبدل التقاط طلب النظام.

### ما يجب فحصه

-   أحداث تتبع التخزين المؤقت هي JSONL وتتضمن لقطات مرحلية مثل `session:loaded`، `prompt:before`، `stream:context`، و `session:after`.
-   تأثير الرمز المميز للتخزين المؤقت لكل دورة مرئي في أسطح الاستخدام العادية عبر `cacheRead` و `cacheWrite` (على سبيل المثال `/usage full` وملخصات استخدام الجلسة).

## استكشاف الأخطاء وإصلاحها السريع

-   `cacheWrite` مرتفع في معظم الأدوار: تحقق من مدخلات طلب النظام المتقلبة وتأكد من أن النموذج/المزود يدعم إعدادات التخزين المؤقت الخاصة بك.
-   لا تأثير لـ `cacheRetention`: تأكد من تطابق مفتاح النموذج مع `agents.defaults.models["provider/model"]`.
-   طلبات Bedrock Nova/Mistral مع إعدادات التخزين المؤقت: من المتوقع إجبار وقت التشغيل على `none`.

وثائق ذات صلة:

-   [أنثروبيك](../providers/anthropic.md)
-   [استخدام الرموز المميزة والتكاليف](./token-use.md)
-   [تقليم الجلسة](../concepts/session-pruning.md)
-   [مرجع تكوين البوابة](../gateway/configuration-reference.md)

[سطح بيانات الاعتماد SecretRef](./secretref-credential-surface.md)[استخدام API والتكاليف](./api-usage-costs.md)