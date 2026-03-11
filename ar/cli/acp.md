

  أوامر سطر الأوامر

  
# acp

شغّل جسر [بروتوكول عميل الوكيل (ACP)](https://agentclientprotocol.com/) الذي يتحدث إلى بوابة OpenClaw Gateway. يتحدث هذا الأمر بروتوكول ACP عبر الإدخال/الإخراج القياسي لبيئات التطوير المتكاملة ويُمرر المطالبات إلى البوابة عبر WebSocket. يحتفظ بجلسات ACP معينة بمفاتيح جلسة البوابة.

## الاستخدام

```bash
openclaw acp

# بوابة بعيدة
openclaw acp --url wss://gateway-host:18789 --token <token>

# بوابة بعيدة (الرمز من ملف)
openclaw acp --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token

# الانضمام إلى مفتاح جلسة موجود
openclaw acp --session agent:main:main

# الانضمام بواسطة التسمية (يجب أن تكون موجودة مسبقًا)
openclaw acp --session-label "support inbox"

# إعادة تعيين مفتاح الجلسة قبل المطالبة الأولى
openclaw acp --session agent:main:main --reset-session
```

## عميل ACP (تصحيح)

استخدم عميل ACP المدمج للتحقق من سلامة الجسر دون بيئة تطوير متكاملة. يقوم بتشغيل جسر ACP ويسمح لك بكتابة المطالبات بشكل تفاعلي.

```bash
openclaw acp client

# توجيه الجسر المُشغل إلى بوابة بعيدة
openclaw acp client --server-args --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token

# تجاوز أمر الخادم (الافتراضي: openclaw)
openclaw acp client --server "node" --server-args openclaw.mjs acp --url ws://127.0.0.1:19001
```

نموذج الأذونات (وضع تصحيح العميل):

-   الموافقة التلقائية تعتمد على قائمة السماح وتنطبق فقط على معرفات الأدوات الأساسية الموثوقة.
-   نطاق الموافقة التلقائية `read` يقتصر على دليل العمل الحالي (`--cwd` عند تعيينه).
-   أسماء الأدوات غير المعروفة/غير الأساسية، وقراءات خارج النطاق، والأدوات الخطيرة تتطلب دائمًا موافقة صريحة عبر مطالبة.
-   يتم التعامل مع `toolCall.kind` المقدم من الخادم كبيانات وصفية غير موثوقة (وليست مصدرًا للتفويض).

## كيفية استخدام هذا

استخدم ACP عندما تتحدث بيئة تطوير متكاملة (أو عميل آخر) بروتوكول عميل الوكيل وتريد منها تشغيل جلسة بوابة OpenClaw Gateway.

1.  تأكد من تشغيل البوابة (محليًا أو بعيدًا).
2.  قم بتكوين هدف البوابة (التكوين أو الأعلام).
3.  وجه بيئة التطوير المتكاملة لتشغيل `openclaw acp` عبر الإدخال/الإخراج القياسي.

مثال على التكوين (مستمر):

```bash
openclaw config set gateway.remote.url wss://gateway-host:18789
openclaw config set gateway.remote.token <token>
```

مثال على التشغيل المباشر (بدون كتابة تكوين):

```bash
openclaw acp --url wss://gateway-host:18789 --token <token>
# مفضل لسلامة العمليات المحلية
openclaw acp --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token
```

## اختيار الوكلاء

لا يختار ACP الوكلاء مباشرة. يقوم بالتوجيه عبر مفتاح جلسة البوابة. استخدم مفاتيح الجلسات المحددة للوكيل لاستهداف وكيل معين:

```bash
openclaw acp --session agent:main:main
openclaw acp --session agent:design:main
openclaw acp --session agent:qa:bug-123
```

تتطابق كل جلسة ACP مع مفتاح جلسة بوابة واحد. يمكن أن يكون للوكيل الواحد العديد من الجلسات؛ يستخدم ACP افتراضيًا جلسة معزولة `acp:` ما لم تقم بتجاوز المفتاح أو التسمية.

## إعداد محرر Zed

أضف وكيل ACP مخصص في `~/.config/zed/settings.json` (أو استخدم واجهة مستخدم إعدادات Zed):

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": ["acp"],
      "env": {}
    }
  }
}
```

لاستهداف بوابة أو وكيل معين:

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": [
        "acp",
        "--url",
        "wss://gateway-host:18789",
        "--token",
        "<token>",
        "--session",
        "agent:design:main"
      ],
      "env": {}
    }
  }
}
```

في Zed، افتح لوحة الوكيل واختر "OpenClaw ACP" لبدء محادثة.

## تعيين الجلسات

افتراضيًا، تحصل جلسات ACP على مفتاح جلسة بوابة معزول ببادئة `acp:`. لإعادة استخدام جلسة معروفة، مرر مفتاح جلسة أو تسمية:

-   `--session `: استخدم مفتاح جلسة بوابة محدد.
-   `--session-label `: حل جلسة موجودة بواسطة التسمية.
-   `--reset-session`: أنشئ معرف جلسة جديد لذلك المفتاح (نفس المفتاح، محضر جديد).

إذا كان عميل ACP الخاص بك يدعم البيانات الوصفية، يمكنك تجاوزها لكل جلسة:

```json
{
  "_meta": {
    "sessionKey": "agent:main:main",
    "sessionLabel": "support inbox",
    "resetSession": true
  }
}
```

تعلم المزيد عن مفاتيح الجلسات في [/concepts/session](../concepts/session.md).

## الخيارات

-   `--url `: عنوان URL لـ WebSocket الخاص بالبوابة (يُستخدم افتراضيًا gateway.remote.url عند التكوين).
-   `--token `: رمز مصادقة البوابة.
-   `--token-file `: قراءة رمز مصادقة البوابة من ملف.
-   `--password `: كلمة مرور مصادقة البوابة.
-   `--password-file `: قراءة كلمة مرور مصادقة البوابة من ملف.
-   `--session `: مفتاح الجلسة الافتراضي.
-   `--session-label `: التسمية الافتراضية للجلسة لحلها.
-   `--require-existing`: فشل إذا لم يكن مفتاح/تسمية الجلسة موجودًا.
-   `--reset-session`: إعادة تعيين مفتاح الجلسة قبل أول استخدام.
-   `--no-prefix-cwd`: عدم إضافة بادئة للمطالبات بدليل العمل.
-   `--verbose, -v`: تسجيل تفصيلي إلى stderr.

ملاحظة أمنية:

-   يمكن أن يكون `--token` و `--password` مرئيين في قوائم العمليات المحلية على بعض الأنظمة.
-   يُفضل استخدام `--token-file`/`--password-file` أو متغيرات البيئة (`OPENCLAW_GATEWAY_TOKEN`, `OPENCLAW_GATEWAY_PASSWORD`).
-   عمليات الخلفية الفرعية لوقت تشغيل ACP تستقبل `OPENCLAW_SHELL=acp`، والتي يمكن استخدامها لقواعد shell/profile خاصة بالسياق.
-   يضبط `openclaw acp client` `OPENCLAW_SHELL=acp-client` على عملية الجسر المُشغلة.

### خيارات عميل acp

-   `--cwd `: دليل العمل لجلسة ACP.
-   `--server `: أمر خادم ACP (الافتراضي: `openclaw`).
-   `--server-args <args...>`: وسائط إضافية تُمرر إلى خادم ACP.
-   `--server-verbose`: تمكين التسجيل التفصيلي على خادم ACP.
-   `--verbose, -v`: تسجيل تفصيلي للعميل.

[مرجع سطر الأوامر](../cli.md)[وكيل](./agent.md)