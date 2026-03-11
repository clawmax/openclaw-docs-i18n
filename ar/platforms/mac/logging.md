

  التطبيق المصاحب لنظام macOS

  
# تسجيل macOS

## سجل تشخيص ملف دوار (لوحة التصحيح)

يقوم OpenClaw بتوجيه سجلات تطبيق macOS عبر swift-log (التسجيل الموحد افتراضيًا) ويمكنه كتابة سجل ملف محلي ودوار على القرص عندما تحتاج إلى تسجيل دائم.

-   مستوى التفصيل: **لوحة التصحيح → السجلات → تسجيل التطبيق → مستوى التفصيل**
-   التمكين: **لوحة التصحيح → السجلات → تسجيل التطبيق → "كتابة سجل تشخيص دوار (JSONL)"**
-   الموقع: `~/Library/Logs/OpenClaw/diagnostics.jsonl` (يدور تلقائيًا؛ الملفات القديمة تحصل على اللاحقة `.1`, `.2`, …)
-   المسح: **لوحة التصحيح → السجلات → تسجيل التطبيق → "مسح"**

ملاحظات:

-   هذا **معطل افتراضيًا**. قم بتمكينه فقط أثناء التصحيح النشط.
-   تعامل مع الملف على أنه حساس؛ لا تشاركه دون مراجعة.

## بيانات التسجيل الموحد الخاصة على macOS

يقوم التسجيل الموحد بإخفاء معظم الحمولات ما لم يقم نظام فرعي باختيار `privacy -off`. وفقًا لكتابة Peter حول [مخاوف خصوصية تسجيل macOS](https://steipete.me/posts/2025/logging-privacy-shenanigans) (2025)، يتم التحكم في هذا بواسطة ملف plist في `/Library/Preferences/Logging/Subsystems/` مفتاحه باسم النظام الفرعي. فقط إدخالات السجل الجديدة تلتقط العلم، لذا قم بتمكينه قبل إعادة إنتاج المشكلة.

## تمكينه لـ OpenClaw (ai.openclaw)

-   اكتب ملف plist في ملف مؤقت أولاً، ثم قم بتثبيته بشكل ذري كـ root:

```bash
cat <<'EOF' >/tmp/ai.openclaw.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>DEFAULT-OPTIONS</key>
    <dict>
        <key>Enable-Private-Data</key>
        <true/>
    </dict>
</dict>
</plist>
EOF
sudo install -m 644 -o root -g wheel /tmp/ai.openclaw.plist /Library/Preferences/Logging/Subsystems/ai.openclaw.plist
```

-   لا يلزم إعادة التشغيل؛ يلاحظ logd الملف بسرعة، لكن فقط أسطر السجل الجديدة ستشمل الحمولات الخاصة.
-   اعرض الناتج الأغنى باستخدام المساعد الحالي، مثل `./scripts/clawlog.sh --category WebChat --last 5m`.

## تعطيله بعد التصحيح

-   قم بإزالة التجاوز: `sudo rm /Library/Preferences/Logging/Subsystems/ai.openclaw.plist`.
-   يمكنك تشغيل `sudo log config --reload` اختياريًا لإجبار logd على إسقاط التجاوز فورًا.
-   تذكر أن هذا السطح يمكن أن يشمل أرقام الهواتف ومحتوى الرسائل؛ احتفظ بملف plist في مكانه فقط أثناء حاجتك النشطة للتفاصيل الإضافية.

[أيقونة شريط القائمة](./icon.md)[أذونات macOS](./permissions.md)

---