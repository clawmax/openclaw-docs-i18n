title: "دليل تثبيت OpenClaw باستخدام بيئة تشغيل Bun التجريبية"
description: "تعلم كيفية تثبيت وتشغيل OpenClaw باستخدام Bun للتطوير المحلي. يغطي التثبيت، أوامر البناء، نصوص دورة الحياة، والمحاذير المهمة."
keywords: ["bun", "تثبيت openclaw", "بيئة تشغيل تجريبية", "bun install", "نصوص دورة حياة bun", "بديل pnpm", "بيئة تشغيل typescript"]
---

  طرق تثبيت أخرى

  
# Bun (تجريبي)

الهدف: تشغيل هذا المستودع باستخدام **Bun** (اختياري، غير موصى به لـ WhatsApp/Telegram) دون الابتعاد عن سير عمل pnpm. ⚠️ **غير موصى به لبيئة تشغيل Gateway** (أخطاء WhatsApp/Telegram). استخدم Node للإنتاج.

## الحالة

-   Bun هو بيئة تشغيل محلية اختيارية لتشغيل TypeScript مباشرة (`bun run …`, `bun --watch …`).
-   `pnpm` هو الافتراضي لعمليات البناء ويظل مدعومًا بالكامل (ويستخدمه بعض أدوات التوثيق).
-   لا يمكن لـ Bun استخدام `pnpm-lock.yaml` وسيتجاهله.

## التثبيت

الافتراضي:

```bash
bun install
```

ملاحظة: `bun.lock`/`bun.lockb` يتم تجاهلهما في git، لذلك لا يوجد تغيير في المستودع بأي حال. إذا كنت تريد *عدم كتابة أي ملف قفل*:

```bash
bun install --no-save
```

## البناء / الاختبار (Bun)

```bash
bun run build
bun run vitest run
```

## نصوص دورة حياة Bun (محظورة افتراضيًا)

قد يحظر Bun نصوص دورة حياة التبعيات ما لم يتم الوثوق بها صراحةً (`bun pm untrusted` / `bun pm trust`). بالنسبة لهذا المستودع، النصوص المحظورة شائعيًا غير مطلوبة:

-   `@whiskeysockets/baileys` `preinstall`: يتحقق من إصدار Node الرئيسي >= 20 (نحن نعمل على Node 22+).
-   `protobufjs` `postinstall`: يصدر تحذيرات حول مخططات الإصدار غير المتوافقة (لا توجد مخرجات بناء).

إذا واجهت مشكلة تشغيل حقيقية تتطلب هذه النصوص، ثق بها صراحةً:

```bash
bun pm trust @whiskeysockets/baileys protobufjs
```

## محاذير

-   بعض النصوص لا تزال تذكر pnpm بشكل ثابت (مثل `docs:build`, `ui:*`, `protocol:check`). شغلها عبر pnpm حاليًا.

[Ansible](./ansible.md)[التحديث](./updating.md)