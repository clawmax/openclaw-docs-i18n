

  أوامر CLI

  
# hooks

إدارة هوات الوكيل (أتمتة قائمة على الأحداث لأوامر مثل `/new`، `/reset`، وبدء تشغيل البوابة). ذات صلة:

-   الهوات: [الهوات](../automation/hooks.md)
-   هوات الإضافات: [الإضافات](../tools/plugin.md#plugin-hooks)

## عرض قائمة جميع الهوات

```bash
openclaw hooks list
```

عرض قائمة بجميع الهوات المكتشفة من أدلة مساحة العمل، المدارة، والمضمّنة. **الخيارات:**

-   `--eligible`: عرض الهوات المؤهلة فقط (المتطلبات مستوفاة)
-   `--json`: الإخراج بتنسيق JSON
-   `-v, --verbose`: عرض معلومات مفصلة تشمل المتطلبات المفقودة

**مثال على الإخراج:**

```
الهوات (4/4 جاهزة)

جاهزة:
  🚀 boot-md ✓ - تشغيل BOOT.md عند بدء تشغيل البوابة
  📎 bootstrap-extra-files ✓ - حقن ملفات إضافية لتهيئة مساحة العمل أثناء تهيئة الوكيل
  📝 command-logger ✓ - تسجيل جميع أحداث الأوامر في ملف مركزي للمراجعة
  💾 session-memory ✓ - حفظ سياق الجلسة في الذاكرة عند إصدار أمر /new
```

**مثال (مفصل):**

```bash
openclaw hooks list --verbose
```

يعرض المتطلبات المفقودة للهوات غير المؤهلة. **مثال (JSON):**

```bash
openclaw hooks list --json
```

يعيد JSON منظمًا للاستخدام البرمجي.

## الحصول على معلومات الهوك

```bash
openclaw hooks info <name>
```

عرض معلومات مفصلة عن هوك محدد. **المعطيات:**

-   ``: اسم الهوك (مثل، `session-memory`)

**الخيارات:**

-   `--json`: الإخراج بتنسيق JSON

**مثال:**

```bash
openclaw hooks info session-memory
```

**الإخراج:**

```
💾 session-memory ✓ جاهز

حفظ سياق الجلسة في الذاكرة عند إصدار أمر /new

التفاصيل:
  المصدر: openclaw-bundled
  المسار: /path/to/openclaw/hooks/bundled/session-memory/HOOK.md
  المعالج: /path/to/openclaw/hooks/bundled/session-memory/handler.ts
  الصفحة الرئيسية: https://docs.openclaw.ai/automation/hooks#session-memory
  الأحداث: command:new

المتطلبات:
  التكوين: ✓ workspace.dir
```

## التحقق من أهلية الهوات

```bash
openclaw hooks check
```

عرض ملخص لحالة أهلية الهوات (كم عدد الجاهزة مقابل غير الجاهزة). **الخيارات:**

-   `--json`: الإخراج بتنسيق JSON

**مثال على الإخراج:**

```
حالة الهوات

إجمالي الهوات: 4
جاهزة: 4
غير جاهزة: 0
```

## تمكين هوك

```bash
openclaw hooks enable <name>
```

تمكين هوك محدد عن طريق إضافته إلى تكوينك (`~/.openclaw/config.json`). **ملاحظة:** الهوات المدارة بواسطة إضافات تظهر كـ `plugin:` في `openclaw hooks list` ولا يمكن تمكينها/تعطيلها هنا. قم بتمكين/تعطيل الإضافة بدلاً من ذلك. **المعطيات:**

-   ``: اسم الهوك (مثل، `session-memory`)

**مثال:**

```bash
openclaw hooks enable session-memory
```

**الإخراج:**

```
✓ تم تمكين الهوك: 💾 session-memory
```

**ما يفعله:**

-   يتحقق من وجود الهوك وأهليته
-   يقوم بتحديث `hooks.internal.entries..enabled = true` في تكوينك
-   يحفظ التكوين على القرص

**بعد التمكين:**

-   أعد تشغيل البوابة لإعادة تحميل الهوات (إعادة تشغيل تطبيق شريط القوائم على macOS، أو إعادة تشغيل عملية البوابة الخاصة بك في وضع التطوير).

## تعطيل هوك

```bash
openclaw hooks disable <name>
```

تعطيل هوك محدد عن طريق تحديث تكوينك. **المعطيات:**

-   ``: اسم الهوك (مثل، `command-logger`)

**مثال:**

```bash
openclaw hooks disable command-logger
```

**الإخراج:**

```
⏸ تم تعطيل الهوك: 📝 command-logger
```

**بعد التعطيل:**

-   أعد تشغيل البوابة لإعادة تحميل الهوات

## تثبيت الهوات

```bash
openclaw hooks install <path-or-spec>
openclaw hooks install <npm-spec> --pin
```

تثبيت حزمة هوات من مجلد/أرشيف محلي أو npm. مواصفات npm هي **للسجل فقط** (اسم الحزمة + **إصدار محدد** اختياري أو **علامة توزيع**). يتم رفض مواصفات Git/URL/الملف ونطاقات semver. يتم تشغيل تثبيت التبعيات بـ `--ignore-scripts` للسلامة. تبقى المواصفات العارية و `@latest` على المسار المستقر. إذا قام npm بحل أي منهما إلى إصدار أولي، يتوقف OpenClaw ويطلب منك الموافقة صراحة باستخدام علامة إصدار أولي مثل `@beta`/`@rc` أو إصدار أولي محدد. **ما يفعله:**

-   ينسخ حزمة الهوك إلى `~/.openclaw/hooks/`
-   يقوم بتمكين الهوات المثبتة في `hooks.internal.entries.*`
-   يسجل التثبيت تحت `hooks.internal.installs`

**الخيارات:**

-   `-l, --link`: ربط دليل محلي بدلاً من النسخ (يضيفه إلى `hooks.internal.load.extraDirs`)
-   `--pin`: تسجيل تثبيتات npm كـ `name@version` محدد تم حله في `hooks.internal.installs`

**الأرشيفات المدعومة:** `.zip`, `.tgz`, `.tar.gz`, `.tar` **أمثلة:**

```bash
# دليل محلي
openclaw hooks install ./my-hook-pack

# أرشيف محلي
openclaw hooks install ./my-hook-pack.zip

# حزمة NPM
openclaw hooks install @openclaw/my-hook-pack

# ربط دليل محلي بدون نسخ
openclaw hooks install -l ./my-hook-pack
```

## تحديث الهوات

```bash
openclaw hooks update <id>
openclaw hooks update --all
```

تحديث حزم الهوات المثبتة (تثبيتات npm فقط). **الخيارات:**

-   `--all`: تحديث جميع حزم الهوات المتعقبة
-   `--dry-run`: عرض ما سيتم تغييره دون الكتابة

عند وجود تجزئة سلامة مخزنة وتغير تجزئة الأداة التي تم جلبها، يطبع OpenClaw تحذيرًا ويطلب التأكيد قبل المتابعة. استخدم `--yes` العام لتجاوز المطالبات في عمليات التشغيل غير التفاعلية أو في CI.

## الهوات المضمّنة

### session-memory

يحفظ سياق الجلسة في الذاكرة عند إصدار أمر `/new`. **التمكين:**

```bash
openclaw hooks enable session-memory
```

**الإخراج:** `~/.openclaw/workspace/memory/YYYY-MM-DD-slug.md` **انظر:** [توثيق session-memory](../automation/hooks.md#session-memory)

### bootstrap-extra-files

يحقن ملفات تهيئة إضافية (على سبيل المثال `AGENTS.md` / `TOOLS.md` المحلية لمستودع أحادي) أثناء `agent:bootstrap`. **التمكين:**

```bash
openclaw hooks enable bootstrap-extra-files
```

**انظر:** [توثيق bootstrap-extra-files](../automation/hooks.md#bootstrap-extra-files)

### command-logger

يسجل جميع أحداث الأوامر في ملف مركزي للمراجعة. **التمكين:**

```bash
openclaw hooks enable command-logger
```

**الإخراج:** `~/.openclaw/logs/commands.log` **عرض السجلات:**

```bash
# الأوامر الحديثة
tail -n 20 ~/.openclaw/logs/commands.log

# طباعة منسقة
cat ~/.openclaw/logs/commands.log | jq .

# التصفية حسب الإجراء
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**انظر:** [توثيق command-logger](../automation/hooks.md#command-logger)

### boot-md

يشغل `BOOT.md` عند بدء تشغيل البوابة (بعد بدء القنوات). **الأحداث**: `gateway:startup` **التمكين**:

```bash
openclaw hooks enable boot-md
```

**انظر:** [توثيق boot-md](../automation/hooks.md#boot-md)

[health](./health.md)[logs](./logs.md)