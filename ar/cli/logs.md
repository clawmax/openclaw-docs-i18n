title: "استخدام وأمثلة أمر سجلات OpenClaw CLI"
description: "تعلم كيفية استخدام أمر سجلات openclaw logs في CLI لتعقب سجلات ملفات البوابة عبر RPC. شاهد أمثلة للمتابعة، والإخراج بصيغة JSON، والمنطقة الزمنية المحلية."
keywords: ["سجلات openclaw", "سجلات cli", "سجلات البوابة", "تعقب السجلات", "تسجيل RPC", "سجلات سطر الأوامر", "مراقبة السجلات"]
---

  أوامر CLI

  
# logs

تعقب سجلات ملفات البوابة عبر RPC (يعمل في الوضع البعيد). متعلق:

-   نظرة عامة على التسجيل: [التسجيل](../logging.md)

## أمثلة

```bash
openclaw logs
openclaw logs --follow
openclaw logs --json
openclaw logs --limit 500
openclaw logs --local-time
openclaw logs --follow --local-time
```

استخدم `--local-time` لعرض الطوابع الزمنية وفقًا لمنطقتك الزمنية المحلية.

[hooks](./hooks.md)[memory](./memory.md)

---