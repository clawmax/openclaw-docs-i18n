

  الإضافات

  
# إضافة Zalo الشخصية

دعم Zalo الشخصي لـ OpenClaw عبر إضافة، باستخدام `zca-js` الأصلي لأتمتة حساب مستخدم Zalo عادي.

> **تحذير:** قد تؤدي الأتمتة غير الرسمية إلى تعليق/حظر الحساب. الاستخدام على مسؤوليتك الخاصة.

## التسمية

معرف القناة هو `zalouser` لتوضيح أنها تؤتمت **حساب مستخدم Zalo شخصي** (غير رسمي). نحتفظ بـ `zalo` محجوزًا لتكامل محتمل مستقبلي مع واجهة برمجة تطبيقات Zalo الرسمية.

## أين تعمل

تعمل هذه الإضافة **داخل عملية البوابة**. إذا كنت تستخدم بوابة بعيدة، قم بتثبيتها/ضبطها على **الجهاز الذي يشغل البوابة**، ثم أعد تشغيل البوابة. لا يلزم وجود ملف ثنائي خارجي لـ `zca`/`openzca` CLI.

## التثبيت

### الخيار أ: التثبيت من npm

```bash
openclaw plugins install @openclaw/zalouser
```

أعد تشغيل البوابة بعد ذلك.

### الخيار ب: التثبيت من مجلد محلي (لتطوير)

```bash
openclaw plugins install ./extensions/zalouser
cd ./extensions/zalouser && pnpm install
```

أعد تشغيل البوابة بعد ذلك.

## التكوين

يوجد تكوين القناة تحت `channels.zalouser` (وليس `plugins.entries.*`):

```json
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing",
    },
  },
}
```

## واجهة سطر الأوامر

```bash
openclaw channels login --channel zalouser
openclaw channels logout --channel zalouser
openclaw channels status --probe
openclaw message send --channel zalouser --target <threadId> --message "Hello from OpenClaw"
openclaw directory peers list --channel zalouser --query "name"
```

## أداة الوكيل

اسم الأداة: `zalouser` الإجراءات: `send`, `image`, `link`, `friends`, `groups`, `me`, `status` تدعم إجراءات رسائل القناة أيضًا `react` للتفاعلات على الرسائل.

[إضافة المكالمات الصوتية](./voice-call.md)[بيان الإضافة](./manifest.md)

---