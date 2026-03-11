

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