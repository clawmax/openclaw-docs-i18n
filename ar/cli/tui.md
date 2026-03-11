

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