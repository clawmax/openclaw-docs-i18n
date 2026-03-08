title: "استطلاعات الرأي الآلية لتلغرام وواتساب وديسكورد ومايكروسوفت تيمز"
description: "تعلم كيفية إنشاء استطلاعات الرأي عبر سطر الأوامر (CLI) وبروتوكول الاستدعاء البعيد (RPC) وأدوات الوكيل عبر تلغرام وواتساب وديسكورد ومايكروسوفت تيمز. يتضمن خيارات وأمثلة خاصة بكل قناة."
keywords: ["استطلاعات الرأي الآلية", "استطلاع تلغرام", "استطلاع واتساب", "استطلاع ديسكورد", "استطلاع ميكروسوفت تيمز", "openclaw cli", "واجهة برمجة تطبيقات الاستطلاع", "استطلاعات متعددة القنوات"]
---

  الأتمتة

  
# استطلاعات الرأي

## القنوات المدعومة

-   تلغرام
-   واتساب (القناة الويب)
-   ديسكورد
-   مايكروسوفت تيمز (البطاقات التكيفية)

## سطر الأوامر (CLI)

```bash
# Telegram
openclaw message poll --channel telegram --target 123456789 \
  --poll-question "Ship it?" --poll-option "Yes" --poll-option "No"
openclaw message poll --channel telegram --target -1001234567890:topic:42 \
  --poll-question "Pick a time" --poll-option "10am" --poll-option "2pm" \
  --poll-duration-seconds 300

# WhatsApp
openclaw message poll --target +15555550123 \
  --poll-question "Lunch today?" --poll-option "Yes" --poll-option "No" --poll-option "Maybe"
openclaw message poll --target 123456789@g.us \
  --poll-question "Meeting time?" --poll-option "10am" --poll-option "2pm" --poll-option "4pm" --poll-multi

# Discord
openclaw message poll --channel discord --target channel:123456789 \
  --poll-question "Snack?" --poll-option "Pizza" --poll-option "Sushi"
openclaw message poll --channel discord --target channel:123456789 \
  --poll-question "Plan?" --poll-option "A" --poll-option "B" --poll-duration-hours 48

# MS Teams
openclaw message poll --channel msteams --target conversation:19:abc@thread.tacv2 \
  --poll-question "Lunch?" --poll-option "Pizza" --poll-option "Sushi"
```

الخيارات:

-   `--channel`: `whatsapp` (الافتراضي)، `telegram`، `discord`، أو `msteams`
-   `--poll-multi`: السماح بتحديد خيارات متعددة
-   `--poll-duration-hours`: خاص بديسكورد فقط (الافتراضي 24 عند حذفه)
-   `--poll-duration-seconds`: خاص بتلغرام فقط (5-600 ثانية)
-   `--poll-anonymous` / `--poll-public`: خاص بتلغرام فقط (خصوصية الاستطلاع)

## بروتوكول الاستدعاء البعيد للبوابة (Gateway RPC)

الطريقة: `poll` المعاملات:

-   `to` (مقطع، مطلوب)
-   `question` (مقطع، مطلوب)
-   `options` (مصفوفة مقاطع، مطلوبة)
-   `maxSelections` (رقم، اختياري)
-   `durationHours` (رقم، اختياري)
-   `durationSeconds` (رقم، اختياري، خاص بتلغرام فقط)
-   `isAnonymous` (منطقي، اختياري، خاص بتلغرام فقط)
-   `channel` (مقطع، اختياري، الافتراضي: `whatsapp`)
-   `idempotencyKey` (مقطع، مطلوب)

## الاختلافات بين القنوات

-   تلغرام: 2-10 خيارات. يدعم مواضيع المنتدى عبر `threadId` أو أهداف `:topic:`. يستخدم `durationSeconds` بدلاً من `durationHours`، محدود بـ 5-600 ثانية. يدعم استطلاعات الرأي المجهولة والعامة.
-   واتساب: 2-12 خيارًا، يجب أن يكون `maxSelections` ضمن عدد الخيارات، يتجاهل `durationHours`.
-   ديسكورد: 2-10 خيارات، `durationHours` مقيدة بين 1-768 ساعة (الافتراضي 24). `maxSelections > 1` يُفعّل الاختيار المتعدد؛ ديسكورد لا يدعم عدد اختيارات صارم.
-   مايكروسوفت تيمز: استطلاعات الرأي باستخدام البطاقات التكيفية (تديرها OpenClaw). لا يوجد واجهة برمجة تطبيقات أصلية للاستطلاع؛ `durationHours` يتم تجاهله.

## أداة الوكيل (Message)

استخدم أداة `message` مع إجراء `poll` (`to`, `pollQuestion`, `pollOption`, اختياري `pollMulti`, `pollDurationHours`, `channel`). بالنسبة لتلغرام، تقبل الأداة أيضًا `pollDurationSeconds`, `pollAnonymous`, و `pollPublic`. استخدم `action: "poll"` لإنشاء الاستطلاع. يتم رفض حقول الاستطلاع الممررة مع `action: "send"`. ملاحظة: ديسكورد ليس لديها وضع "اختر بالضبط N"؛ `pollMulti` يتحول إلى اختيار متعدد. يتم عرض استطلاعات تيمز كبطاقات تكيفية وتتطلب بقاء البوابة متصلة لتسجيل الأصوات في `~/.openclaw/msteams-polls.json`.

[Gmail PubSub](./gmail-pubsub.md)[Auth Monitoring](./auth-monitoring.md)

---