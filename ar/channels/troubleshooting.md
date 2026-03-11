

  التكوين

  
# استكشاف أخطاء القناة

استخدم هذه الصفحة عندما تتصل قناة ولكن سلوكها خاطئ.

## سلم الأوامر

شغّل هذه الأوامر بالترتيب أولاً:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

المستوى الأساسي السليم:

-   `Runtime: running`
-   `RPC probe: ok`
-   فحص القناة يظهر متصل/جاهز

## WhatsApp

### علامات فشل WhatsApp

| العرض | أسرع فحص | الإصلاح |
| --- | --- | --- |
| متصل ولكن لا توجد ردود في الرسائل المباشرة | `openclaw pairing list whatsapp` | قم بالموافقة على المرسل أو غيّر سياسة الرسائل المباشرة/القائمة المسموح بها. |
| يتم تجاهل رسائل المجموعة | تحقق من `requireMention` + أنماط الإشارة في التكوين | أشر إلى البوت أو خفّض سياسة الإشارة لتلك المجموعة. |
| انقطاع عشوائي/حلقات إعادة تسجيل دخول | `openclaw channels status --probe` + السجلات | أعد تسجيل الدخول وتحقق من أن دليل بيانات الاعتماد سليم. |

استكشاف الأخطاء الكامل: [/channels/whatsapp#troubleshooting-quick](./whatsapp.md#troubleshooting-quick)

## Telegram

### علامات فشل Telegram

| العرض | أسرع فحص | الإصلاح |
| --- | --- | --- |
| `/start` ولكن لا يوجد تدفق ردود قابل للاستخدام | `openclaw pairing list telegram` | قم بالموافقة على الاقتران أو غيّر سياسة الرسائل المباشرة. |
| البوت متصل ولكن المجموعة تبقى صامتة | تحقق من شرط الإشارة ووضع الخصوصية للبوت | عطّل وضع الخصوصية لرؤية المجموعة أو أشر إلى البوت. |
| فشل الإرسال مع أخطاء الشبكة | افحص السجلات بحثًا عن إخفاقات استدعاء واجهة برمجة تطبيقات Telegram | أصلح توجيه DNS/IPv6/الوكيل إلى `api.telegram.org`. |
| تم الترقية والقائمة المسموح بها تمنعك | `openclaw security audit` وقوائم السماح في التكوين | شغّل `openclaw doctor --fix` أو استبدل `@username` بمعرفات المرسل الرقمية. |

استكشاف الأخطاء الكامل: [/channels/telegram#troubleshooting](./telegram.md#troubleshooting)

## Discord

### علامات فشل Discord

| العرض | أسرع فحص | الإصلاح |
| --- | --- | --- |
| البوت متصل ولكن لا توجد ردود في الخادم | `openclaw channels status --probe` | اسمح بالخادم/القناة وتحقق من نية محتوى الرسالة. |
| يتم تجاهل رسائل المجموعة | تحقق من السجلات بحثًا عن عمليات إسقاط بوابة الإشارة | أشر إلى البوت أو عيّن `requireMention: false` للخادم/القناة. |
| ردود الرسائل المباشرة مفقودة | `openclaw pairing list discord` | قم بالموافقة على اقتران الرسائل المباشرة أو اضبط سياسة الرسائل المباشرة. |

استكشاف الأخطاء الكامل: [/channels/discord#troubleshooting](./discord.md#troubleshooting)

## Slack

### علامات فشل Slack

| العرض | أسرع فحص | الإصلاح |
| --- | --- | --- |
| وضع المقبس متصل ولكن لا توجد استجابات | `openclaw channels status --probe` | تحقق من رمز التطبيق + رمز البوت والنطاقات المطلوبة. |
| الرسائل المباشرة محظورة | `openclaw pairing list slack` | قم بالموافقة على الاقتران أو خفّض سياسة الرسائل المباشرة. |
| يتم تجاهل رسالة القناة | تحقق من `groupPolicy` وقائمة السماح بالقناة | اسمح بالقناة أو غيّر السياسة إلى `open`. |

استكشاف الأخطاء الكامل: [/channels/slack#troubleshooting](./slack.md#troubleshooting)

## iMessage و BlueBubbles

### علامات فشل iMessage و BlueBubbles

| العرض | أسرع فحص | الإصلاح |
| --- | --- | --- |
| لا توجد أحداث واردة | تحقق من إمكانية الوصول إلى خطاف الويب/الخادم وأذونات التطبيق | أصلح عنوان URL لخطاف الويب أو حالة خادم BlueBubbles. |
| يمكن الإرسال ولكن لا يوجد استقبال على macOS | تحقق من أذونات الخصوصية في macOS لأتمتة الرسائل | أعد منح أذونات TCC وأعد تشغيل عملية القناة. |
| مرسل الرسالة المباشرة محظور | `openclaw pairing list imessage` أو `openclaw pairing list bluebubbles` | قم بالموافقة على الاقتران أو حدّث القائمة المسموح بها. |

استكشاف الأخطاء الكامل:

-   [/channels/imessage#troubleshooting-macos-privacy-and-security-tcc](./imessage.md#troubleshooting-macos-privacy-and-security-tcc)
-   [/channels/bluebubbles#troubleshooting](./bluebubbles.md#troubleshooting)

## Signal

### علامات فشل Signal

| العرض | أسرع فحص | الإصلاح |
| --- | --- | --- |
| البرنامج الخفي يمكن الوصول إليه ولكن البوت صامت | `openclaw channels status --probe` | تحقق من عنوان URL/حساب برنامج `signal-cli` الخفي ووضع الاستقبال. |
| الرسالة المباشرة محظورة | `openclaw pairing list signal` | قم بالموافقة على المرسل أو اضبط سياسة الرسائل المباشرة. |
| لا يتم تشغيل ردود المجموعة | تحقق من قائمة السماح للمجموعة وأنماط الإشارة | أضف المرسل/المجموعة أو خفّض البوابة. |

استكشاف الأخطاء الكامل: [/channels/signal#troubleshooting](./signal.md#troubleshooting)

## Matrix

### علامات فشل Matrix

| العرض | أسرع فحص | الإصلاح |
| --- | --- | --- |
| تم تسجيل الدخول ولكن يتجاهل رسائل الغرفة | `openclaw channels status --probe` | تحقق من `groupPolicy` وقائمة السماح بالغرفة. |
| لا تتم معالجة الرسائل المباشرة | `openclaw pairing list matrix` | قم بالموافقة على المرسل أو اضبط سياسة الرسائل المباشرة. |
| تفشل الغرف المشفرة | تحقق من وحدة التشفير وإعدادات التشفير | فعّل دعم التشفير وأعد الانضمام/مزامنة الغرفة. |

استكشاف الأخطاء الكامل: [/channels/matrix#troubleshooting](./matrix.md#troubleshooting)

[تحليل موقع القناة](./location.md)

---