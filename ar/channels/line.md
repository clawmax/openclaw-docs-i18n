

  منصات المراسلة

  
# LINE

يتصل LINE بـ OpenClaw عبر واجهة برمجة تطبيقات مراسلة LINE. تعمل الإضافة كمستقبل لـ webhook على البوابة وتستخدم رمز وصول القناة + سر القناة للمصادقة. الحالة: مدعومة عبر الإضافة. الرسائل المباشرة، محادثات المجموعات، الوسائط، المواقع، رسائل Flex، رسائل القوالب، والردود السريعة مدعومة. التفاعلات والخيوط غير مدعومة.

## الإضافة مطلوبة

قم بتثبيت إضافة LINE:

```bash
openclaw plugins install @openclaw/line
```

التثبيت المحلي (عند التشغيل من مستودع git):

```bash
openclaw plugins install ./extensions/line
```

## الإعداد

1.  أنشئ حسابًا لمطوري LINE وافتح لوحة التحكم: [https://developers.line.biz/console/](https://developers.line.biz/console/)
2.  أنشئ (أو اختر) موفرًا وأضف قناة من نوع **Messaging API**.
3.  انسخ **رمز وصول القناة** و**سر القناة** من إعدادات القناة.
4.  فعّل **استخدام webhook** في إعدادات واجهة برمجة تطبيقات المراسلة.
5.  عيّن عنوان URL لـ webhook لينتهي عند نقطة نهاية البوابة الخاصة بك (HTTPS مطلوب):

```
https://gateway-host/line/webhook
```

تستجيب البوابة للتحقق من webhook الخاص بـ LINE (GET) والأحداث الواردة (POST). إذا كنت بحاجة إلى مسار مخصص، عيّن `channels.line.webhookPath` أو `channels.line.accounts..webhookPath` وقم بتحديث عنوان URL وفقًا لذلك. ملاحظة أمنية:

-   يعتمد التحقق من توقيع LINE على محتوى الجسم (HMAC فوق الجسم الخام)، لذلك يطبق OpenClaw حدودًا صارمة لحجم الجسم وزمن المهلة قبل التحقق.

## التهيئة

التهيئة الدنيا:

```json
{
  channels: {
    line: {
      enabled: true,
      channelAccessToken: "LINE_CHANNEL_ACCESS_TOKEN",
      channelSecret: "LINE_CHANNEL_SECRET",
      dmPolicy: "pairing",
    },
  },
}
```

متغيرات البيئة (للحساب الافتراضي فقط):

-   `LINE_CHANNEL_ACCESS_TOKEN`
-   `LINE_CHANNEL_SECRET`

ملفات الرمز/السر:

```json
{
  channels: {
    line: {
      tokenFile: "/path/to/line-token.txt",
      secretFile: "/path/to/line-secret.txt",
    },
  },
}
```

حسابات متعددة:

```json
{
  channels: {
    line: {
      accounts: {
        marketing: {
          channelAccessToken: "...",
          channelSecret: "...",
          webhookPath: "/line/marketing",
        },
      },
    },
  },
}
```

## التحكم في الوصول

الرسائل المباشرة تكون افتراضيًا على وضع الاقتران. المرسلون غير المعروفين يحصلون على رمز اقتران ويتم تجاهل رسائلهم حتى تتم الموافقة عليهم.

```bash
openclaw pairing list line
openclaw pairing approve line <CODE>
```

قوائم السماح والسياسات:

-   `channels.line.dmPolicy`: `pairing | allowlist | open | disabled`
-   `channels.line.allowFrom`: معرفات مستخدمي LINE المسموح لهم للرسائل المباشرة
-   `channels.line.groupPolicy`: `allowlist | open | disabled`
-   `channels.line.groupAllowFrom`: معرفات مستخدمي LINE المسموح لهم للمجموعات
-   تجاوزات لكل مجموعة: `channels.line.groups..allowFrom`
-   ملاحظة وقت التشغيل: إذا كان `channels.line` مفقودًا تمامًا، يتراجع وقت التشغيل إلى `groupPolicy="allowlist"` للتحقق من المجموعات (حتى لو تم تعيين `channels.defaults.groupPolicy`).

معرفات LINE حساسة لحالة الأحرف. المعرفات الصالحة تبدو كالتالي:

-   المستخدم: `U` + 32 حرفًا سداسيًا عشريًا
-   المجموعة: `C` + 32 حرفًا سداسيًا عشريًا
-   الغرفة: `R` + 32 حرفًا سداسيًا عشريًا

## سلوك الرسالة

-   يتم تقسيم النص إلى أجزاء عند 5000 حرف.
-   يتم إزالة تنسيق Markdown؛ يتم تحويل كتل التعليمات البرمجية والجداول إلى بطاقات Flex عندما يكون ذلك ممكنًا.
-   يتم تخزين الردود المتدفقة في المخزن المؤقت؛ يتلقى LINE أجزاء كاملة مع رسوم متحركة للتحميل أثناء عمل الوكيل.
-   يتم تحديد حجم تنزيل الوسائط بواسطة `channels.line.mediaMaxMb` (الافتراضي 10).

## بيانات القناة (رسائل غنية)

استخدم `channelData.line` لإرسال ردود سريعة، أو مواقع، أو بطاقات Flex، أو رسائل قوالب.

```json
{
  text: "ها أنت ذا",
  channelData: {
    line: {
      quickReplies: ["الحالة", "مساعدة"],
      location: {
        title: "المكتب",
        address: "123 الشارع الرئيسي",
        latitude: 35.681236,
        longitude: 139.767125,
      },
      flexMessage: {
        altText: "بطاقة الحالة",
        contents: {
          /* حمولة Flex */
        },
      },
      templateMessage: {
        type: "confirm",
        text: "المتابعة؟",
        confirmLabel: "نعم",
        confirmData: "yes",
        cancelLabel: "لا",
        cancelData: "no",
      },
    },
  },
}
```

تأتي إضافة LINE أيضًا مع أمر `/card` للنماذج المسبقة لرسائل Flex:

```bash
/card info "مرحبًا" "شكرًا للانضمام!"
```

## استكشاف الأخطاء وإصلاحها

-   **فشل التحقق من webhook:** تأكد من أن عنوان URL لـ webhook هو HTTPS وأن `channelSecret` يتطابق مع ما في لوحة تحكم LINE.
-   **لا توجد أحداث واردة:** تأكد من تطابق مسار webhook مع `channels.line.webhookPath` وأن البوابة يمكن الوصول إليها من LINE.
-   **أخطاء تنزيل الوسائط:** قم برفع قيمة `channels.line.mediaMaxMb` إذا تجاوزت الوسائط الحد الافتراضي.

[IRC](./irc.md)[Matrix](./matrix.md)

---