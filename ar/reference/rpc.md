title: "محولات RPC وAPI في OpenClaw لدمج واجهات سطر الأوامر الخارجية"
description: "تعرف على كيفية دمج OpenClaw لواجهات سطر الأوامر الخارجية باستخدام محولات JSON-RPC، بما في ذلك نمط خادم HTTP وعملية الطفل stdio لقنوات مثل Signal وiMessage."
keywords: ["json-rpc", "محول rpc", "signal-cli", "دمج iMessage", "بوابة api", "واجهة سطر أوامر خارجية", "openclaw rpc", "عملية stdio"]
---

  RPC وAPI

  
# محولات RPC

يدمج OpenClaw واجهات سطر الأوامر الخارجية عبر JSON-RPC. يتم استخدام نمطين اليوم.

## النمط أ: خادم HTTP (signal-cli)

-   يعمل `signal-cli` كخادم مع JSON-RPC عبر HTTP.
-   تيار الأحداث هو SSE (`/api/v1/events`).
-   فحص الصحة: `/api/v1/check`.
-   يدير OpenClaw دورة الحياة عندما يكون `channels.signal.autoStart=true`.

راجع [Signal](../channels/signal.md) للإعداد ونقاط النهاية.

## النمط ب: عملية الطفل stdio (قديم: imsg)

> **ملاحظة:** للإعدادات الجديدة لـ iMessage، استخدم [BlueBubbles](../channels/bluebubbles.md) بدلاً من ذلك.

-   يقوم OpenClaw بتشغيل `imsg rpc` كعملية فرعية (دمج iMessage القديم).
-   يتم إرسال JSON-RPC سطراً بسطر عبر stdin/stdout (كائن JSON واحد لكل سطر).
-   لا حاجة لمنفذ TCP أو خادم.

الطرق الأساسية المستخدمة:

-   `watch.subscribe` → إشعارات (`method: "message"`)
-   `watch.unsubscribe`
-   `send`
-   `chats.list` (فحص/تشخيص)

راجع [iMessage](../channels/imessage.md) للإعداد القديم والعناوين (يفضل `chat_id`).

## إرشادات المحول

-   تملك البوابة العملية (البدء/الإيقاف مرتبط بدورة حياة المزود).
-   حافظ على مرونة عملاء RPC: مهلات زمنية، إعادة تشغيل عند الخروج.
-   افضل المعرفات الثابتة (مثل `chat_id`) على السلاسل النصية المعروضة.

[webhooks](../cli/webhooks.md)[قاعدة بيانات نماذج الأجهزة](./device-models.md)

---