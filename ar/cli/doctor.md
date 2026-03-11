

  أوامر CLI

  
# doctor

فحوصات صحية + إصلاحات سريعة للبوابة والقنوات. ذات صلة:

-   استكشاف الأخطاء وإصلاحها: [استكشاف الأخطاء وإصلاحها](../gateway/troubleshooting.md)
-   تدقيق الأمان: [الأمان](../gateway/security.md)

## أمثلة

```bash
openclaw doctor
openclaw doctor --repair
openclaw doctor --deep
```

ملاحظات:

-   المطالبات التفاعلية (مثل إصلاحات keychain/OAuth) تعمل فقط عندما يكون stdin هو TTY ولم يتم تعيين `--non-interactive`. عمليات التشغيل بدون واجهة مستخدم (cron، Telegram، بدون طرفية) ستتخطى المطالبات.
-   `--fix` (اسم مستعار لـ `--repair`) يكتب نسخة احتياطية إلى `~/.openclaw/openclaw.json.bak` ويتجاهل مفاتيح التكوين غير المعروفة، مع سرد كل إزالة.
-   فحوصات سلامة الحالة تكتشف الآن ملفات النصوص اليتيمة في دليل الجلسات ويمكنها أرشفتها كـ `.deleted.` لاستعادة المساحة بأمان.
-   يتضمن Doctor فحص جاهزية بحث الذاكرة ويمكنه التوصية بـ `openclaw configure --section model` عندما تكون بيانات اعتماد التضمين مفقودة.
-   إذا تم تمكين وضع الحماية (sandbox) ولكن Docker غير متاح، يبلغ Doctor عن تحذير عالي الإشارة مع علاج (`install Docker` أو `openclaw config set agents.defaults.sandbox.mode off`).

## macOS: تجاوزات بيئة launchctl

إذا سبق وقمت بتشغيل `launchctl setenv OPENCLAW_GATEWAY_TOKEN ...` (أو `...PASSWORD`)، فإن تلك القيمة تتجاوز ملف التكوين الخاص بك ويمكن أن تسبب أخطاء مستمرة "غير مصرح".

```bash
launchctl getenv OPENCLAW_GATEWAY_TOKEN
launchctl getenv OPENCLAW_GATEWAY_PASSWORD

launchctl unsetenv OPENCLAW_GATEWAY_TOKEN
launchctl unsetenv OPENCLAW_GATEWAY_PASSWORD
```

[الوثائق](./docs.md)[البوابة](./gateway.md)