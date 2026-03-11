

  أوامر CLI

  
# agent

قم بتشغيل دورة وكيل عبر البوابة (استخدم `--local` للتضمين). استخدم `--agent ` لاستهداف وكيل مُهيأ مباشرة. متعلق:

-   أداة إرسال الوكيل: [إرسال الوكيل](../tools/agent-send.md)

## أمثلة

```bash
openclaw agent --to +15555550123 --message "status update" --deliver
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```

## ملاحظات

-   عندما يؤدي هذا الأمر إلى إعادة توليد `models.json`، يتم حفظ بيانات اعتماد موفري الخدمة المدارة بواسطة SecretRef كعلامات غير سرية (على سبيل المثال أسماء متغيرات البيئة أو `secretref-managed`)، وليس كنص عادي للسر تم حله.

[acp](./acp.md)[agents](./agents.md)

---