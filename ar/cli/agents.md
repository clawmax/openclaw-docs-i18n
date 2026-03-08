

  أوامر سطر الأوامر

  
# agents

إدارة الوكلاء المعزولين (مساحة العمل + المصادقة + التوجيه). مواضيع ذات صلة:

-   التوجيه متعدد الوكلاء: [التوجيه متعدد الوكلاء](../concepts/multi-agent.md)
-   مساحة عمل الوكيل: [مساحة عمل الوكيل](../concepts/agent-workspace.md)

## أمثلة

```bash
openclaw agents list
openclaw agents add work --workspace ~/.openclaw/workspace-work
openclaw agents bindings
openclaw agents bind --agent work --bind telegram:ops
openclaw agents unbind --agent work --bind telegram:ops
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
openclaw agents set-identity --agent main --avatar avatars/openclaw.png
openclaw agents delete work
```

## ارتباطات التوجيه

استخدم ارتباطات التوجيه لتثبيت حركة المرور الواردة من القناة على وكيل محدد. عرض قائمة الارتباطات:

```bash
openclaw agents bindings
openclaw agents bindings --agent work
openclaw agents bindings --json
```

إضافة ارتباطات:

```bash
openclaw agents bind --agent work --bind telegram:ops --bind discord:guild-a
```

إذا حذفت `accountId` (`--bind `)، يقوم OpenClaw بحلها من الإعدادات الافتراضية للقناة وخطافات إعداد الإضافة عندما تكون متاحة.

### سلوك نطاق الارتباط

-   الارتباط بدون `accountId` يطابق فقط الحساب الافتراضي للقناة.
-   `accountId: "*"` هو الخيار الاحتياطي على مستوى القناة (جميع الحسابات) وهو أقل تحديدًا من ارتباط حساب صريح.
-   إذا كان لدى نفس الوكيل بالفعل ارتباط قناة مطابق بدون `accountId`، وقمت لاحقًا بربطه باستخدام `accountId` صريح أو محلول، فإن OpenClaw يقوم بترقية هذا الارتباط الموجود في مكانه بدلاً من إضافة نسخة مكررة.

مثال:

```bash
# ارتباط القناة الأولي فقط
openclaw agents bind --agent work --bind telegram

# لاحقًا، الترقية إلى ارتباط محدد بنطاق الحساب
openclaw agents bind --agent work --bind telegram:ops
```

بعد الترقية، يكون التوجيه لهذا الارتباط محددًا بـ `telegram:ops`. إذا كنت تريد أيضًا توجيهًا للحساب الافتراضي، أضفه صراحةً (على سبيل المثال `--bind telegram:default`). إزالة الارتباطات:

```bash
openclaw agents unbind --agent work --bind telegram:ops
openclaw agents unbind --agent work --all
```

## ملفات الهوية

يمكن أن تتضمن كل مساحة عمل وكيل ملف `IDENTITY.md` في جذر مساحة العمل:

-   مثال على المسار: `~/.openclaw/workspace/IDENTITY.md`
-   `set-identity --from-identity` يقرأ من جذر مساحة العمل (أو من `--identity-file` صريح)

يتم حل مسارات الصورة الرمزية (Avatar) بالنسبة لجذر مساحة العمل.

## تعيين الهوية

يكتب `set-identity` الحقول في `agents.list[].identity`:

-   `name`
-   `theme`
-   `emoji`
-   `avatar` (مسار نسبي لمساحة العمل، عنوان URL http(s)، أو data URI)

التحميل من `IDENTITY.md`:

```bash
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
```

تجاوز الحقول بشكل صريح:

```bash
openclaw agents set-identity --agent main --name "OpenClaw" --emoji "🦞" --avatar avatars/openclaw.png
```

عينة تهيئة:

```json
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "OpenClaw",
          theme: "space lobster",
          emoji: "🦞",
          avatar: "avatars/openclaw.png",
        },
      },
    ],
  },
}
```

[agent](./agent.md)[approvals](./approvals.md)