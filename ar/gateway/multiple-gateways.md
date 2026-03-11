

  التكوين والعمليات

  
# بوابات متعددة

يجب أن تستخدم معظم الإعدادات بوابة واحدة لأن بوابة واحدة يمكنها التعامل مع اتصالات مراسلة ووكلاء متعددين. إذا كنت بحاجة إلى عزل أقوى أو تكرار (مثل بوت إنقاذ)، فشغل بوابات منفصلة بملفات تعريف/منافذ معزولة.

## قائمة التحقق للعزل (مطلوبة)

-   `OPENCLAW_CONFIG_PATH` — ملف التكوين لكل مثيل
-   `OPENCLAW_STATE_DIR` — الجلسات، بيانات الاعتماد، ذاكرة التخزين المؤقت لكل مثيل
-   `agents.defaults.workspace` — مساحة العمل الجذرية لكل مثيل
-   `gateway.port` (أو `--port`) — فريد لكل مثيل
-   المنافذ المشتقة (المتصفح/اللوحة القماشية) يجب ألا تتداخل

إذا تم مشاركة هذه العناصر، فستواجه سباقات تكوين وتعارضات في المنافذ.

## موصى به: الملفات الشخصية (--profile)

تقوم الملفات الشخصية تلقائيًا بتحديد نطاق `OPENCLAW_STATE_DIR` + `OPENCLAW_CONFIG_PATH` وإضافة لاحقة لأسماء الخدمات.

```bash
# رئيسي
openclaw --profile main setup
openclaw --profile main gateway --port 18789

# إنقاذ
openclaw --profile rescue setup
openclaw --profile rescue gateway --port 19001
```

الخدمات لكل ملف تعريف:

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

## دليل بوت الإنقاذ

شغل بوابة ثانية على نفس المضيف مع ما يلي خاص بها:

-   ملف تعريف/تكوين
-   دليل حالة
-   مساحة عمل
-   منفذ أساسي (بالإضافة إلى المنافذ المشتقة)

هذا يحافظ على عزل بوت الإنقاذ عن البوت الرئيسي حتى يتمكن من تصحيح الأخطاء أو تطبيق تغييرات التكوين إذا كان البوت الأساسي معطلاً. تباعد المنافذ: اترك على الأقل 20 منفذًا بين المنافذ الأساسية حتى لا تتصادم منافذ المتصفح/اللوحة القماشية/CDP المشتقة أبدًا.

### كيفية التثبيت (بوت الإنقاذ)

```bash
# البوت الرئيسي (موجود أو جديد، بدون معامل --profile)
# يعمل على المنفذ 18789 + منافذ Chrome CDC/Canvas/...
openclaw onboard
openclaw gateway install

# بوت الإنقاذ (ملف تعريف معزول + منافذ)
openclaw --profile rescue onboard
# ملاحظات:
# - سيتم إضافة لاحقة "-rescue" لاسم مساحة العمل افتراضيًا
# - يجب أن يكون المنفذ على الأقل 18789 + 20 منفذًا،
#   من الأفضل اختيار منفذ أساسي مختلف تمامًا، مثل 19789،
# - بقية عملية الإعداد الأولي هي نفسها كالمعتاد

# لتثبيت الخدمة (إذا لم يحدث ذلك تلقائيًا أثناء الإعداد الأولي)
openclaw --profile rescue gateway install
```

## تعيين المنافذ (مشتق)

المنفذ الأساسي = `gateway.port` (أو `OPENCLAW_GATEWAY_PORT` / `--port`).

-   منفذ خدمة تحكم المتصفح = الأساسي + 2 (حلقة راجعة فقط)
-   يتم تقديم مضيف اللوحة القماشية على خادم HTTP للبوابة (نفس منفذ `gateway.port`)
-   يتم تخصيص منافذ CDP لملف تعريف المتصفح تلقائيًا من `browser.controlPort + 9 .. + 108`

إذا قمت بتجاوز أي من هذه الإعدادات في التكوين أو البيئة، فيجب أن تحافظ على كونها فريدة لكل مثيل.

## ملاحظات المتصفح/CDP (مشكلة شائعة)

-   **لا** تثبت `browser.cdpUrl` على نفس القيم في مثيلات متعددة.
-   يحتاج كل مثيل إلى منفذ تحكم متصفح خاص به ونطاق CDP (مشتق من منفذ بوابته).
-   إذا كنت بحاجة إلى منافذ CDP صريحة، فاضبط `browser.profiles..cdpPort` لكل مثيل.
-   Chrome عن بُعد: استخدم `browser.profiles..cdpUrl` (لكل ملف تعريف، لكل مثيل).

## مثال يدوي للبيئة

```
OPENCLAW_CONFIG_PATH=~/.openclaw/main.json \
OPENCLAW_STATE_DIR=~/.openclaw-main \
openclaw gateway --port 18789

OPENCLAW_CONFIG_PATH=~/.openclaw/rescue.json \
OPENCLAW_STATE_DIR=~/.openclaw-rescue \
openclaw gateway --port 19001
```

## فحوصات سريعة

```bash
openclaw --profile main status
openclaw --profile rescue status
openclaw --profile rescue browser status
```

[أداة التنفيذ في الخلفية والعملية](./background-process.md)[استكشاف الأخطاء وإصلاحها](./troubleshooting.md)

---