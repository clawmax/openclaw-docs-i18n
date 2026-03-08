title: "استخدام وأمثلة أمر OpenClaw CLI للاتصال الصوتي"
description: "تعلم كيفية استخدام أوامر OpenClaw CLI للاتصال الصوتي لإجراء وإدارة وإنهاء مكالمات الذكاء الاصطناعي الصوتية، وكشف نقاط نهاية الويب هوك بشكل آمن عبر Tailscale."
keywords: ["openclaw voicecall", "أوامر صوتية لسطر الأوامر", "مكالمة صوتية بالذكاء الاصطناعي", "webhooks عبر Tailscale", "إضافة الاتصال الصوتي", "أوامر سطر الأوامر", "أتمتة المكالمات الصوتية", "كشف نقاط نهاية الويب هوك"]
---

  أوامر سطر الأوامر

  
# voicecall

`voicecall` هو أمر مُقدم من خلال إضافة. يظهر فقط إذا كانت إضافة المكالمات الصوتية مثبتة ومفعلة. الوثائق الرئيسية:

-   إضافة المكالمات الصوتية: [المكالمة الصوتية](../plugins/voice-call.md)

## الأوامر الشائعة

```bash
openclaw voicecall status --call-id <id>
openclaw voicecall call --to "+15555550123" --message "Hello" --mode notify
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall end --call-id <id>
```

## كشف نقاط نهاية الويب هوك (Tailscale)

```bash
openclaw voicecall expose --mode serve
openclaw voicecall expose --mode funnel
openclaw voicecall expose --mode off
```

ملاحظة أمنية: قم بكشف نقطة نهاية الويب هوك فقط للشبكات التي تثق بها. يُفضل استخدام وضع Serve في Tailscale على وضع Funnel عندما يكون ذلك ممكنًا.

[تحديث](./update.md)[webhooks](./webhooks.md)

---