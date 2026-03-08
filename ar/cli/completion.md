

  أوامر CLI

  
# completion

قم بإنشاء نصوص إكمال الأوامر للطرفية وقم بتثبيتها اختياريًا في ملف تعريف طرفيتك.

## الاستخدام

```bash
openclaw completion
openclaw completion --shell zsh
openclaw completion --install
openclaw completion --shell fish --install
openclaw completion --write-state
openclaw completion --shell bash --write-state
```

## الخيارات

-   `-s, --shell `: الطرفية المستهدفة (`zsh`, `bash`, `powershell`, `fish`; الافتراضي: `zsh`)
-   `-i, --install`: تثبيت الإكمال عن طريق إضافة سطر مصدر إلى ملف تعريف طرفيتك
-   `--write-state`: كتابة نص/نصوص الإكمال إلى `$OPENCLAW_STATE_DIR/completions` دون طباعتها إلى stdout
-   `-y, --yes`: تخطي مطالبات تأكيد التثبيت

## ملاحظات

-   `--install` يكتب كتلة صغيرة بعنوان "إكمال OpenClaw" في ملف تعريف طرفيتك ويوجهها إلى النص المخزن مؤقتًا.
-   بدون `--install` أو `--write-state`، يطبع الأمر النص إلى stdout.
-   توليد الإكمال يقوم بتحميل شجرة الأوامر بسرعة بحيث يتم تضمين الأوامر الفرعية المتداخلة.

[clawbot](./clawbot.md)[config](./config.md)