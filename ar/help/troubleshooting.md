title: "دليل استكشاف أخطاء OpenClaw وأوامر البدء السريع"
description: "تشخيص وإصلاح مشاكل OpenClaw بسرعة. اتبع سلالم الأوامر خطوة بخطوة لاستكشاف أخطاء الحالة، والبوابة، والقنوات، والسجلات، والمهام المجدولة، والعقد، والمتصفح."
keywords: ["استكشاف أخطاء openclaw", "حالة openclaw", "فحص البوابة", "حالة القنوات", "سجلات openclaw", "openclaw doctor", "دليل استكشاف الأخطاء", "تشخيص الأخطاء"]
---

  المساعدة

  
# استكشاف الأخطاء وإصلاحها

إذا كان لديك دقيقتان فقط، استخدم هذه الصفحة كبوابة أولية للفرز.

## أول 60 ثانية

شغّل هذا السلم بالضبط بالترتيب:

```bash
openclaw status
openclaw status --all
openclaw gateway probe
openclaw gateway status
openclaw doctor
openclaw channels status --probe
openclaw logs --follow
```

المخرجات الجيدة في سطر واحد:

-   `openclaw status` → تُظهر القنوات المُهيأة ولا تظهر أخطاء مصادقة واضحة.
-   `openclaw status --all` → التقرير الكامل موجود وقابل للمشاركة.
-   `openclaw gateway probe` → هدف البوابة المتوقع يمكن الوصول إليه.
-   `openclaw gateway status` → `Runtime: running` و `RPC probe: ok`.
-   `openclaw doctor` → لا توجد أخطاء في التكوين/الخدمة تمنع التشغيل.
-   `openclaw channels status --probe` → القنوات تُبلغ بـ `connected` أو `ready`.
-   `openclaw logs --follow` → نشاط مستمر، لا توجد أخطاء فادحة متكررة.

## Anthropic سياق طويل 429

إذا رأيت: `HTTP 429: rate_limit_error: Extra usage is required for long context requests`، انتقل إلى [/gateway/troubleshooting#anthropic-429-extra-usage-required-for-long-context](../gateway/troubleshooting.md#anthropic-429-extra-usage-required-for-long-context).

## فشل تثبيت الإضافة بسبب افتقار openclaw extensions

إذا فشل التثبيت مع `package.json missing openclaw.extensions`، فإن حزمة الإضافة تستخدم شكلًا قديمًا لم يعد OpenClaw يقبله. الإصلاح في حزمة الإضافة:

1.  أضف `openclaw.extensions` إلى `package.json`.
2.  وجّه المداخل إلى ملفات وقت التشغيل المبنية (عادةً `./dist/index.js`).
3.  أعد نشر الإضافة وشغّل `openclaw plugins install <npm-spec>` مرة أخرى.

مثال:

```json
{
  "name": "@openclaw/my-plugin",
  "version": "1.2.3",
  "openclaw": {
    "extensions": ["./dist/index.js"]
  }
}
```

المرجع: [/tools/plugin#distribution-npm](../tools/plugin.md#distribution-npm)

## شجرة القرار

```bash
openclaw status
openclaw gateway status
openclaw channels status --probe
openclaw pairing list --channel <channel> [--account <id>]
openclaw logs --follow
```

المخرجات الجيدة تبدو كالتالي:

-   `Runtime: running`
-   `RPC probe: ok`
-   قناتك تظهر متصلة/جاهزة في `channels status --probe`
-   المرسل يظهر معتمدًا (أو أن سياسة الرسائل المباشرة مفتوحة/في القائمة المسموحة)

التوقيعات الشائعة في السجلات:

-   `drop guild message (mention required` → حظر بوابات الإشارة الرسالة في Discord.
-   `pairing request` → المرسل غير معتمد وينتظر موافقة الاقتران عبر الرسائل المباشرة.
-   `blocked` / `allowlist` في سجلات القناة → المرسل، الغرفة، أو المجموعة مفلترة.

الصفحات المتعمقة:

-   [/gateway/troubleshooting#no-replies](../gateway/troubleshooting.md#no-replies)
-   [/channels/troubleshooting](../channels/troubleshooting.md)
-   [/channels/pairing](../channels/pairing.md)

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

المخرجات الجيدة تبدو كالتالي:

-   `Dashboard: http://...` تظهر في `openclaw gateway status`
-   `RPC probe: ok`
-   لا توجد حلقة مصادقة في السجلات

التوقيعات الشائعة في السجلات:

-   `device identity required` → سياق HTTP/غير آمن لا يمكنه إكمال مصادقة الجهاز.
-   `unauthorized` / حلقة إعادة اتصال → رمز/كلمة مرور خاطئة أو عدم تطابق وضع المصادقة.
-   `gateway connect failed:` → واجهة المستخدم تستهدف عنوان URL/منفذ خاطئ أو بوابة غير قابلة للوصول.

الصفحات المتعمقة:

-   [/gateway/troubleshooting#dashboard-control-ui-connectivity](../gateway/troubleshooting.md#dashboard-control-ui-connectivity)
-   [/web/control-ui](../web/control-ui.md)
-   [/gateway/authentication](../gateway/authentication.md)

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

المخرجات الجيدة تبدو كالتالي:

-   `Service: ... (loaded)`
-   `Runtime: running`
-   `RPC probe: ok`

التوقيعات الشائعة في السجلات:

-   `Gateway start blocked: set gateway.mode=local` → وضع البوابة غير مضبوط/بعيد.
-   `refusing to bind gateway ... without auth` → ربط غير loopback بدون رمز/كلمة مرور.
-   `another gateway instance is already listening` أو `EADDRINUSE` → المنفذ مشغول مسبقًا.

الصفحات المتعمقة:

-   [/gateway/troubleshooting#gateway-service-not-running](../gateway/troubleshooting.md#gateway-service-not-running)
-   [/gateway/background-process](../gateway/background-process.md)
-   [/gateway/configuration](../gateway/configuration.md)

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

المخرجات الجيدة تبدو كالتالي:

-   ناقل القناة متصل.
-   فحوصات الاقتران/القائمة المسموحة تمر بنجاح.
-   يتم اكتشاف الإشارات حيث تكون مطلوبة.

التوقيعات الشائعة في السجلات:

-   `mention required` → حظرت بوابات الإشارة الجماعية المعالجة.
-   `pairing` / `pending` → مرسل الرسائل المباشرة غير معتمد بعد.
-   `not_in_channel`, `missing_scope`, `Forbidden`, `401/403` → مشكلة في رمز إذن القناة.

الصفحات المتعمقة:

-   [/gateway/troubleshooting#channel-connected-messages-not-flowing](../gateway/troubleshooting.md#channel-connected-messages-not-flowing)
-   [/channels/troubleshooting](../channels/troubleshooting.md)

```bash
openclaw status
openclaw gateway status
openclaw cron status
openclaw cron list
openclaw cron runs --id <jobId> --limit 20
openclaw logs --follow
```

المخرجات الجيدة تبدو كالتالي:

-   `cron.status` تُظهر مفعلة مع استيقاظ تالي.
-   `cron runs` تُظهر مدخلات `ok` حديثة.
-   نبضات الحياة مفعلة وليست خارج ساعات النشاط.

التوقيعات الشائعة في السجلات:

-   `cron: scheduler disabled; jobs will not run automatically` → المهام المجدولة معطلة.
-   `heartbeat skipped` مع `reason=quiet-hours` → خارج ساعات النشاط المُهيأة.
-   `requests-in-flight` → المسار الرئيسي مشغول؛ تم تأجيل استيقاظ نبضات الحياة.
-   `unknown accountId` → هدف تسليم نبضات الحياة (الحساب) غير موجود.

الصفحات المتعمقة:

-   [/gateway/troubleshooting#cron-and-heartbeat-delivery](../gateway/troubleshooting.md#cron-and-heartbeat-delivery)
-   [/automation/troubleshooting](../automation/troubleshooting.md)
-   [/gateway/heartbeat](../gateway/heartbeat.md)

```bash
openclaw status
openclaw gateway status
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw logs --follow
```

المخرجات الجيدة تبدو كالتالي:

-   العقدة مدرجة كمتصلة ومقترنة لدور `node`.
-   توجد القدرة للأمر الذي تستدعيه.
-   حالة الإذن مُمنحة للأداة.

التوقيعات الشائعة في السجلات:

-   `NODE_BACKGROUND_UNAVAILABLE` → أحضر تطبيق العقدة إلى المقدمة.
-   `*_PERMISSION_REQUIRED` → إذن نظام التشغيل مرفوض/مفقود.
-   `SYSTEM_RUN_DENIED: approval required` → موافقة التنفيذ معلقة.
-   `SYSTEM_RUN_DENIED: allowlist miss` → الأمر غير موجود في القائمة المسموحة للتنفيذ.

الصفحات المتعمقة:

-   [/gateway/troubleshooting#node-paired-tool-fails](../gateway/troubleshooting.md#node-paired-tool-fails)
-   [/nodes/troubleshooting](../nodes/troubleshooting.md)
-   [/tools/exec-approvals](../tools/exec-approvals.md)

```bash
openclaw status
openclaw gateway status
openclaw browser status
openclaw logs --follow
openclaw doctor
```

المخرجات الجيدة تبدو كالتالي:

-   حالة المتصفح تُظهر `running: true` ومتصفح/ملف تعريف محدد.
-   ملف تعريف `openclaw` يبدأ أو أن ناقل امتداد `chrome` لديه لسان متصل.

التوقيعات الشائعة في السجلات:

-   `Failed to start Chrome CDP on port` → فشل تشغيل المتصفح المحلي.
-   `browser.executablePath not found` → مسار الملف الثنائي المُهيأ خاطئ.
-   `Chrome extension relay is running, but no tab is connected` → الامتداد غير متصل.
-   `Browser attachOnly is enabled ... not reachable` → ملف تعريف الإرفاق فقط ليس لديه هدف CDP نشط.

الصفحات المتعمقة:

-   [/gateway/troubleshooting#browser-tool-fails](../gateway/troubleshooting.md#browser-tool-fails)
-   [/tools/browser-linux-troubleshooting](../tools/browser-linux-troubleshooting.md)
-   [/tools/chrome-extension](../tools/chrome-extension.md)

[المساعدة](../help.md)[الأسئلة الشائعة](./faq.md)