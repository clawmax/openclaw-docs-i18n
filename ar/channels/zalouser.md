

  منصات المراسلة

  
# Zalo الشخصي

الحالة: تجريبي. يقوم هذا التكامل بأتمتة **حساب Zalo الشخصي** عبر `zca-js` الأصلي داخل OpenClaw.

> **تحذير:** هذا تكامل غير رسمي وقد يؤدي إلى تعليق/حظر الحساب. استخدمه على مسؤوليتك الخاصة.

## الإضافة المطلوبة

يتم توزيع Zalo الشخصي كإضافة وليس مضمنًا في التثبيت الأساسي.

-   التثبيت عبر سطر الأوامر: `openclaw plugins install @openclaw/zalouser`
-   أو من نسخة مصدرية: `openclaw plugins install ./extensions/zalouser`
-   التفاصيل: [الإضافات](../tools/plugin.md)

لا يلزم وجود ثنائي `zca`/`openzca` خارجي لسطر الأوامر.

## الإعداد السريع (للمبتدئين)

1.  قم بتثبيت الإضافة (انظر أعلاه).
2.  تسجيل الدخول (رمز QR، على جهاز البوابة):
    -   `openclaw channels login --channel zalouser`
    -   امسح رمز QR ضوئيًا باستخدام تطبيق Zalo للهاتف المحمول.
3.  تمكين القناة:

```json
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing",
    },
  },
}
```

4.  أعد تشغيل البوابة (أو أكمل عملية الإعداد).
5.  الوصول للرسائل المباشرة يكون افتراضيًا عبر الاقتران؛ وافق على رمز الاقتران عند أول اتصال.

## ما هو

-   يعمل بالكامل داخل العملية عبر `zca-js`.
-   يستخدم مستمعي الأحداث الأصليين لاستقبال الرسائل الواردة.
-   يرسل الردود مباشرة عبر واجهة برمجة التطبيقات JS (نص/وسائط/رابط).
-   مصمم لحالات استخدام "الحساب الشخصي" حيث لا تتوفر واجهة برمجة تطبيقات Zalo Bot.

## التسمية

معرف القناة هو `zalouser` لتوضيح أن هذا يقوم بأتمتة **حساب مستخدم Zalo شخصي** (غير رسمي). نحتفظ بـ `zalo` محجوزًا لتكامل محتمل مستقبلي لواجهة برمجة تطبيقات Zalo الرسمية.

## العثور على المعرفات (الدليل)

استخدم دليل سطر الأوامر لاكتشاف الأقران/المجموعات ومعرفاتهم:

```bash
openclaw directory self --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory groups list --channel zalouser --query "work"
```

## الحدود

-   يتم تقسيم النص الصادر إلى حوالي 2000 حرف (حدود عميل Zalo).
-   البث محظور افتراضيًا.

## التحكم في الوصول (الرسائل المباشرة)

تدعم `channels.zalouser.dmPolicy`: `pairing | allowlist | open | disabled` (الافتراضي: `pairing`). تقبل `channels.zalouser.allowFrom` معرفات المستخدمين أو أسمائهم. أثناء عملية الإعداد، يتم تحويل الأسماء إلى معرفات باستخدام بحث جهات الاتصال داخل العملية للإضافة. الموافقة عبر:

-   `openclaw pairing list zalouser`
-   `openclaw pairing approve zalouser `

## الوصول للمجموعات (اختياري)

-   الافتراضي: `channels.zalouser.groupPolicy = "open"` (المجموعات مسموح بها). استخدم `channels.defaults.groupPolicy` لتجاوز الافتراضي عندما لا يكون مضبوطًا.
-   قصر الوصول على قائمة السماح باستخدام:
    -   `channels.zalouser.groupPolicy = "allowlist"`
    -   `channels.zalouser.groups` (المفاتيح هي معرفات المجموعات أو أسمائها)
-   حظر جميع المجموعات: `channels.zalouser.groupPolicy = "disabled"`.
-   يمكن لمعالج التكوين أن يطلب قوائم السماح للمجموعات.
-   عند بدء التشغيل، يحول OpenClaw أسماء المجموعات/المستخدمين في قوائم السماح إلى معرفات ويسجل التعيين؛ يتم الاحتفاظ بالإدخالات غير المحولة كما هي مكتوبة.

مثال:

```json
{
  channels: {
    zalouser: {
      groupPolicy: "allowlist",
      groups: {
        "123456789": { allow: true },
        "Work Chat": { allow: true },
      },
    },
  },
}
```

### التحكم في الإشارات للمجموعات

-   تتحكم `channels.zalouser.groups..requireMention` في ما إذا كانت ردود المجموعة تتطلب إشارة.
-   ترتيب التحويل: معرف/اسم المجموعة المطابق تمامًا -> الرمز المعياري للمجموعة -> `*` -> الافتراضي (`true`).
-   ينطبق هذا على كل من المجموعات المسموح بها ووضع المجموعات المفتوحة.

مثال:

```json
{
  channels: {
    zalouser: {
      groupPolicy: "allowlist",
      groups: {
        "*": { allow: true, requireMention: true },
        "Work Chat": { allow: true, requireMention: false },
      },
    },
  },
}
```

## الحسابات المتعددة

تتطابق الحسابات مع ملفات تعريف `zalouser` في حالة OpenClaw. مثال:

```json
{
  channels: {
    zalouser: {
      enabled: true,
      defaultAccount: "default",
      accounts: {
        work: { enabled: true, profile: "work" },
      },
    },
  },
}
```

## الكتابة، التفاعلات، وإقرارات التسليم

-   يرسل OpenClaw حدث كتابة قبل إرسال الرد (بأفضل جهد).
-   إجراء تفاعل الرسالة `react` مدعوم لـ `zalouser` في إجراءات القناة.
    -   استخدم `remove: true` لإزالة رمز تفاعل معين من رسالة.
    -   دلالات التفاعل: [التفاعلات](../tools/reactions.md)
-   بالنسبة للرسائل الواردة التي تتضمن بيانات وصفية للأحداث، يرسل OpenClaw إقرارات بالتسليم + المشاهدة (بأفضل جهد).

## استكشاف الأخطاء وإصلاحها

**تسجيل الدخول لا يثبت:**

-   `openclaw channels status --probe`
-   إعادة تسجيل الدخول: `openclaw channels logout --channel zalouser && openclaw channels login --channel zalouser`

**اسم قائمة السماح/المجموعة لم يتم تحويله:**

-   استخدم المعرفات الرقمية في `allowFrom`/`groups`، أو أسماء الأصدقاء/المجموعات المطابقة تمامًا.

**تم الترقية من إعداد قديم يعتمد على سطر الأوامر:**

-   قم بإزالة أي افتراضات قديمة لعملية `zca` خارجية.
-   تعمل القناة الآن بالكامل داخل OpenClaw دون ثنائيات سطر الأوامر الخارجية.

[Zalo](./zalo.md)[الاقتران](./pairing.md)