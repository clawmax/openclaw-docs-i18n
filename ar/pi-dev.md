

  إعداد المطور

  
# سير عمل تطوير Pi

يُلخص هذا الدليل سير عمل سليم للعمل على دمج Pi في OpenClaw.

## التحقق من الأنواع والتدقيق

-   التحقق من الأنواع والبناء: `pnpm build`
-   التدقيق: `pnpm lint`
-   التحقق من التنسيق: `pnpm format`
-   البوابة الكاملة قبل الدفع: `pnpm lint && pnpm build && pnpm test`

## تشغيل اختبارات Pi

شغّل مجموعة الاختبارات المُركزة على Pi مباشرة باستخدام Vitest:

```bash
pnpm test -- \
  "src/agents/pi-*.test.ts" \
  "src/agents/pi-embedded-*.test.ts" \
  "src/agents/pi-tools*.test.ts" \
  "src/agents/pi-settings.test.ts" \
  "src/agents/pi-tool-definition-adapter*.test.ts" \
  "src/agents/pi-extensions/**/*.test.ts"
```

لتضمين تمرين مزود الاختبار المباشر:

```
OPENCLAW_LIVE_TEST=1 pnpm test -- src/agents/pi-embedded-runner-extraparams.live.test.ts
```

يغطي هذا مجموعات اختبارات Pi الرئيسية:

-   `src/agents/pi-*.test.ts`
-   `src/agents/pi-embedded-*.test.ts`
-   `src/agents/pi-tools*.test.ts`
-   `src/agents/pi-settings.test.ts`
-   `src/agents/pi-tool-definition-adapter.test.ts`
-   `src/agents/pi-extensions/*.test.ts`

## الاختبار اليدوي

سير العمل الموصى به:

-   تشغيل البوابة في وضع التطوير:
    -   `pnpm gateway:dev`
-   تشغيل الوكيل مباشرة:
    -   `pnpm openclaw agent --message "Hello" --thinking low`
-   استخدام واجهة المستخدم النصية (TUI) لتصحيح الأخطاء التفاعلي:
    -   `pnpm tui`

لاختبار سلوك استدعاء الأداة، اطلب إجراء `read` أو `exec` لترى دفق الأداة ومعالجة الحمولة.

## إعادة تعيين الحالة من الصفر

توجد الحالة تحت دليل حالة OpenClaw. الافتراضي هو `~/.openclaw`. إذا تم تعيين `OPENCLAW_STATE_DIR`، فاستخدم ذلك الدليل بدلاً من ذلك. لإعادة تعيين كل شيء:

-   `openclaw.json` للإعدادات
-   `credentials/` لملفات تعريف المصادقة والرموز
-   `agents//sessions/` لتاريخ جلسات الوكيل
-   `agents//sessions.json` لفهرس الجلسات
-   `sessions/` إذا كانت المسارات القديمة موجودة
-   `workspace/` إذا أردت مساحة عمل فارغة

إذا أردت إعادة تعيين الجلسات فقط، احذف `agents//sessions/` و `agents//sessions.json` لذلك الوكيل. احتفظ بـ `credentials/` إذا كنت لا تريد إعادة المصادقة.

## المراجع

-   [https://docs.openclaw.ai/testing](https://docs.openclaw.ai/testing)
-   [https://docs.openclaw.ai/start/getting-started](https://docs.openclaw.ai/start/getting-started)

[الإعداد](./start/setup.md)[خط أنابيب التكامل المستمر](./ci.md)

---