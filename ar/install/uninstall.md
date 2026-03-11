

  الصيانة

  
# إلغاء التثبيت

مساران:

*   **الطريق السهل** إذا كان `openclaw` لا يزال مثبتًا.
*   **الإزالة اليدوية للخدمة** إذا كان CLI غير موجود ولكن الخدمة لا تزال قيد التشغيل.

## الطريق السهل (CLI لا يزال مثبتًا)

موصى به: استخدم أداة الإلغاء المدمجة:

```bash
openclaw uninstall
```

غير التفاعلي (للأتمتة / npx):

```bash
openclaw uninstall --all --yes --non-interactive
npx -y openclaw uninstall --all --yes --non-interactive
```

الخطوات اليدوية (نفس النتيجة):

1.  أوقف خدمة البوابة:

```bash
openclaw gateway stop
```

2.  ألغِ تثبيت خدمة البوابة (launchd/systemd/schtasks):

```bash
openclaw gateway uninstall
```

3.  احذف الحالة + الإعدادات:

```bash
rm -rf "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"
```

إذا قمت بتعيين `OPENCLAW_CONFIG_PATH` إلى موقع مخصص خارج مجلد الحالة، احذف ذلك الملف أيضًا.

4.  احذف مساحة العمل الخاصة بك (اختياري، يزيل ملفات الوكيل):

```bash
rm -rf ~/.openclaw/workspace
```

5.  أزل تثبيت CLI (اختر الطريقة التي استخدمتها):

```bash
npm rm -g openclaw
pnpm remove -g openclaw
bun remove -g openclaw
```

6.  إذا قمت بتثبيت تطبيق macOS:

```bash
rm -rf /Applications/OpenClaw.app
```

ملاحظات:

*   إذا استخدمت ملفات تعريف (`--profile` / `OPENCLAW_PROFILE`)، كرر الخطوة 3 لكل مجلد حالة (القيم الافتراضية هي `~/.openclaw-`).
*   في الوضع البعيد، يعيش مجلد الحالة على **مضيف البوابة**، لذا قم بتشغيل الخطوات من 1 إلى 4 هناك أيضًا.

## الإزالة اليدوية للخدمة (CLI غير مثبت)

استخدم هذا إذا استمرت خدمة البوابة في العمل ولكن `openclaw` مفقود.

### macOS (launchd)

التسمية الافتراضية هي `ai.openclaw.gateway` (أو `ai.openclaw.`؛ قد لا تزال التسميات القديمة `com.openclaw.*` موجودة):

```bash
launchctl bootout gui/$UID/ai.openclaw.gateway
rm -f ~/Library/LaunchAgents/ai.openclaw.gateway.plist
```

إذا استخدمت ملف تعريف، استبدل التسمية واسم plist بـ `ai.openclaw.`. أزل أي ملفات plist قديمة `com.openclaw.*` إذا كانت موجودة.

### Linux (وحدة systemd للمستخدم)

اسم الوحدة الافتراضي هو `openclaw-gateway.service` (أو `openclaw-gateway-.service`):

```bash
systemctl --user disable --now openclaw-gateway.service
rm -f ~/.config/systemd/user/openclaw-gateway.service
systemctl --user daemon-reload
```

### Windows (مهمة مجدولة)

اسم المهمة الافتراضي هو `OpenClaw Gateway` (أو `OpenClaw Gateway ()`). يعيش سكريبت المهمة تحت مجلد الحالة الخاص بك.

```bash
schtasks /Delete /F /TN "OpenClaw Gateway"
Remove-Item -Force "$env:USERPROFILE\.openclaw\gateway.cmd"
```

إذا استخدمت ملف تعريف، احذف اسم المهمة المطابق و `~\.openclaw-\gateway.cmd`.

## التثبيت العادي مقابل استنساخ المصدر

### التثبيت العادي (install.sh / npm / pnpm / bun)

إذا استخدمت `https://openclaw.ai/install.sh` أو `install.ps1`، تم تثبيت CLI باستخدام `npm install -g openclaw@latest`. أزله باستخدام `npm rm -g openclaw` (أو `pnpm remove -g` / `bun remove -g` إذا قمت بالتثبيت بهذه الطريقة).

### استنساخ المصدر (git clone)

إذا كنت تشغل من نسخة مستنسخة من المستودع (`git clone` + `openclaw ...` / `bun run openclaw ...`):

1.  ألغِ تثبيت خدمة البوابة **قبل** حذف المستودع (استخدم الطريق السهل أعلاه أو الإزالة اليدوية للخدمة).
2.  احذف مجلد المستودع.
3.  أزل الحالة + مساحة العمل كما هو موضح أعلاه.

[دليل الترحيل](./migrating.md)[استضافة VPS](../vps.md)