title: "دليل اختبار OpenClaw AI: الأوامر، المعايير، و Docker"
description: "تعلم كيفية تشغيل اختبارات OpenClaw AI، بما في ذلك اختبارات الوحدة، التكامل، والاختبارات الحية. احصل على أوامر للتغطية، المعايير، والاختبار الشامل (E2E) المعتمد على Docker."
keywords: ["اختبار openclaw", "أوامر vitest", "اختبارات docker e2e", "تغطية الاختبار", "معايير cli", "تكامل gateway", "pnpm test", "معيار زمن استجابة النموذج"]
---

  ملاحظات الإصدار

  
# الاختبارات

-   مجموعة الاختبارات الكاملة (مجموعات، حية، Docker): [الاختبار](../help/testing.md)
-   `pnpm test:force`: ينهي أي عملية gateway عالقة تحتجز منفذ التحكم الافتراضي، ثم يشغل مجموعة Vitest الكاملة بمنفذ gateway معزول حتى لا تتعارض اختبارات الخادم مع نسخة قيد التشغيل. استخدم هذا عندما تترك عملية gateway سابقة المنفذ 18789 مشغولاً.
-   `pnpm test:coverage`: يشغل مجموعة اختبارات الوحدة مع تغطية V8 (عبر `vitest.unit.config.ts`). الحدود الدنيا العالمية هي 70% للأسطر/الفروع/الدوال/العبارات. تستثني التغطية نقاط الدخول كثيفة التكامل (ربط CLI، جسور gateway/telegram، خادم الويب الثابت) للحفاظ على الهدف مركزاً على المنطق القابل لاختبار الوحدة.
-   `pnpm test` على Node 24+: يقوم OpenClaw تلقائياً بتعطيل `vmForks` في Vitest ويستخدم `forks` لتجنب `ERR_VM_MODULE_LINK_FAILURE` / `module is already linked`. يمكنك فرض السلوك باستخدام `OPENCLAW_TEST_VM_FORKS=0|1`.
-   `pnpm test`: يشغل مسار الوحدة الأساسي السريع افتراضياً للحصول على ملاحظات محلية سريعة.
-   `pnpm test:channels`: يشغل مجموعات الاختبارات الكثيفة بالقنوات.
-   `pnpm test:extensions`: يشغل مجموعات اختبارات الامتدادات/الإضافات.
-   تكامل Gateway: اختياري عبر `OPENCLAW_TEST_INCLUDE_GATEWAY=1 pnpm test` أو `pnpm test:gateway`.
-   `pnpm test:e2e`: يشغل اختبارات التدخين الشاملة (E2E) للـ gateway (اقتران متعدد النسخ لـ WS/HTTP/node). الافتراضي هو `vmForks` + عمال تكيفيين في `vitest.e2e.config.ts`؛ اضبط باستخدام `OPENCLAW_E2E_WORKERS=` وعيّن `OPENCLAW_E2E_VERBOSE=1` للسجلات التفصيلية.
-   `pnpm test:live`: يشغل اختبارات المزود الحية (minimax/zai). يتطلب مفاتيح API و `LIVE=1` (أو `*_LIVE_TEST=1` خاص بالمزود) لإلغاء التخطي.

## بوابة طلب السحب (PR) المحلية

للتحقق من بوابة طلب السحب/الهبوط المحلي، شغّل:

-   `pnpm check`
-   `pnpm build`
-   `pnpm test`
-   `pnpm check:docs`

إذا تعثر `pnpm test` على مضيف محمّل، أعد تشغيله مرة واحدة قبل اعتباره تراجعاً، ثم عزله باستخدام `pnpm vitest run <path/to/test>`. للمضيفين محدودي الذاكرة، استخدم:

-   `OPENCLAW_TEST_PROFILE=low OPENCLAW_TEST_SERIAL_GATEWAY=1 pnpm test`

## معيار زمن استجابة النموذج (مفاتيح محلية)

النص البرمجي: [`scripts/bench-model.ts`](https://github.com/openclaw/openclaw/blob/main/scripts/bench-model.ts) الاستخدام:

-   `source ~/.profile && pnpm tsx scripts/bench-model.ts --runs 10`
-   متغيرات بيئة اختيارية: `MINIMAX_API_KEY`, `MINIMAX_BASE_URL`, `MINIMAX_MODEL`, `ANTHROPIC_API_KEY`
-   المطالبة الافتراضية: "رد بكلمة واحدة: ok. بدون علامات ترقيم أو نص إضافي."

آخر تشغيل (2025-12-31، 20 تشغيلاً):

-   minimax الوسيط 1279ms (الحد الأدنى 1114، الحد الأقصى 2431)
-   opus الوسيط 2454ms (الحد الأدنى 1224، الحد الأقصى 3170)

## معيار بدء تشغيل CLI

النص البرمجي: [`scripts/bench-cli-startup.ts`](https://github.com/openclaw/openclaw/blob/main/scripts/bench-cli-startup.ts) الاستخدام:

-   `pnpm tsx scripts/bench-cli-startup.ts`
-   `pnpm tsx scripts/bench-cli-startup.ts --runs 12`
-   `pnpm tsx scripts/bench-cli-startup.ts --entry dist/entry.js --timeout-ms 45000`

يقيس هذا الأداء لهذه الأوامر:

-   `--version`
-   `--help`
-   `health --json`
-   `status --json`
-   `status`

يتضمن الناتج المتوسط، p50، p95، الحد الأدنى/الأقصى، وتوزيع رمز الخروج/الإشارة لكل أمر.

## الاختبار الشامل (E2E) للانضمام (Docker)

Docker اختياري؛ هذا مطلوب فقط لاختبارات التدخين المعتمدة على الحاويات للانضمام. تدفق البدء البارد الكامل في حاوية Linux نظيفة:

```
scripts/e2e/onboard-docker.sh
```

يقود هذا النص البرمجي المعالج التفاعلي عبر طرفية زائفة، يتحقق من ملفات التكوين/مساحة العمل/الجلسة، ثم يبدأ الـ gateway ويشغل `openclaw health`.

## اختبار تدخين استيراد QR (Docker)

يضمن تحميل `qrcode-terminal` تحت Node 22+ في Docker:

```bash
pnpm test:docker:qr
```

[قائمة التحقق للإصدار](./RELEASING.md)[تكامل Kilo gateway](../design/kilo-gateway-integration.md)