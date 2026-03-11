

  أوامر CLI

  
# pairing

الموافقة على طلبات إقران الرسائل المباشرة أو فحصها (للقنوات التي تدعم الإقران). ذات صلة:

-   سير عمل الإقران: [الإقران](../channels/pairing.md)

## الأوامر

```bash
openclaw pairing list telegram
openclaw pairing list --channel telegram --account work
openclaw pairing list telegram --json

openclaw pairing approve telegram <code>
openclaw pairing approve --channel telegram --account work <code> --notify
```

## ملاحظات

-   إدخال القناة: قم بتمريرها بشكل موضعي (`pairing list telegram`) أو باستخدام `--channel `.
-   `pairing list` يدعم `--account ` للقنوات متعددة الحسابات.
-   `pairing approve` يدعم `--account ` و `--notify`.
-   إذا تم تكوين قناة واحدة فقط قادرة على الإقران، فإن `pairing approve ` مسموح به.

[onboard](./onboard.md)[plugins](./plugins.md)