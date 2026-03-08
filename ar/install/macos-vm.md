

  الاستضافة والنشر

  
# الأجهزة الظاهرية (VMs) لنظام macOS

## التكوين الافتراضي الموصى به (معظم المستخدمين)

-   **خادم VPS صغير يعمل بنظام Linux** للحصول على بوابة (Gateway) دائمة التشغيل وتكلفة منخفضة. راجع [استضافة VPS](../vps.md).
-   **جهاز مخصص** (مثل Mac mini أو جهاز Linux) إذا كنت تريد تحكمًا كاملاً وعنوان **IP سكني** لأتمتة المتصفح. تمنع العديد من المواقع عناوين IP مراكز البيانات، لذا غالبًا ما يعمل التصفح المحلي بشكل أفضل.
-   **هجين:** احتفظ بالبوابة على خادم VPS رخيص، وقم بتوصيل جهاز Mac الخاص بك كـ **عقدة (node)** عندما تحتاج إلى أتمتة المتصفح/واجهة المستخدم. راجع [العقد](../nodes.md) و [بوابة عن بعد](../gateway/remote.md).

استخدم جهازًا ظاهريًا (VM) لنظام macOS عندما تحتاج تحديدًا إلى إمكانيات خاصة بنظام macOS فقط (مثل iMessage/BlueBubbles) أو تريد عزلًا صارمًا عن جهاز Mac الذي تستخدمه يوميًا.

## خيارات الأجهزة الظاهرية (VMs) لنظام macOS

### جهاز ظاهري محلي على جهاز Apple Silicon Mac الخاص بك (Lume)

شغل OpenClaw في جهاز ظاهري معزول لنظام macOS على جهاز Apple Silicon Mac الحالي باستخدام [Lume](https://cua.ai/docs/lume). هذا يمنحك:

-   بيئة macOS كاملة في عزلة (جهازك المضيف يبقى نظيفًا)
-   دعم iMessage عبر BlueBubbles (مستحيل على Linux/Windows)
-   إعادة تعيين فورية عن طريق استنساخ الأجهزة الظاهرية
-   لا توجد تكاليف إضافية للأجهزة أو السحابة

### موفرو استضافة أجهزة Mac (في السحابة)

إذا كنت تريد نظام macOS في السحابة، فإن موفري استضافة أجهزة Mac يعملون أيضًا:

-   [MacStadium](https://www.macstadium.com/) (أجهزة Mac مستضافة)
-   بائعي أجهزة Mac المستضافة الآخرين يعملون أيضًا؛ اتبع وثائقهم الخاصة بإعداد الجهاز الظاهري و SSH

بمجرد حصولك على وصول SSH إلى جهاز ظاهري لنظام macOS، تابع من الخطوة 6 أدناه.

* * *

## المسار السريع (Lume، للمستخدمين ذوي الخبرة)

1.  قم بتثبيت Lume
2.  `lume create openclaw --os macos --ipsw latest`
3.  أكمل "مساعد الإعداد"، وقم بتمكين "تسجيل الدخول عن بعد" (SSH)
4.  `lume run openclaw --no-display`
5.  ادخل عبر SSH، قم بتثبيت OpenClaw، قم بتكوين القنوات
6.  انتهيت

* * *

## ما تحتاجه (Lume)

-   جهاز Apple Silicon Mac (M1/M2/M3/M4)
-   نظام macOS Sequoia أو أحدث على الجهاز المضيف
-   مساحة قرص حرة تبلغ حوالي 60 جيجابايت لكل جهاز ظاهري
-   حوالي 20 دقيقة

* * *

## 1) تثبيت Lume

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/trycua/cua/main/libs/lume/scripts/install.sh)"
```

إذا لم يكن المسار `~/.local/bin` مضافًا إلى PATH الخاص بك:

```bash
echo 'export PATH="$PATH:$HOME/.local/bin"' >> ~/.zshrc && source ~/.zshrc
```

تحقق:

```bash
lume --version
```

الوثائق: [تثبيت Lume](https://cua.ai/docs/lume/guide/getting-started/installation)

* * *

## 2) إنشاء الجهاز الظاهري لنظام macOS

```bash
lume create openclaw --os macos --ipsw latest
```

سيقوم هذا بتنزيل نظام macOS وإنشاء الجهاز الظاهري. ستفتح نافذة VNC تلقائيًا. ملاحظة: قد يستغرق التنزيل بعض الوقت حسب سرعة اتصالك.

* * *

## 3) أكمل مساعد الإعداد

في نافذة VNC:

1.  اختر اللغة والمنطقة
2.  تخطى Apple ID (أو سجل الدخول إذا كنت تريد iMessage لاحقًا)
3.  أنشئ حساب مستخدم (تذكر اسم المستخدم وكلمة المرور)
4.  تخطى جميع الميزات الاختيارية

بعد اكتمال الإعداد، قم بتمكين SSH:

1.  افتح "إعدادات النظام" → "عام" → "المشاركة"
2.  فعّل "تسجيل الدخول عن بعد"

* * *

## 4) احصل على عنوان IP للجهاز الظاهري

```bash
lume get openclaw
```

ابحث عن عنوان IP (عادةً `192.168.64.x`).

* * *

## 5) الدخول عبر SSH إلى الجهاز الظاهري

```bash
ssh youruser@192.168.64.X
```

استبدل `youruser` باسم الحساب الذي أنشأته، و IP بعنوان IP الخاص بجهازك الظاهري.

* * *

## 6) تثبيت OpenClaw

داخل الجهاز الظاهري:

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

اتبع التعليمات أثناء عملية الإعداد لتكوين موفر النموذج الخاص بك (Anthropic، OpenAI، إلخ).

* * *

## 7) تكوين القنوات

قم بتحرير ملف التكوين:

```bash
nano ~/.openclaw/openclaw.json
```

أضف قنواتك:

```json
{
  "channels": {
    "whatsapp": {
      "dmPolicy": "allowlist",
      "allowFrom": ["+15551234567"]
    },
    "telegram": {
      "botToken": "YOUR_BOT_TOKEN"
    }
  }
}
```

ثم سجل الدخول إلى WhatsApp (امسح رمز QR):

```bash
openclaw channels login
```

* * *

## 8) تشغيل الجهاز الظاهري بدون واجهة رسومية

أوقف الجهاز الظاهري وأعد تشغيله بدون عرض:

```bash
lume stop openclaw
lume run openclaw --no-display
```

سيعمل الجهاز الظاهري في الخلفية. سيبقى برنامج OpenClaw الخفي (daemon) البوابة قيد التشغيل. للتحقق من الحالة:

```bash
ssh youruser@192.168.64.X "openclaw status"
```

* * *

## مكافأة: تكامل iMessage

هذه هي الميزة الفريدة للتشغيل على نظام macOS. استخدم [BlueBubbles](https://bluebubbles.app) لإضافة iMessage إلى OpenClaw. داخل الجهاز الظاهري:

1.  حمل BlueBubbles من bluebubbles.app
2.  سجل الدخول باستخدام Apple ID الخاص بك
3.  فعّل Web API وحدد كلمة مرور
4.  وجه webhooks الخاص بـ BlueBubbles إلى بوابة (gateway) الخاص بك (مثال: `https://your-gateway-host:3000/bluebubbles-webhook?password=`)

أضف إلى تكوين OpenClaw الخاص بك:

```json
{
  "channels": {
    "bluebubbles": {
      "serverUrl": "http://localhost:1234",
      "password": "your-api-password",
      "webhookPath": "/bluebubbles-webhook"
    }
  }
}
```

أعد تشغيل البوابة. الآن يمكن للوكيل الخاص بك إرسال واستقبال رسائل iMessage. تفاصيل الإعداد الكاملة: [قناة BlueBubbles](../channels/bluebubbles.md)

* * *

## احفظ صورة ذهبية (حالة نظيفة)

قبل التخصيص أكثر، التقط لقطة للحالة النظيفة:

```bash
lume stop openclaw
lume clone openclaw openclaw-golden
```

أعد التعيين في أي وقت:

```bash
lume stop openclaw && lume delete openclaw
lume clone openclaw-golden openclaw
lume run openclaw --no-display
```

* * *

## التشغيل 24/7

احتفظ بالجهاز الظاهري قيد التشغيل عن طريق:

-   إبقاء جهاز Mac الخاص بك موصولًا بالطاقة
-   تعطيل وضع السكون في "إعدادات النظام" → "توفير الطاقة"
-   استخدام `caffeinate` إذا لزم الأمر

للحصول على تشغيل دائم حقيقي، فكر في استخدام جهاز Mac mini مخصص أو خادم VPS صغير. راجع [استضافة VPS](../vps.md).

* * *

## استكشاف الأخطاء وإصلاحها

| المشكلة | الحل |
| --- | --- |
| لا يمكن الدخول عبر SSH إلى الجهاز الظاهري | تحقق من تمكين "تسجيل الدخول عن بعد" في "إعدادات النظام" للجهاز الظاهري |
| عنوان IP للجهاز الظاهري لا يظهر | انتظر حتى يتم إقلاع الجهاز الظاهري بالكامل، ثم نفذ `lume get openclaw` مرة أخرى |
| أمر Lume غير موجود | أضف `~/.local/bin` إلى PATH الخاص بك |
| رمز QR لـ WhatsApp لا يتم مسحه | تأكد من أنك سجلت الدخول إلى الجهاز الظاهري (وليس الجهاز المضيف) عند تنفيذ `openclaw channels login` |

* * *

## وثائق ذات صلة

-   [استضافة VPS](../vps.md)
-   [العقد](../nodes.md)
-   [بوابة عن بعد](../gateway/remote.md)
-   [قناة BlueBubbles](../channels/bluebubbles.md)
-   [بداية سريعة لـ Lume](https://cua.ai/docs/lume/guide/getting-started/quickstart)
-   [مرجع أوامر Lume CLI](https://cua.ai/docs/lume/reference/cli-reference)
-   [إعداد جهاز ظاهري بدون مراقبة](https://cua.ai/docs/lume/guide/fundamentals/unattended-setup) (متقدم)
-   [عزل Docker](./docker.md) (نهج بديل للعزل)

[GCP](./gcp.md)[exe.dev](./exe-dev.md)