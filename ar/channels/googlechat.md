

  منصات المراسلة

  
# Google Chat

الحالة: جاهز للرسائل الخاصة + المساحات عبر webhooks واجهة برمجة تطبيقات Google Chat (HTTP فقط).

## الإعداد السريع (للمبتدئين)

1.  أنشئ مشروع Google Cloud وقم بتمكين **واجهة برمجة تطبيقات Google Chat**.
    -   انتقل إلى: [بيانات اعتماد واجهة برمجة تطبيقات Google Chat](https://console.cloud.google.com/apis/api/chat.googleapis.com/credentials)
    -   قم بتمكين الواجهة البرمجية إذا لم تكن مفعلة بالفعل.
2.  أنشئ **حساب خدمة**:
    -   اضغط على **إنشاء بيانات اعتماد** > **حساب خدمة**.
    -   سمّه كما تشاء (مثال: `openclaw-chat`).
    -   اترك الأذونات فارغة (اضغط **متابعة**).
    -   اترك المبادئ مع الوصول فارغة (اضغط **تم**).
3.  أنشئ وقم بتنزيل **مفتاح JSON**:
    -   في قائمة حسابات الخدمة، انقر على الحساب الذي أنشأته للتو.
    -   انتقل إلى علامة التبويب **المفاتيح**.
    -   انقر **إضافة مفتاح** > **إنشاء مفتاح جديد**.
    -   اختر **JSON** واضغط **إنشاء**.
4.  خزّن ملف JSON الذي تم تنزيله على مضيف البوابة الخاص بك (مثال: `~/.openclaw/googlechat-service-account.json`).
5.  أنشئ تطبيق Google Chat في [تكوين دردشة Google Cloud Console](https://console.cloud.google.com/apis/api/chat.googleapis.com/hangouts-chat):
    -   املأ **معلومات التطبيق**:
        -   **اسم التطبيق**: (مثال: `OpenClaw`)
        -   **رابط الصورة الرمزية**: (مثال: `https://openclaw.ai/logo.png`)
        -   **الوصف**: (مثال: `مساعد الذكاء الاصطناعي الشخصي`)
    -   فعّل **الميزات التفاعلية**.
    -   ضمن **الوظائف**، حدد **الانضمام إلى المساحات ومحادثات المجموعة**.
    -   ضمن **إعدادات الاتصال**، اختر **رابط نقطة نهاية HTTP**.
    -   ضمن **المحفزات**، اختر **استخدام رابط نقطة نهاية HTTP مشترك لجميع المحفزات** وعيّنه على الرابط العام للبوابة الخاص بك متبوعًا بـ `/googlechat`.
        -   *تلميح: شغّل `openclaw status` للعثور على الرابط العام للبوابة.*
    -   ضمن **الرؤية**، حدد **جعل تطبيق الدردشة هذا متاحًا لأشخاص ومجموعات محددين في `نطاقك`**.
    -   أدخل عنوان بريدك الإلكتروني (مثال: `user@example.com`) في مربع النص.
    -   انقر **حفظ** في الأسفل.
6.  **فعّل حالة التطبيق**:
    -   بعد الحفظ، **حدّث الصفحة**.
    -   ابحث عن قسم **حالة التطبيق** (عادة بالقرب من الأعلى أو الأسفل بعد الحفظ).
    -   غيّر الحالة إلى **مباشر - متاح للمستخدمين**.
    -   انقر **حفظ** مرة أخرى.
7.  قم بتكوين OpenClaw مع مسار حساب الخدمة + جمهور webhook:
    -   متغير البيئة: `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE=/path/to/service-account.json`
    -   أو التكوين: `channels.googlechat.serviceAccountFile: "/path/to/service-account.json"`.
8.  عيّن نوع جمهور webhook + القيمة (يطابق تكوين تطبيق الدردشة الخاص بك).
9.  ابدأ البوابة. سيقوم Google Chat بإرسال POST إلى مسار webhook الخاص بك.

## الإضافة إلى Google Chat

بمجرد تشغيل البوابة وإضافة بريدك الإلكتروني إلى قائمة الرؤية:

1.  انتقل إلى [Google Chat](https://chat.google.com/).
2.  انقر على أيقونة **+** (زائد) بجوار **الرسائل المباشرة**.
3.  في شريط البحث (حيث تضيف الأشخاص عادةً)، اكتب **اسم التطبيق** الذي قمت بتكوينه في Google Cloud Console.
    -   **ملاحظة**: لن يظهر الروبوت في قائمة التصفح "المتجر" لأنه تطبيق خاص. يجب البحث عنه بالاسم.
4.  اختر الروبوت الخاص بك من النتائج.
5.  انقر **إضافة** أو **دردشة** لبدء محادثة فردية.
6.  أرسل "مرحبًا" لتفعيل المساعد!

## الرابط العام (Webhook فقط)

تتطلب webhooks الخاصة بـ Google Chat نقطة نهاية HTTPS عامة. لأسباب أمنية، **قم بتعريض مسار `/googlechat` فقط** للإنترنت. احتفظ بلوحة تحكم OpenClaw ونقاط النهاية الحساسة الأخرى على شبكتك الخاصة.

### الخيار أ: Tailscale Funnel (موصى به)

استخدم Tailscale Serve للوحة التحكم الخاصة و Funnel لمسار webhook العام. هذا يحافظ على `/` خاصًا مع تعريض `/googlechat` فقط.

1.  **تحقق من العنوان الذي ترتبط به البوابة:**
    
    Copy
    
    ```bash
    ss -tlnp | grep 18789
    ```
    
    لاحظ عنوان IP (مثال: `127.0.0.1`، `0.0.0.0`، أو عنوان Tailscale الخاص بك مثل `100.x.x.x`).
2.  **عرض لوحة التحكم للشبكة الذيلية فقط (المنفذ 8443):**
    
    Copy
    
    ```bash
    # إذا كان مرتبطًا بـ localhost (127.0.0.1 أو 0.0.0.0):
    tailscale serve --bg --https 8443 http://127.0.0.1:18789
    
    # إذا كان مرتبطًا بعنوان Tailscale IP فقط (مثال: 100.106.161.80):
    tailscale serve --bg --https 8443 http://100.106.161.80:18789
    ```
    
3.  **عرض مسار webhook فقط للعامة:**
    
    Copy
    
    ```bash
    # إذا كان مرتبطًا بـ localhost (127.0.0.1 أو 0.0.0.0):
    tailscale funnel --bg --set-path /googlechat http://127.0.0.1:18789/googlechat
    
    # إذا كان مرتبطًا بعنوان Tailscale IP فقط (مثال: 100.106.161.80):
    tailscale funnel --bg --set-path /googlechat http://100.106.161.80:18789/googlechat
    ```
    
4.  **تفويض العقدة للوصول إلى Funnel:** إذا طُلب منك، قم بزيارة رابط التفويض المعروض في الناتج لتمكين Funnel لهذه العقدة في سياسة الشبكة الذيلية الخاصة بك.
5.  **تحقق من التكوين:**
    
    Copy
    
    ```
    tailscale serve status
    tailscale funnel status
    ```
    

سيكون رابط webhook العام الخاص بك: `https://<node-name>..ts.net/googlechat` تبقى لوحة التحكم الخاصة بك للشبكة الذيلية فقط: `https://<node-name>..ts.net:8443/` استخدم الرابط العام (بدون `:8443`) في تكوين تطبيق Google Chat.

> ملاحظة: يستمر هذا التكوين عبر إعادة التشغيل. لإزالته لاحقًا، شغّل `tailscale funnel reset` و `tailscale serve reset`.

### الخيار ب: وكيل عكسي (Caddy)

إذا كنت تستخدم وكيلًا عكسيًا مثل Caddy، فقم بالوكالة فقط للمسار المحدد:

```
your-domain.com {
    reverse_proxy /googlechat* localhost:18789
}
```

مع هذا التكوين، سيتم تجاهل أي طلب إلى `your-domain.com/` أو إرجاعه كـ 404، بينما يتم توجيه `your-domain.com/googlechat` بأمان إلى OpenClaw.

### الخيار ج: نفق Cloudflare

قم بتكوين قواعد دخول النفق الخاص بك لتوجيه مسار webhook فقط:

-   **المسار**: `/googlechat` -> `http://localhost:18789/googlechat`
-   **القاعدة الافتراضية**: HTTP 404 (غير موجود)

## آلية العمل

1.  يرسل Google Chat طلبات POST عبر webhooks إلى البوابة. يتضمن كل طلب رأس `Authorization: Bearer `.
    -   يتحقق OpenClaw من مصادقة حامل الرمز قبل قراءة/تحليل أجسام webhooks الكاملة عند وجود الرأس.
    -   يتم دعم طلبات إضافة Google Workspace التي تحمل `authorizationEventObject.systemIdToken` في الجسم من خلال ميزانية أولية للجسم قبل المصادقة أكثر صرامة.
2.  يتحقق OpenClaw من الرمز مقابل `audienceType` + `audience` المُكوّنة:
    -   `audienceType: "app-url"` → الجمهور هو رابط webhook الخاص بك HTTPS.
    -   `audienceType: "project-number"` → الجمهور هو رقم مشروع Cloud.
3.  يتم توجيه الرسائل حسب المساحة:
    -   تستخدم الرسائل الخاصة مفتاح الجلسة `agent::googlechat:dm:`.
    -   تستخدم المساحات مفتاح الجلسة `agent::googlechat:group:`.
4.  الوصول إلى الرسائل الخاصة يكون بالاقتران افتراضيًا. يتلقى المرسلون غير المعروفين رمز اقتران؛ قم بالموافقة باستخدام:
    -   `openclaw pairing approve googlechat `
5.  تتطلب مساحات المجموعة الإشارة بـ @-mention افتراضيًا. استخدم `botUser` إذا كانت كشف الإشارة يحتاج إلى اسم مستخدم التطبيق.

## الأهداف

استخدم هذه المعرفات للتسليم وقوائم السماح:

-   الرسائل المباشرة: `users/` (موصى به).
-   البريد الإلكتروني الخام `name@example.com` قابل للتغيير ويستخدم فقط لمطابقة قائمة السماح المباشرة عند `channels.googlechat.dangerouslyAllowNameMatching: true`.
-   مهمل: يتم التعامل مع `users/` على أنه معرف مستخدم، وليس قائمة سماح بالبريد الإلكتروني.
-   المساحات: `spaces/`.

## أبرز نقاط التكوين

```json
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      // or serviceAccountRef: { source: "file", provider: "filemain", id: "/channels/googlechat/serviceAccount" }
      audienceType: "app-url",
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890", // optional; helps mention detection
      dm: {
        policy: "pairing",
        allowFrom: ["users/1234567890"],
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": {
          allow: true,
          requireMention: true,
          users: ["users/1234567890"],
          systemPrompt: "Short answers only.",
        },
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```

ملاحظات:

-   يمكن أيضًا تمرير بيانات اعتماد حساب الخدمة مباشرة باستخدام `serviceAccount` (سلسلة JSON).
-   `serviceAccountRef` مدعوم أيضًا (مرجع سر env/file SecretRef)، بما في ذلك المراجع لكل حساب تحت `channels.googlechat.accounts..serviceAccountRef`.
-   مسار webhook الافتراضي هو `/googlechat` إذا لم يتم تعيين `webhookPath`.
-   `dangerouslyAllowNameMatching` يعيد تمكين مطابقة المبدأ القابل للتغيير بالبريد الإلكتروني لقوائم السماح (وضع التوافق لكسر الزجاج).
-   التفاعلات متاحة عبر أداة `reactions` و `channels action` عند تمكين `actions.reactions`.
-   `typingIndicator` تدعم `none`، `message` (افتراضي)، و `reaction` (يتطلب التفاعل OAuth للمستخدم).
-   يتم تنزيل المرفقات عبر واجهة برمجة تطبيقات الدردشة وتخزينها في خط أنابيب الوسائط (الحجم محدود بـ `mediaMaxMb`).

تفاصيل مرجع الأسرار: [إدارة الأسرار](../gateway/secrets.md).

## استكشاف الأخطاء وإصلاحها

### 405 Method Not Allowed

إذا أظهر Google Cloud Logs Explorer أخطاء مثل:

```bash
status code: 405, reason phrase: HTTP error response: HTTP/1.1 405 Method Not Allowed
```

هذا يعني أن معالج webhook غير مسجل. الأسباب الشائعة:

1.  **القناة غير مكونة**: قسم `channels.googlechat` مفقود من تكوينك. تحقق باستخدام:
    
    Copy
    
    ```bash
    openclaw config get channels.googlechat
    ```
    
    إذا أعاد "Config path not found"، أضف التكوين (انظر [أبرز نقاط التكوين](#config-highlights)).
2.  **المكوّن الإضافي غير مفعّل**: تحقق من حالة المكوّن الإضافي:
    
    Copy
    
    ```bash
    openclaw plugins list | grep googlechat
    ```
    
    إذا أظهر "disabled"، أضف `plugins.entries.googlechat.enabled: true` إلى تكوينك.
3.  **البوابة لم تتم إعادة تشغيلها**: بعد إضافة التكوين، أعد تشغيل البوابة:
    
    Copy
    
    ```bash
    openclaw gateway restart
    ```
    

تحقق من أن القناة تعمل:

```bash
openclaw channels status
# يجب أن يظهر: Google Chat default: enabled, configured, ...
```

### مشاكل أخرى

-   تحقق من `openclaw channels status --probe` للبحث عن أخطاء المصادقة أو تكوين الجمهور المفقود.
-   إذا لم تصل أي رسائل، تأكد من رابط webhook لتطبيق الدردشة + اشتراكات الأحداث.
-   إذا منع التحكم بالإشارات الردود، عيّن `botUser` على اسم مورد التطبيق وتحقق من `requireMention`.
-   استخدم `openclaw logs --follow` أثناء إرسال رسالة اختبار لمعرفة ما إذا كانت الطلبات تصل إلى البوابة.

وثائق ذات صلة:

-   [تكوين البوابة](../gateway/configuration.md)
-   [الأمان](../gateway/security.md)
-   [التفاعلات](../tools/reactions.md)

[Feishu](./feishu.md)[iMessage](./imessage.md)