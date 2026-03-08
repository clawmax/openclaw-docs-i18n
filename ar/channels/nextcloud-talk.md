title: "دليل إعداد وتكوين إضافة OpenClaw لـ Nextcloud Talk"
description: "تعلم كيفية تثبيت وتكوين بوت OpenClaw لـ Nextcloud Talk. يدعم الرسائل المباشرة، غرف المجموعات، التفاعلات، وترميز Markdown."
keywords: ["nextcloud talk", "إضافة openclaw", "تكامل روبوت المحادثة", "بوت ويب هوك", "رسائل مباشرة", "غرف المجموعات", "تكوين البوت", "منصة مراسلة"]
---

  منصات المراسلة

  
# Nextcloud Talk

الحالة: مدعوم عبر إضافة (بوت ويب هوك). الرسائل المباشرة، الغرف، التفاعلات، والرسائل بتنسيق Markdown مدعومة.

## الإضافة مطلوبة

يتم توزيع Nextcloud Talk كإضافة وليست مضمنة في التثبيت الأساسي. قم بالتثبيت عبر سطر الأوامر (سجل npm):

```bash
openclaw plugins install @openclaw/nextcloud-talk
```

التثبيت المحلي (عند التشغيل من مستودع git):

```bash
openclaw plugins install ./extensions/nextcloud-talk
```

إذا اخترت Nextcloud Talk أثناء التكوين/التهيئة وتم اكتشاف نسخة git محلية، فسيعرض OpenClaw مسار التثبيت المحلي تلقائيًا. التفاصيل: [الإضافات](../tools/plugin.md)

## الإعداد السريع (للمبتدئين)

1.  قم بتثبيت إضافة Nextcloud Talk.
2.  على خادم Nextcloud الخاص بك، أنشئ بوتًا:
    
    نسخ
    
    ```bash
    ./occ talk:bot:install "OpenClaw" "<shared-secret>" "<webhook-url>" --feature reaction
    ```
    
3.  قم بتمكين البوت في إعدادات الغرفة المستهدفة.
4.  قم بتكوين OpenClaw:
    -   التكوين: `channels.nextcloud-talk.baseUrl` + `channels.nextcloud-talk.botSecret`
    -   أو متغيرات البيئة: `NEXTCLOUD_TALK_BOT_SECRET` (الحساب الافتراضي فقط)
5.  أعد تشغيل البوابة (أو أكمل عملية التهيئة).

الحد الأدنى للتكوين:

```json
{
  channels: {
    "nextcloud-talk": {
      enabled: true,
      baseUrl: "https://cloud.example.com",
      botSecret: "shared-secret",
      dmPolicy: "pairing",
    },
  },
}
```

## ملاحظات

-   لا يمكن للبوتات بدء الرسائل المباشرة. يجب على المستخدم مراسلة البوت أولاً.
-   يجب أن يكون عنوان URL لـ webhook قابلاً للوصول من قبل البوابة؛ قم بتعيين `webhookPublicUrl` إذا كنت خلف وكيل.
-   رفع الوسائط غير مدعوم بواسطة واجهة برمجة تطبيقات البوت؛ يتم إرسال الوسائط كعناوين URL.
-   لا يميز حمولة webhook بين الرسائل المباشرة والغرف؛ قم بتعيين `apiUser` + `apiPassword` لتمكين البحث عن نوع الغرفة (وإلا يتم التعامل مع الرسائل المباشرة كغرف).

## التحكم في الوصول (الرسائل المباشرة)

-   الافتراضي: `channels.nextcloud-talk.dmPolicy = "pairing"`. المرسلون غير المعروفين يحصلون على رمز إقران.
-   الموافقة عبر:
    -   `openclaw pairing list nextcloud-talk`
    -   `openclaw pairing approve nextcloud-talk `
-   الرسائل المباشرة العامة: `channels.nextcloud-talk.dmPolicy="open"` بالإضافة إلى `channels.nextcloud-talk.allowFrom=["*"]`.
-   `allowFrom` يطابق معرّفات مستخدم Nextcloud فقط؛ يتم تجاهل أسماء العرض.

## الغرف (المجموعات)

-   الافتراضي: `channels.nextcloud-talk.groupPolicy = "allowlist"` (مقيد بالذكر).
-   قائمة الغرف المسموح بها باستخدام `channels.nextcloud-talk.rooms`:

```json
{
  channels: {
    "nextcloud-talk": {
      rooms: {
        "room-token": { requireMention: true },
      },
    },
  },
}
```

-   لعدم السماح بأي غرف، اترك قائمة السماح فارغة أو عيّن `channels.nextcloud-talk.groupPolicy="disabled"`.

## الإمكانيات

| الميزة | الحالة |
| --- | --- |
| الرسائل المباشرة | مدعومة |
| الغرف | مدعومة |
| السلاسل النصية | غير مدعوم |
| الوسائط | عنوان URL فقط |
| التفاعلات | مدعومة |
| الأوامر الأصلية | غير مدعوم |

## مرجع التكوين (Nextcloud Talk)

التكوين الكامل: [التكوين](../gateway/configuration.md) خيارات المزود:

-   `channels.nextcloud-talk.enabled`: تمكين/تعطيل بدء تشغيل القناة.
-   `channels.nextcloud-talk.baseUrl`: عنوان URL لنسخة Nextcloud.
-   `channels.nextcloud-talk.botSecret`: السر المشترك للبوت.
-   `channels.nextcloud-talk.botSecretFile`: مسار ملف السر.
-   `channels.nextcloud-talk.apiUser`: مستخدم واجهة برمجة التطبيقات للبحث عن الغرف (كشف الرسائل المباشرة).
-   `channels.nextcloud-talk.apiPassword`: كلمة مرور واجهة برمجة التطبيقات/التطبيق للبحث عن الغرف.
-   `channels.nextcloud-talk.apiPasswordFile`: مسار ملف كلمة مرور واجهة برمجة التطبيقات.
-   `channels.nextcloud-talk.webhookPort`: منفذ مستمع webhook (الافتراضي: 8788).
-   `channels.nextcloud-talk.webhookHost`: مضيف webhook (الافتراضي: 0.0.0.0).
-   `channels.nextcloud-talk.webhookPath`: مسار webhook (الافتراضي: /nextcloud-talk-webhook).
-   `channels.nextcloud-talk.webhookPublicUrl`: عنوان URL لـ webhook يمكن الوصول إليه خارجيًا.
-   `channels.nextcloud-talk.dmPolicy`: `pairing | allowlist | open | disabled`.
-   `channels.nextcloud-talk.allowFrom`: قائمة السماح للرسائل المباشرة (معرّفات المستخدم). `open` تتطلب `"*"`.
-   `channels.nextcloud-talk.groupPolicy`: `allowlist | open | disabled`.
-   `channels.nextcloud-talk.groupAllowFrom`: قائمة السماح للمجموعات (معرّفات المستخدم).
-   `channels.nextcloud-talk.rooms`: إعدادات وقائمة السماح لكل غرفة.
-   `channels.nextcloud-talk.historyLimit`: حد سجل المجموعة (0 يعطّل).
-   `channels.nextcloud-talk.dmHistoryLimit`: حد سجل الرسائل المباشرة (0 يعطّل).
-   `channels.nextcloud-talk.dms`: تجاوزات لكل رسالة مباشرة (historyLimit).
-   `channels.nextcloud-talk.textChunkLimit`: حجم تجزئة النص الصادر (أحرف).
-   `channels.nextcloud-talk.chunkMode`: `length` (الافتراضي) أو `newline` للتقسيم على الأسطر الفارغة (حدود الفقرات) قبل التجزئة حسب الطول.
-   `channels.nextcloud-talk.blockStreaming`: تعطيل بث الكتل لهذه القناة.
-   `channels.nextcloud-talk.blockStreamingCoalesce`: ضبط دمج بث الكتل.
-   `channels.nextcloud-talk.mediaMaxMb`: الحد الأقصى للوسائط الواردة (ميغابايت).

[Microsoft Teams](./msteams.md)[Nostr](./nostr.md)