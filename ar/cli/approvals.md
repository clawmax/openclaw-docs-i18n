title: "إدارة موافقات تنفيذ OpenClaw CLI للبوابة المحلية والعقد"
description: "تعلم كيفية الحصول على موافقات التنفيذ وتعيينها وإدارتها للمضيفين المحليين والبوابات والعقد باستخدام أوامر OpenClaw CLI وأدوات القائمة المسموح بها."
keywords: ["موافقات openclaw", "موافقات تنفيذ cli", "أوامر القائمة المسموح بها", "موافقات مضيف العقدة", "موافقات البوابة", "إدارة موافقات التنفيذ", "openclaw cli", "موافقات سطر الأوامر"]
---

  أوامر CLI

  
# الموافقات

إدارة موافقات التنفيذ لـ **المضيف المحلي**، أو **مضيف البوابة**، أو **مضيف العقدة**. بشكل افتراضي، تستهدف الأوامر ملف الموافقات المحلي على القرص. استخدم `--gateway` لاستهداف البوابة، أو `--node` لاستهداف عقدة محددة. مواضيع ذات صلة:

-   موافقات التنفيذ: [موافقات التنفيذ](../tools/exec-approvals.md)
-   العقد: [العقد](../nodes.md)

## الأوامر الشائعة

```bash
openclaw approvals get
openclaw approvals get --node <id|name|ip>
openclaw approvals get --gateway
```

## استبدال الموافقات من ملف

```bash
openclaw approvals set --file ./exec-approvals.json
openclaw approvals set --node <id|name|ip> --file ./exec-approvals.json
openclaw approvals set --gateway --file ./exec-approvals.json
```

## أدوات القائمة المسموح بها

```bash
openclaw approvals allowlist add "~/Projects/**/bin/rg"
openclaw approvals allowlist add --agent main --node <id|name|ip> "/usr/bin/uptime"
openclaw approvals allowlist add --agent "*" "/usr/bin/uname"

openclaw approvals allowlist remove "~/Projects/**/bin/rg"
```

## ملاحظات

-   `--node` يستخدم نفس المحلل المستخدم في `openclaw nodes` (المعرف، الاسم، عنوان IP، أو بادئة المعرف).
-   القيمة الافتراضية لـ `--agent` هي `"*"`، والتي تنطبق على جميع الوكلاء.
-   يجب على مضيف العقدة الإعلان عن `system.execApprovals.get/set` (تطبيق macOS أو مضيف عقدة بدون واجهة).
-   يتم تخزين ملفات الموافقات لكل مضيف في `~/.openclaw/exec-approvals.json`.

[الوكلاء](./agents.md)[المتصفح](./browser.md)