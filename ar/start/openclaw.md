title: "دليل إعداد مساعد OpenClaw الشخصي لـ WhatsApp و Telegram"
description: "تعلم كيفية إعداد OpenClaw كمساعد ذكي شخصي على WhatsApp و Telegram و Discord. قم بتكوين الأمان، ومساحة العمل، والجلسات، والنبضات الدورية."
keywords: ["إعداد openclaw", "مساعد شخصي", "بوت واتساب", "بوت تيليجرام", "مساحة عمل الوكيل", "نبضة دورية", "إدارة الجلسات", "تكوين openclaw"]
---

  أدلة

  
# إعداد المساعد الشخصي

OpenClaw هو بوابة لـ **Pi** agents تعمل على WhatsApp + Telegram + Discord + iMessage. تضيف الإضافات (Plugins) دعم Mattermost. هذا الدليل هو إعداد "المساعد الشخصي": رقم WhatsApp مخصص واحد يتصرف وكأنه وكيلك الدائم التشغيل.

## ⚠️ السلامة أولاً

أنت تضع الوكيل في موضع يمكنه من:

-   تشغيل أوامر على جهازك (اعتمادًا على إعداد أدوات Pi الخاصة بك)
-   قراءة/كتابة الملفات في مساحة عملك
-   إرسال رسائل للخارج عبر WhatsApp/Telegram/Discord/Mattermost (الإضافة)

ابدأ بحذر:

-   قم دائمًا بتعيين `channels.whatsapp.allowFrom` (لا تشغل أبدًا بشكل مفتوح للعالم على جهاز Mac الشخصي الخاص بك).
-   استخدم رقم WhatsApp مخصص للمساعد.
-   النبضات الدورية (Heartbeats) الآن افتراضيًا كل 30 دقيقة. قم بتعطيلها حتى تثق في الإعداد عن طريق تعيين `agents.defaults.heartbeat.every: "0m"`.

## المتطلبات الأساسية

-   OpenClaw مثبتًا وتم إعداد الدخول الأولي — راجع [البدء](./getting-started.md) إذا لم تقم بذلك بعد
-   رقم هاتف ثانٍ (SIM / eSIM / مسبق الدفع) للمساعد

## إعداد الهاتفين (موصى به)

هذا ما تريده: إذا ربطت واتسابك الشخصي بـ OpenClaw، فكل رسالة موجهة إليك تصبح "مدخلات للوكيل". هذا نادرًا ما يكون ما تريده.

## بدء سريع في 5 دقائق

1.  اقتران WhatsApp Web (يعرض رمز QR؛ امسحه بهاتف المساعد):

```bash
openclaw channels login
```

2.  ابدأ تشغيل البوابة (اتركها تعمل):

```bash
openclaw gateway --port 18789
```

3.  ضع تكوينًا بسيطًا في `~/.openclaw/openclaw.json`:

```json
{
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

الآن أرسل رسالة إلى رقم المساعد من هاتفك المدرج في القائمة المسموح بها. عند انتهاء إعداد الدخول الأولي، نفتح لوحة التحكم تلقائيًا ونطبع رابطًا نظيفًا (غير مميز). إذا طلب المصادقة، الصق الرمز المميز (token) من `gateway.auth.token` في إعدادات واجهة التحكم. لإعادة الفتح لاحقًا: `openclaw dashboard`.

## امنح الوكيل مساحة عمل (AGENTS)

يقرأ OpenClaw تعليمات التشغيل و"الذاكرة" من دليل مساحة العمل الخاص به. افتراضيًا، يستخدم OpenClaw `~/.openclaw/workspace` كمساحة عمل للوكيل، وسينشئها (بالإضافة إلى ملفات البداية `AGENTS.md`، `SOUL.md`، `TOOLS.md`، `IDENTITY.md`، `USER.md`، `HEARTBEAT.md`) تلقائيًا أثناء الإعداد/أول تشغيل للوكيل. يتم إنشاء `BOOTSTRAP.md` فقط عندما تكون مساحة العمل جديدة بالكامل (يجب ألا تعود بعد حذفها). `MEMORY.md` اختياري (لا يتم إنشاؤه تلقائيًا)؛ عند وجوده، يتم تحميله للجلسات العادية. جلسات الوكلاء الفرعية (Subagent) تحقن فقط `AGENTS.md` و `TOOLS.md`. نصيحة: تعامل مع هذا المجلد على أنه "ذاكرة" OpenClaw واجعله مستودع git (خاص بشكل مثالي) حتى يتم نسخ ملفات `AGENTS.md` + الذاكرة احتياطيًا. إذا كان git مثبتًا، يتم تهيئة مساحات العمل الجديدة تمامًا تلقائيًا.

```bash
openclaw setup
```

تخطيط مساحة العمل الكامل + دليل النسخ الاحتياطي: [مساحة عمل الوكيل](../concepts/agent-workspace.md) سير عمل الذاكرة: [الذاكرة](../concepts/memory.md) اختياري: اختر مساحة عمل مختلفة باستخدام `agents.defaults.workspace` (يدعم `~`).

```json
{
  agent: {
    workspace: "~/.openclaw/workspace",
  },
}
```

إذا كنت تقوم بالفعل بشحن ملفات مساحة العمل الخاصة بك من مستودع، يمكنك تعطيل إنشاء ملفات التهيئة (bootstrap) تمامًا:

```json
{
  agent: {
    skipBootstrap: true,
  },
}
```

## التكوين الذي يحوله إلى "مساعد"

يأتي OpenClaw بإعداد مساعد جيد افتراضيًا، لكنك عادةً سترغب في ضبط:

-   الهوية/التعليمات في `SOUL.md`
-   إعدادات التفكير الافتراضية (إذا رغبت)
-   النبضات الدورية (Heartbeats) (بمجرد أن تثق به)

مثال:

```json
{
  logging: { level: "info" },
  agent: {
    model: "anthropic/claude-opus-4-6",
    workspace: "~/.openclaw/workspace",
    thinkingDefault: "high",
    timeoutSeconds: 1800,
    // ابدأ بـ 0؛ فعّله لاحقًا.
    heartbeat: { every: "0m" },
  },
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true },
      },
    },
  },
  routing: {
    groupChat: {
      mentionPatterns: ["@openclaw", "openclaw"],
    },
  },
  session: {
    scope: "per-sender",
    resetTriggers: ["/new", "/reset"],
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 10080,
    },
  },
}
```

## الجلسات والذاكرة

-   ملفات الجلسة: `~/.openclaw/agents//sessions/{{SessionId}}.jsonl`
-   بيانات وصفية للجلسة (استخدام الرموز المميزة، المسار الأخير، إلخ): `~/.openclaw/agents//sessions/sessions.json` (قديم: `~/.openclaw/sessions/sessions.json`)
-   `/new` أو `/reset` يبدأ جلسة جديدة لتلك الدردشة (قابل للتكوين عبر `resetTriggers`). إذا أُرسلت وحدها، يرد الوكيل بتحية قصيرة لتأكيد إعادة التعيين.
-   `/compact [تعليمات]` يضغط سياق الجلسة ويبلغ عن ميزانية السياق المتبقية.

## النبضات الدورية (الوضع الاستباقي)

افتراضيًا، يشغل OpenClaw نبضة دورية كل 30 دقيقة مع المطالبة: `اقرأ HEARTBEAT.md إذا كان موجودًا (سياق مساحة العمل). اتبعه بدقة. لا تستنتج أو تكرر مهام قديمة من دردشات سابقة. إذا لم يكن هناك شيء يحتاج إلى اهتمام، رد بـ HEARTBEAT_OK.` عيّن `agents.defaults.heartbeat.every: "0m"` لتعطيله.

-   إذا كان `HEARTBEAT.md` موجودًا ولكنه فارغ فعليًا (فقط أسطر فارغة وعناوين ماركداون مثل `# Heading`)، يتخطى OpenClaw تشغيل النبضة الدورية لتوفير استدعاءات API.
-   إذا كان الملف مفقودًا، لا تزال النبضة الدورية تعمل ويقرر النموذج ما يجب فعله.
-   إذا رد الوكيل بـ `HEARTBEAT_OK` (مع حشو قصير اختياري؛ انظر `agents.defaults.heartbeat.ackMaxChars`)، فإن OpenClaw يمنع تسليم المخرجات لتلك النبضة الدورية.
-   افتراضيًا، يُسمح بتسليم النبضات الدورية إلى أهداف النمط المباشر `user:`. عيّن `agents.defaults.heartbeat.directPolicy: "block"` لمنع التسليم المباشر مع الحفاظ على تشغيل النبضات الدورية نشطًا.
-   تشغل النبضات الدورية أدوار وكيل كاملة — الفترات الأقصر تحرق المزيد من الرموز المميزة (tokens).

```json
{
  agent: {
    heartbeat: { every: "30m" },
  },
}
```

## الوسائط الداخلة والخارجة

يمكن عرض المرفقات الواردة (الصور/الصوت/المستندات) لأمرك عبر القوالب:

-   `{{MediaPath}}` (مسار الملف المؤقت المحلي)
-   `{{MediaUrl}}` (رابط URL وهمي)
-   `{{Transcript}}` (إذا كان تمكين تحويل الصوت إلى نص مفعل)

المرفقات الصادرة من الوكيل: قم بتضمين `MEDIA:<مسار-أو-رابط>` في سطر منفرد (بدون مسافات). مثال:

```
ها هي لقطة الشاشة.
MEDIA:https://example.com/screenshot.png
```

يقوم OpenClaw باستخراج هذه وإرسالها كوسائط جنبًا إلى جنب مع النص.

## قائمة مراجعة العمليات

```bash
openclaw status          # الحالة المحلية (بيانات الاعتماد، الجلسات، الأحداث في قائمة الانتظار)
openclaw status --all    # تشخيص كامل (للقراءة فقط، قابل للنسخ)
openclaw status --deep   # يضيف فحوصات صحة البوابة (Telegram + Discord)
openclaw health --json   # لقطة صحية للبوابة (WebSocket)
```

توجد السجلات تحت `/tmp/openclaw/` (افتراضي: `openclaw-YYYY-MM-DD.log`).

## الخطوات التالية

-   WebChat: [WebChat](../web/webchat.md)
-   عمليات البوابة: [كتيب تشغيل البوابة](../gateway.md)
-   Cron + التنبيهات: [وظائف Cron](../automation/cron-jobs.md)
-   رفيق شريط قائمة macOS: [تطبيق OpenClaw لنظام macOS](../platforms/macos.md)
-   تطبيق عقدة iOS: [تطبيق iOS](../platforms/ios.md)
-   تطبيق عقدة Android: [تطبيق Android](../platforms/android.md)
-   حالة Windows: [Windows (WSL2)](../platforms/windows.md)
-   حالة Linux: [تطبيق Linux](../platforms/linux.md)
-   الأمان: [الأمان](../gateway/security.md)

[إعداد الدخول الأولي: تطبيق macOS](./onboarding.md)[مرجع سطر الأوامر (CLI)](./wizard-cli-reference.md)