

  الأتمتة

  
# استكشاف أخطاء الأتمتة

استخدم هذه الصفحة لمشكلات المجدول والتسليم (`cron` + `heartbeat`).

## سلم الأوامر

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

ثم قم بتشغيل فحوصات الأتمتة:

```bash
openclaw cron status
openclaw cron list
openclaw system heartbeat last
```

## Cron لا يعمل

```bash
openclaw cron status
openclaw cron list
openclaw cron runs --id <jobId> --limit 20
openclaw logs --follow
```

المخرجات الجيدة تبدو كالتالي:

-   `cron status` يبلغ عن تمكين و `nextWakeAtMs` مستقبلي.
-   المهمة مفعلة ولها جدول/نطاق زمني صالح.
-   `cron runs` تظهر `ok` أو سبب تخطي صريح.

المؤشرات الشائعة:

-   `cron: scheduler disabled; jobs will not run automatically` → cron معطل في الإعدادات/البيئة.
-   `cron: timer tick failed` → انهارت نبضة المجدول؛ افحص سياق المكدس/السجلات المحيط.
-   `reason: not-due` في مخرجات التشغيل → تم استدعاء تشغيل يدوي بدون `--force` والمهمة ليست مستحقة بعد.

## Cron عمل ولكن لم يتم التسليم

```bash
openclaw cron runs --id <jobId> --limit 20
openclaw cron list
openclaw channels status --probe
openclaw logs --follow
```

المخرجات الجيدة تبدو كالتالي:

-   حالة التشغيل هي `ok`.
-   وضع/هدف التسليم مضبوط للمهام المعزولة.
-   فحص القناة يبلغ عن اتصال القناة المستهدفة.

المؤشرات الشائعة:

-   نجح التشغيل ولكن وضع التسليم هو `none` → لا يُتوقع رسالة خارجية.
-   هدف التسليم مفقود/غير صالح (`channel`/`to`) → قد ينجح التشغيل داخليًا ولكنه يتخطى الإرسال الخارجي.
-   أخطاء مصادقة القناة (`unauthorized`, `missing_scope`, `Forbidden`) → تم حظر التسليم بسبب أذونات/بيانات اعتماد القناة.

## Heartbeat تم كبته أو تخطيه

```bash
openclaw system heartbeat last
openclaw logs --follow
openclaw config get agents.defaults.heartbeat
openclaw channels status --probe
```

المخرجات الجيدة تبدو كالتالي:

-   Heartbeat مفعل بفترة زمنية غير صفرية.
-   نتيجة آخر heartbeat هي `ran` (أو سبب التخطي مفهوم).

المؤشرات الشائعة:

-   `heartbeat skipped` مع `reason=quiet-hours` → خارج `activeHours`.
-   `requests-in-flight` → المسار الرئيسي مشغول؛ تم تأجيل heartbeat.
-   `empty-heartbeat-file` → تم تخطي فترة heartbeat لأن ملف `HEARTBEAT.md` لا يحتوي على محتوى قابل للتنفيذ ولم يتم قائمة أي حدث cron موسوم.
-   `alerts-disabled` → إعدادات الرؤية تمنع رسائل heartbeat الصادرة.

## مفاجآت النطاق الزمني و activeHours

```bash
openclaw config get agents.defaults.heartbeat.activeHours
openclaw config get agents.defaults.heartbeat.activeHours.timezone
openclaw config get agents.defaults.userTimezone || echo "agents.defaults.userTimezone not set"
openclaw cron list
openclaw logs --follow
```

قواعد سريعة:

-   `Config path not found: agents.defaults.userTimezone` تعني أن المفتاح غير مضبوط؛ يتراجع heartbeat إلى نطاق زمني المضيف (أو `activeHours.timezone` إذا كان مضبوطًا).
-   Cron بدون `--tz` يستخدم النطاق الزمني لبوابة المضيف.
-   `activeHours` الخاص بـ heartbeat يستخدم دقة النطاق الزمني المحدد (`user`, `local`, أو IANA tz صريح).
-   الطوابع الزمنية ISO بدون نطاق زمني تُعامل كـ UTC لجدول cron `at`.

المؤشرات الشائعة:

-   تعمل المهام في وقت الساعة الخاطئ بعد تغييرات النطاق الزمني للمضيف.
-   يتم دائمًا تخطي heartbeat خلال وقت النهار لأن `activeHours.timezone` خاطئ.

ذات صلة:

-   [/automation/cron-jobs](./cron-jobs.md)
-   [/gateway/heartbeat](../gateway/heartbeat.md)
-   [/automation/cron-vs-heartbeat](./cron-vs-heartbeat.md)
-   [/concepts/timezone](../concepts/timezone.md)

[Cron مقابل Heartbeat](./cron-vs-heartbeat.md)[Webhooks](./webhook.md)