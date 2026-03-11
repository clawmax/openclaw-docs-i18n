

  المتصفح

  
# استكشاف أخطاء المتصفح وإصلاحها

## المشكلة: "فشل بدء Chrome CDP على المنفذ 18800"

يفشل خادم التحكم في متصفح OpenClaw في تشغيل Chrome/Brave/Edge/Chromium مع الخطأ:

```json
{"error":"Error: Failed to start Chrome CDP on port 18800 for profile \"openclaw\"."}
```

### السبب الجذري

على Ubuntu (والعديد من توزيعات Linux)، يكون تثبيت Chromium الافتراضي عبارة عن **حزمة snap**. يسبب حصر AppArmor الخاص بـ Snap في التداخل مع طريقة إنشاء ومراقبة OpenClaw لعملية المتصفح. يقوم أمر `apt install chromium` بتثبيت حزمة وهمية تعيد التوجيه إلى snap:

```
Note, selecting 'chromium-browser' instead of 'chromium'
chromium-browser is already the newest version (2:1snap1-0ubuntu2).
```

هذا ليس متصفحًا حقيقيًا — إنه مجرد غلاف.

### الحل 1: تثبيت Google Chrome (مُوصى به)

قم بتثبيت حزمة Google Chrome الرسمية `.deb`، والتي لا يتم عزلها بواسطة snap:

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
sudo apt --fix-broken install -y  # إذا كانت هناك أخطاء في التبعيات
```

ثم قم بتحديث تكوين OpenClaw (`~/.openclaw/openclaw.json`):

```json
{
  "browser": {
    "enabled": true,
    "executablePath": "/usr/bin/google-chrome-stable",
    "headless": true,
    "noSandbox": true
  }
}
```

### الحل 2: استخدام Snap Chromium مع وضع الإرفاق فقط

إذا كنت مضطرًا لاستخدام snap Chromium، فقم بتكوين OpenClaw للإرفاق بمتصفح تم بدء تشغيله يدويًا:

1.  تحديث التكوين:

```json
{
  "browser": {
    "enabled": true,
    "attachOnly": true,
    "headless": true,
    "noSandbox": true
  }
}
```

2.  بدء تشغيل Chromium يدويًا:

```
chromium-browser --headless --no-sandbox --disable-gpu \
  --remote-debugging-port=18800 \
  --user-data-dir=$HOME/.openclaw/browser/openclaw/user-data \
  about:blank &
```

3.  اختياريًا: إنشاء خدمة systemd للمستخدم لبدء تشغيل Chrome تلقائيًا:

```bash
# ~/.config/systemd/user/openclaw-browser.service
[Unit]
Description=OpenClaw Browser (Chrome CDP)
After=network.target

[Service]
ExecStart=/snap/bin/chromium --headless --no-sandbox --disable-gpu --remote-debugging-port=18800 --user-data-dir=%h/.openclaw/browser/openclaw/user-data about:blank
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
```

تفعيل الخدمة باستخدام: `systemctl --user enable --now openclaw-browser.service`

### التحقق من عمل المتصفح

التحقق من الحالة:

```bash
curl -s http://127.0.0.1:18791/ | jq '{running, pid, chosenBrowser}'
```

اختبار التصفح:

```bash
curl -s -X POST http://127.0.0.1:18791/start
curl -s http://127.0.0.1:18791/tabs
```

### مرجع التكوين

| الخيار | الوصف | الافتراضي |
| --- | --- | --- |
| `browser.enabled` | تمكين التحكم في المتصفح | `true` |
| `browser.executablePath` | المسار إلى ملف ثنائي لمتصفح يعتمد على Chromium (Chrome/Brave/Edge/Chromium) | يتم الكشف تلقائيًا (يفضل المتصفح الافتراضي عندما يكون من نوع Chromium) |
| `browser.headless` | التشغيل بدون واجهة مستخدم رسومية | `false` |
| `browser.noSandbox` | إضافة علم `--no-sandbox` (مطلوب لبعض إعدادات Linux) | `false` |
| `browser.attachOnly` | عدم تشغيل المتصفح، فقط الإرفاق بمتصفح موجود | `false` |
| `browser.cdpPort` | منفذ بروتوكول Chrome DevTools | `18800` |

### المشكلة: "ناقل امتداد Chrome يعمل، ولكن لا يوجد علامة تبويب متصلة"

أنت تستخدم ملف التعريف `chrome` (ناقل الامتداد). يتوقع أن يكون امتداد متصفح OpenClaw متصلاً بعلامة تبويب نشطة. خيارات الإصلاح:

1.  **استخدام المتصفح المُدار:** `openclaw browser start --browser-profile openclaw` (أو تعيين `browser.defaultProfile: "openclaw"`).
2.  **استخدام ناقل الامتداد:** قم بتثبيت الامتداد، وافتح علامة تبويب، وانقر على أيقونة امتداد OpenClaw لإرفاقه.

ملاحظات:

-   يستخدم ملف التعريف `chrome` **متصفح Chromium الافتراضي لنظامك** عندما يكون ذلك ممكنًا.
-   تقوم ملفات التعريف المحلية `openclaw` بتعيين `cdpPort`/`cdpUrl` تلقائيًا؛ قم بتعيينها فقط لـ CDP البعيد.

[امتداد Chrome](./chrome-extension.md)[إرسال الوكيل](./agent-send.md)