title: "دليل أوامر واجهة المستخدم النصية (TUI) لأداة OpenClaw CLI"
description: "تعلم كيفية استخدام أمر tui في أداة OpenClaw CLI لفتح واجهة المستخدم النصية المتصلة بالبوابة. يتضمن أمثلة للاتصال والمصادقة."
keywords: ["openclaw tui", "واجهة المستخدم النصية للأداة السطرية", "اتصال البوابة", "أوامر CLI", "دليل TUI", "openclaw cli", "واجهة المستخدم الطرفية", "مصادقة البوابة"]
---

  أوامر CLI

  
# tui

افتح واجهة المستخدم النصية المتصلة بالبوابة. ذات صلة:

-   دليل TUI: [TUI](../web/tui.md)

ملاحظات:

-   يحل أمر `tui` مراجع الأسرار (SecretRefs) المكونة لمصادقة البوابة للتوثيق بالرمز/كلمة المرور عندما يكون ذلك ممكنًا (مزودي `env`/`file`/`exec`).

## أمثلة

```bash
openclaw tui
openclaw tui --url ws://127.0.0.1:18789 --token <token>
openclaw tui --session main --deliver
```

[system](./system.md)[uninstall](./uninstall.md)

---