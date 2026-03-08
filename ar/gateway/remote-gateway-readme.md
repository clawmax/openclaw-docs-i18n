

  وصول بعيد

  
# إعداد البوابة البعيدة

يستخدم OpenClaw.app نفق SSH للاتصال ببوابة بعيدة. يوضح لك هذا الدليل كيفية إعداده.

## نظرة عامة

## الإعداد السريع

### الخطوة 1: إضافة تكوين SSH

قم بتحرير ملف `~/.ssh/config` وأضف:

```
Host remote-gateway
    HostName <REMOTE_IP>          # e.g., 172.27.187.184
    User <REMOTE_USER>            # e.g., jefferson
    LocalForward 18789 127.0.0.1:18789
    IdentityFile ~/.ssh/id_rsa
```

استبدل `<REMOTE_IP>` و `<REMOTE_USER>` بقيمك.

### الخطوة 2: نسخ مفتاح SSH

انسخ مفتاحك العام إلى الجهاز البعيد (أدخل كلمة المرور مرة واحدة):

```bash
ssh-copy-id -i ~/.ssh/id_rsa <REMOTE_USER>@<REMOTE_IP>
```

### الخطوة 3: تعيين رمز البوابة

```bash
launchctl setenv OPENCLAW_GATEWAY_TOKEN "<your-token>"
```

### الخطوة 4: بدء نفق SSH

```bash
ssh -N remote-gateway &
```

### الخطوة 5: إعادة تشغيل OpenClaw.app

```bash
# أغلق OpenClaw.app (⌘Q)، ثم أعد فتحه:
open /path/to/OpenClaw.app
```

سيقوم التطبيق الآن بالاتصال بالبوابة البعيدة عبر نفق SSH.

* * *

## بدء النفق تلقائياً عند تسجيل الدخول

لبدء نفق SSH تلقائياً عند تسجيل الدخول، قم بإنشاء وكيل تشغيل (Launch Agent).

### إنشاء ملف PLIST

احفظ هذا الملف باسم `~/Library/LaunchAgents/ai.openclaw.ssh-tunnel.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>ai.openclaw.ssh-tunnel</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/bin/ssh</string>
        <string>-N</string>
        <string>remote-gateway</string>
    </array>
    <key>KeepAlive</key>
    <true/>
    <key>RunAtLoad</key>
    <true/>
</dict>
</plist>
```

### تحميل وكيل التشغيل

```bash
launchctl bootstrap gui/$UID ~/Library/LaunchAgents/ai.openclaw.ssh-tunnel.plist
```

سيقوم النفق الآن بما يلي:

-   البدء تلقائياً عند تسجيل الدخول
-   إعادة التشغيل إذا تعطل
-   الاستمرار في العمل في الخلفية

ملاحظة قديمة: قم بإزالة أي وكيل تشغيل `com.openclaw.ssh-tunnel` متبقي إذا كان موجوداً.

* * *

## استكشاف الأخطاء وإصلاحها

**التحقق مما إذا كان النفق يعمل:**

```bash
ps aux | grep "ssh -N remote-gateway" | grep -v grep
lsof -i :18789
```

**إعادة تشغيل النفق:**

```bash
launchctl kickstart -k gui/$UID/ai.openclaw.ssh-tunnel
```

**إيقاف النفق:**

```bash
launchctl bootout gui/$UID/ai.openclaw.ssh-tunnel
```

* * *

## آلية العمل

| المكون | وظيفته |
| --- | --- |
| `LocalForward 18789 127.0.0.1:18789` | يقوم بتوجيه المنفذ المحلي 18789 إلى المنفذ البعيد 18789 |
| `ssh -N` | SSH بدون تنفيذ أوامر عن بعد (فقط توجيه المنافذ) |
| `KeepAlive` | يعيد تشغيل النفق تلقائياً إذا تعطل |
| `RunAtLoad` | يبدأ النفق عند تحميل الوكيل |

يتصل تطبيق OpenClaw.app بـ `ws://127.0.0.1:18789` على جهازك العميل. يقوم نفق SSH بتوجيه هذا الاتصال إلى المنفذ 18789 على الجهاز البعيد حيث تعمل البوابة.

[وصول بعيد](./remote.md)[Tailscale](./tailscale.md)

---