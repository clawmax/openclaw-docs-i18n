title: "أوامر دليل OpenClaw CLI للبحث في القنوات"
description: "تعلم كيفية استخدام أمر دليل OpenClaw CLI للعثور على معرفات المستخدمين والمجموعات وجهات الاتصال عبر WhatsApp وSlack وDiscord وقنوات أخرى للرسائل."
keywords: ["دليل openclaw", "بحث دليل cli", "جهات اتصال القناة", "العثور على معرف المستخدم", "هدف الرسالة", "قائمة الأقران", "قائمة المجموعات", "أوامر cli"]
---

  أوامر CLI

  
# directory

عمليات البحث في الدليل للقنوات التي تدعمه (جهات الاتصال/الأقران، والمجموعات، و"أنا").

## الأعلام الشائعة

-   `--channel `: معرف/اسم مستعار للقناة (مطلوب عند تكوين عدة قنوات؛ تلقائي عند تكوين قناة واحدة فقط)
-   `--account `: معرف الحساب (الافتراضي: الافتراضي للقناة)
-   `--json`: إخراج JSON

## ملاحظات

-   الغرض من `directory` هو مساعدتك في العثور على معرفات يمكنك لصقها في أوامر أخرى (خاصة `openclaw message send --target ...`).
-   بالنسبة للعديد من القنوات، تكون النتائج مدعومة بالإعدادات (القوائم المسموح بها / المجموعات المُهيأة) وليست دليلاً مباشرًا من المزود.
-   الإخراج الافتراضي هو `id` (وأحيانًا `name`) مفصولة بعلامة تبويب؛ استخدم `--json` للبرمجة النصية.

## استخدام النتائج مع إرسال الرسائل

```bash
openclaw directory peers list --channel slack --query "U0"
openclaw message send --channel slack --target user:U012ABCDEF --message "hello"
```

## تنسيقات المعرف (حسب القناة)

-   WhatsApp: `+15551234567` (رسالة مباشرة)، `1234567890-1234567890@g.us` (مجموعة)
-   Telegram: `@username` أو معرف دردشة رقمي؛ المجموعات هي معرفات رقمية
-   Slack: `user:U…` و `channel:C…`
-   Discord: `user:` و `channel:`
-   Matrix (المكوّن الإضافي): `user:@user:server`، `room:!roomId:server`، أو `#alias:server`
-   Microsoft Teams (المكوّن الإضافي): `user:` و `conversation:`
-   Zalo (المكوّن الإضافي): معرف المستخدم (Bot API)
-   Zalo Personal / `zalouser` (المكوّن الإضافي): معرف المحادثة (رسالة مباشرة/مجموعة) من `zca` (`me`، `friend list`، `group list`)

## الذات ("أنا")

```bash
openclaw directory self --channel zalouser
```

## الأقران (جهات الاتصال/المستخدمون)

```bash
openclaw directory peers list --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory peers list --channel zalouser --limit 50
```

## المجموعات

```bash
openclaw directory groups list --channel zalouser
openclaw directory groups list --channel zalouser --query "work"
openclaw directory groups members --channel zalouser --group-id <id>
```

[devices](./devices.md)[dns](./dns.md)