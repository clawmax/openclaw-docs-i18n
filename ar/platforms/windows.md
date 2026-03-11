

  نظرة عامة على المنصات

  
# Windows (WSL2)

يوصى باستخدام OpenClaw على Windows **عبر WSL2** (يوصى بـ Ubuntu). يعمل CLI + البوابة داخل لينكس، مما يحافظ على ثبات بيئة التشغيل ويجعل الأدوات أكثر توافقًا (Node/Bun/pnpm، ملفات ثنائية لينكس، المهارات). قد يكون الإصدار الأصلي لنظام Windows أكثر تعقيدًا. يمنحك WSL2 تجربة لينكس كاملة — أمر واحد للتثبيت: `wsl --install`. تطبيقات المرافقة الأصلية لنظام Windows قيد التخطيط.

## التثبيت (WSL2)

-   [بدء الاستخدام](../start/getting-started.md) (استخدم داخل WSL)
-   [التثبيت والتحديثات](../install/updating.md)
-   الدليل الرسمي لـ WSL2 (Microsoft): [https://learn.microsoft.com/windows/wsl/install](https://learn.microsoft.com/windows/wsl/install)

## البوابة

-   [كتيب تشغيل البوابة](../gateway.md)
-   [التكوين](../gateway/configuration.md)

## تثبيت خدمة البوابة (CLI)

داخل WSL2:

```bash
openclaw onboard --install-daemon
```

أو:

```bash
openclaw gateway install
```

أو:

```bash
openclaw configure
```

اختر **خدمة البوابة** عند المطالبة. الإصلاح/الترحيل:

```bash
openclaw doctor
```

## التشغيل التلقائي للبوابة قبل تسجيل الدخول إلى Windows

للإعدادات بدون واجهة مستخدم، تأكد من تشغيل سلسلة الإقلاع الكاملة حتى عندما لا يسجل أحد الدخول إلى Windows.

### 1) الحفاظ على تشغيل خدمات المستخدم دون تسجيل دخول

داخل WSL:

```bash
sudo loginctl enable-linger "$(whoami)"
```

### 2) تثبيت خدمة مستخدم بوابة OpenClaw

داخل WSL:

```bash
openclaw gateway install
```

### 3) بدء WSL تلقائيًا عند إقلاع Windows

في PowerShell كمسؤول:

```bash
schtasks /create /tn "WSL Boot" /tr "wsl.exe -d Ubuntu --exec /bin/true" /sc onstart /ru SYSTEM
```

استبدل `Ubuntu` باسم توزيعتك من:

```bash
wsl --list --verbose
```

### التحقق من سلسلة بدء التشغيل

بعد إعادة التشغيل (قبل تسجيل الدخول إلى Windows)، تحقق من داخل WSL:

```bash
systemctl --user is-enabled openclaw-gateway
systemctl --user status openclaw-gateway --no-pager
```

## متقدم: عرض خدمات WSL عبر الشبكة المحلية (portproxy)

لـ WSL شبكته الافتراضية الخاصة. إذا احتاجت آلة أخرى للوصول إلى خدمة تعمل **داخل WSL** (SSH، خادم TTS محلي، أو البوابة)، يجب عليك توجيه منفذ في Windows إلى عنوان IP الحالي لـ WSL. يتغير عنوان IP الخاص بـ WSL بعد عمليات إعادة التشغيل، لذا قد تحتاج إلى تحديث قاعدة التوجيه. مثال (PowerShell **كمسؤول**):

```powershell
$Distro = "Ubuntu-24.04"
$ListenPort = 2222
$TargetPort = 22

$WslIp = (wsl -d $Distro -- hostname -I).Trim().Split(" ")[0]
if (-not $WslIp) { throw "WSL IP not found." }

netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=$ListenPort `
  connectaddress=$WslIp connectport=$TargetPort
```

السماح بالمنفذ عبر جدار حماية Windows (مرة واحدة):

```
New-NetFirewallRule -DisplayName "WSL SSH $ListenPort" -Direction Inbound `
  -Protocol TCP -LocalPort $ListenPort -Action Allow
```

تحديث portproxy بعد إعادة تشغيل WSL:

```
netsh interface portproxy delete v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 | Out-Null
netsh interface portproxy add v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 `
  connectaddress=$WslIp connectport=$TargetPort | Out-Null
```

ملاحظات:

-   يستهدف SSH من آلة أخرى **عنوان IP مضيف Windows** (مثال: `ssh user@windows-host -p 2222`).
-   يجب أن تشير العقد البعيدة إلى **عنوان URL للبوالة قابل للوصول** (وليس `127.0.0.1`)؛ استخدم `openclaw status --all` للتأكد.
-   استخدم `listenaddress=0.0.0.0` للوصول عبر الشبكة المحلية؛ `127.0.0.1` يبقيها محلية فقط.
-   إذا أردت أن يكون هذا تلقائيًا، سجل مهمة مجدولة لتشغيل خطوة التحديث عند تسجيل الدخول.

## تثبيت WSL2 خطوة بخطوة

### 1) تثبيت WSL2 + Ubuntu

افتح PowerShell (مسؤول):

```bash
wsl --install
# أو اختر توزيعة صراحة:
wsl --list --online
wsl --install -d Ubuntu-24.04
```

أعد التشغيل إذا طلب Windows.

### 2) تمكين systemd (مطلوب لتثبيت البوابة)

في طرفية WSL الخاصة بك:

```bash
sudo tee /etc/wsl.conf >/dev/null <<'EOF'
[boot]
systemd=true
EOF
```

ثم من PowerShell:

```bash
wsl --shutdown
```

أعد فتح Ubuntu، ثم تحقق:

```bash
systemctl --user status
```

### 3) تثبيت OpenClaw (داخل WSL)

اتبع سير عمل بدء الاستخدام لنظام لينكس داخل WSL:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # يقوم بتثبيت تبعيات واجهة المستخدم تلقائيًا عند التشغيل الأول
pnpm build
openclaw onboard
```

الدليل الكامل: [بدء الاستخدام](../start/getting-started.md)

## تطبيق المرافقة لنظام Windows

ليس لدينا تطبيق مرافقة لنظام Windows بعد. المساهمات مرحب بها إذا كنت تريد المساهمة لتحقيق ذلك.

[تطبيق لينكس](./linux.md)[تطبيق أندرويد](./android.md)