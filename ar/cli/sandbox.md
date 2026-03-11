

  أوامر CLI

  
# Sandbox CLI

إدارة حاويات بيئة الاختبار المعزولة (Sandbox) القائمة على Docker لتنفيذ الوكيل المعزول.

## نظرة عامة

يمكن لـ OpenClaw تشغيل الوكلاء في حاويات Docker معزولة لأغراض الأمان. تساعدك أوامر `sandbox` في إدارة هذه الحاويات، خاصة بعد التحديثات أو تغييرات التكوين.

## الأوامر

### openclaw sandbox explain

افحص **الإعدادات الفعالة** لوضع/نطاق/وصول مساحة العمل لبيئة الاختبار، وسياسة أدوات بيئة الاختبار، والبوابات المرتفعة (مع مسارات مفاتيح التكوين للإصلاح).

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

### openclaw sandbox list

اعرض قائمة بجميع حاويات بيئة الاختبار مع حالتها وتكوينها.

```bash
openclaw sandbox list
openclaw sandbox list --browser  # إدراج حاويات المتصفح فقط
openclaw sandbox list --json     # إخراج بتنسيق JSON
```

**يتضمن الإخراج:**

-   اسم الحاوية وحالتها (قيد التشغيل/متوقفة)
-   صورة Docker وما إذا كانت تطابق التكوين
-   العمر (الوقت منذ الإنشاء)
-   وقت الخمول (الوقت منذ آخر استخدام)
-   الجلسة/الوكيل المرتبط

### openclaw sandbox recreate

أزل حاويات بيئة الاختبار لإجبار إعادة إنشائها باستخدام الصور/التكوينات المحدثة.

```bash
openclaw sandbox recreate --all                # إعادة إنشاء جميع الحاويات
openclaw sandbox recreate --session main       # جلسة محددة
openclaw sandbox recreate --agent mybot        # وكيل محدد
openclaw sandbox recreate --browser            # حاويات المتصفح فقط
openclaw sandbox recreate --all --force        # تخطي التأكيد
```

**الخيارات:**

-   `--all`: إعادة إنشاء جميع حاويات بيئة الاختبار
-   `--session `: إعادة إنشاء الحاوية لجلسة محددة
-   `--agent `: إعادة إنشاء الحاويات لوكيل محدد
-   `--browser`: إعادة إنشاء حاويات المتصفح فقط
-   `--force`: تخطي مطالبة التأكيد

**هام:** يتم إعادة إنشاء الحاويات تلقائيًا عند استخدام الوكيل التالي.

## حالات الاستخدام

### بعد تحديث صور Docker

```bash
# سحب الصورة الجديدة
docker pull openclaw-sandbox:latest
docker tag openclaw-sandbox:latest openclaw-sandbox:bookworm-slim

# تحديث التكوين لاستخدام الصورة الجديدة
# تحرير التكوين: agents.defaults.sandbox.docker.image (أو agents.list[].sandbox.docker.image)

# إعادة إنشاء الحاويات
openclaw sandbox recreate --all
```

### بعد تغيير تكوين بيئة الاختبار

```bash
# تحرير التكوين: agents.defaults.sandbox.* (أو agents.list[].sandbox.*)

# إعادة الإنشاء لتطبيق التكوين الجديد
openclaw sandbox recreate --all
```

### بعد تغيير setupCommand

```bash
openclaw sandbox recreate --all
# أو لوكيل واحد فقط:
openclaw sandbox recreate --agent family
```

### لوكيل محدد فقط

```bash
# تحديث حاويات وكيل واحد فقط
openclaw sandbox recreate --agent alfred
```

## لماذا هذا مطلوب؟

**المشكلة:** عند تحديث صور Docker أو تكوين بيئة الاختبار:

-   تستمر الحاويات الحالية في التشغيل بالإعدادات القديمة
-   يتم تنظيف الحاويات فقط بعد 24 ساعة من الخمول
-   تحتفظ الوكلاء المستخدمة بانتظام بالحاويات القديمة قيد التشغيل إلى أجل غير مسمى

**الحل:** استخدم `openclaw sandbox recreate` لإجبار إزالة الحاويات القديمة. سيتم إعادة إنشائها تلقائيًا بالإعدادات الحالية عند الحاجة التالية. نصيحة: يُفضل استخدام `openclaw sandbox recreate` بدلاً من `docker rm` اليدوي. فهو يستخدم تسمية الحاوية الخاصة بـ Gateway ويتجنب عدم التطابق عند تغيير مفاتيح النطاق/الجلسة.

## التكوين

توجد إعدادات بيئة الاختبار في `~/.openclaw/openclaw.json` تحت `agents.defaults.sandbox` (يتم تجاوزها لكل وكيل في `agents.list[].sandbox`):

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "all", // off, non-main, all
        "scope": "agent", // session, agent, shared
        "docker": {
          "image": "openclaw-sandbox:bookworm-slim",
          "containerPrefix": "openclaw-sbx-",
          // ... المزيد من خيارات Docker
        },
        "prune": {
          "idleHours": 24, // التنظيف التلقائي بعد 24 ساعة خمول
          "maxAgeDays": 7, // التنظيف التلقائي بعد 7 أيام
        },
      },
    },
  },
}
```

## انظر أيضًا

-   [توثيق بيئة الاختبار](../gateway/sandboxing.md)
-   [تكوين الوكيل](../concepts/agent-workspace.md)
-   [أمر Doctor](../gateway/doctor.md) - التحقق من إعداد بيئة الاختبار

[reset](./reset.md)[secrets](./secrets.md)

---