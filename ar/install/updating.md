

  الصيانة

  
# التحديث

OpenClaw يتطور بسرعة (قبل الإصدار "1.0"). تعامل مع التحديثات مثل تحديث البنية التحتية: قم بالتحديث → قم بتشغيل الفحوصات → أعد التشغيل (أو استخدم `openclaw update`، والذي يعيد التشغيل) → تحقق.

## الموصى به: إعادة تشغيل مثبت الموقع (الترقية في المكان)

مسار التحديث **المفضل** هو إعادة تشغيل المثبت من الموقع. فهو يكتشف التثبيتات الموجودة، ويقوم بالترقية في المكان، ويشغل `openclaw doctor` عند الحاجة.

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

ملاحظات:

-   أضف `--no-onboard` إذا كنت لا تريد تشغيل معالج الإعداد الأولي مرة أخرى.
-   بالنسبة **لتثبيتات المصدر**، استخدم:
    
    نسخ
    
    ```bash
    curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git --no-onboard
    ```
    
    سيقوم المثبت بـ `git pull --rebase` **فقط** إذا كان المستودع نظيفًا.
-   بالنسبة **للتثبيتات العامة**، يستخدم البرنامج النصي `npm install -g openclaw@latest` في الخلفية.
-   ملاحظة قديمة: `clawdbot` يظل متاحًا كغلاف توافق.

## قبل التحديث

-   اعرف كيف قمت بالتثبيت: **عام** (npm/pnpm) مقابل **من المصدر** (git clone).
-   اعرف كيف تعمل بوابةك: **في الطرفية الأمامية** مقابل **خدمة مُشرفة** (launchd/systemd).
-   خذ لقطة لبيئتك المخصصة:
    -   الإعدادات: `~/.openclaw/openclaw.json`
    -   بيانات الاعتماد: `~/.openclaw/credentials/`
    -   مساحة العمل: `~/.openclaw/workspace`

## التحديث (تثبيت عام)

تثبيت عام (اختر واحدًا):

```bash
npm i -g openclaw@latest
```

```bash
pnpm add -g openclaw@latest
```

**لا** نوصي باستخدام Bun لوقت تشغيل البوابة (أخطاء في WhatsApp/Telegram). للتبديل بين قنوات التحديث (تثبيتات git + npm):

```bash
openclaw update --channel beta
openclaw update --channel dev
openclaw update --channel stable
```

استخدم `--tag <dist-tag|version>` لعلامة/إصدار تثبيت لمرة واحدة. راجع [قنوات التطوير](./development-channels.md) لدلالات القنوات وملاحظات الإصدار. ملاحظة: في تثبيتات npm، تسجل البوابة تلميح تحديث عند بدء التشغيل (تتحقق من علامة القناة الحالية). يمكن تعطيله عبر `update.checkOnStart: false`.

### أداة التحديث التلقائي الأساسية (اختياري)

أداة التحديث التلقائي **معطلة افتراضيًا** وهي ميزة أساسية للبوابة (وليست إضافة).

```json
{
  "update": {
    "channel": "stable",
    "auto": {
      "enabled": true,
      "stableDelayHours": 6,
      "stableJitterHours": 12,
      "betaCheckIntervalHours": 1
    }
  }
}
```

السلوك:

-   `stable`: عند رؤية إصدار جديد، ينتظر OpenClaw `stableDelayHours` ثم يطبق تذبذبًا محددًا لكل تثبيت ضمن `stableJitterHours` (نشر تدريجي).
-   `beta`: يتحقق على وتيرة `betaCheckIntervalHours` (الافتراضي: كل ساعة) ويطبق عند توفر تحديث.
-   `dev`: لا يوجد تطبيق تلقائي؛ استخدم `openclaw update` يدويًا.

استخدم `openclaw update --dry-run` لمعاينة إجراءات التحديث قبل تمكين الأتمتة. ثم:

```bash
openclaw doctor
openclaw gateway restart
openclaw health
```

ملاحظات:

-   إذا كانت بوابةك تعمل كخدمة، فإن `openclaw gateway restart` مفضل على إنهاء عمليات PID.
-   إذا كنت مثبتًا على إصدار محدد، راجع "التراجع / التثبيت" أدناه.

## التحديث (openclaw update)

لـ **تثبيتات المصدر** (git checkout)، نفضل:

```bash
openclaw update
```

تشغل سير عمل تحديث آمن نسبيًا:

-   تتطلب شجرة عمل نظيفة.
-   تنتقل إلى القناة المحددة (علامة أو فرع).
-   تجلب + تعيد التأسيس (rebase) ضد المنبع المُهيأ (قناة dev).
-   تثبت التبعيات، تبني، تبني واجهة التحكم، وتشغل `openclaw doctor`.
-   تعيد تشغيل البوابة افتراضيًا (استخدم `--no-restart` للتخطي).

إذا قمت بالتثبيت عبر **npm/pnpm** (بدون بيانات وصفية لـ git)، فسيحاول `openclaw update` التحديث عبر مدير الحزم الخاص بك. إذا لم تتمكن من اكتشاف التثبيت، استخدم "التحديث (تثبيت عام)" بدلاً من ذلك.

## التحديث (واجهة التحكم / RPC)

تحتوي واجهة التحكم على **تحديث وإعادة تشغيل** (RPC: `update.run`). وهي:

1.  تشغل نفس سير عمل تحديث المصدر مثل `openclaw update` (git checkout فقط).
2.  تكتب علامة إعادة تشغيل مع تقرير منظم (ذيل stdout/stderr).
3.  تعيد تشغيل البوابة وترسل إشارة إلى الجلسة النشطة الأخيرة بالتقرير.

إذا فشلت عملية إعادة التأسيس (rebase)، فإن البوابة توقف وتعيد التشغيل دون تطبيق التحديث.

## التحديث (من المصدر)

من نسخة المستودع: مفضل:

```bash
openclaw update
```

يدوي (مكافئ تقريبًا):

```bash
git pull
pnpm install
pnpm build
pnpm ui:build # تثبت تبعيات واجهة المستخدم تلقائيًا في المرة الأولى
openclaw doctor
openclaw health
```

ملاحظات:

-   `pnpm build` مهم عندما تشغل الملف الثنائي `openclaw` المعبأ ([`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs)) أو تستخدم Node لتشغيل `dist/`.
-   إذا كنت تشغل من نسخة مستودع بدون تثبيت عام، استخدم `pnpm openclaw ...` لأوامر CLI.
-   إذا كنت تشغل مباشرة من TypeScript (`pnpm openclaw ...`)، فإن إعادة البناء عادة غير ضرورية، ولكن **هجرات الإعدادات لا تزال تنطبق** → شغل doctor.
-   التبديل بين التثبيت العام وتثبيت git سهل: قم بتثبيت النوع الآخر، ثم شغل `openclaw doctor` حتى تتم إعادة كتابة نقطة دخول خدمة البوابة إلى التثبيت الحالي.

## دائمًا شغل: openclaw doctor

Doctor هو أمر "التحديث الآمن". إنه متعمدًا بسيطًا: إصلاح + هجرة + تحذير. ملاحظة: إذا كنت على **تثبيت مصدر** (git checkout)، فإن `openclaw doctor` ستعرض تشغيل `openclaw update` أولاً. الأشياء النموذجية التي تقوم بها:

-   تهاجر مفاتيح إعدادات قديمة / مواقع ملفات إعدادات قديمة.
-   تدقق في سياسات DM وتحذر من إعدادات "مفتوحة" محفوفة بالمخاطر.
-   تتحقق من صحة البوابة ويمكن أن تعرض إعادة التشغيل.
-   تكتشف وتهاجر خدمات بوابات أقدم (launchd/systemd؛ مهام مجدولة قديمة) إلى خدمات OpenClaw الحالية.
-   على لينكس، تضمن استمرارية جلسة المستخدم في systemd (حتى تبقى البوابة بعد تسجيل الخروج).

التفاصيل: [Doctor](../gateway/doctor.md)

## بدء / إيقاف / إعادة تشغيل البوابة

واجهة سطر الأوامر (تعمل بغض النظر عن نظام التشغيل):

```bash
openclaw gateway status
openclaw gateway stop
openclaw gateway restart
openclaw gateway --port 18789
openclaw logs --follow
```

إذا كنت تحت إشراف:

-   macOS launchd (LaunchAgent مضمن في التطبيق): `launchctl kickstart -k gui/$UID/ai.openclaw.gateway` (استخدم `ai.openclaw.`؛ القديم `com.openclaw.*` لا يزال يعمل)
-   خدمة مستخدم systemd على لينكس: `systemctl --user restart openclaw-gateway[-].service`
-   Windows (WSL2): `systemctl --user restart openclaw-gateway[-].service`
    -   تعمل `launchctl`/`systemctl` فقط إذا كانت الخدمة مثبتة؛ وإلا شغل `openclaw gateway install`.

دليل التشغيل + تسميات الخدمة الدقيقة: [دليل تشغيل البوابة](../gateway.md)

## التراجع / التثبيت (عند حدوث مشكلة)

### تثبيت (تثبيت عام)

قم بتثبيت إصدار معروف جيد (استبدل `` بآخر إصدار يعمل):

```bash
npm i -g openclaw@<version>
```

```bash
pnpm add -g openclaw@<version>
```

نصيحة: لرؤية الإصدار المنشور الحالي، شغل `npm view openclaw version`. ثم أعد التشغيل وأعد تشغيل doctor:

```bash
openclaw doctor
openclaw gateway restart
```

### تثبيت (مصدر) حسب التاريخ

اختر commit من تاريخ معين (مثال: "حالة main اعتبارًا من 2026-01-01"):

```bash
git fetch origin
git checkout "$(git rev-list -n 1 --before=\"2026-01-01\" origin/main)"
```

ثم أعد تثبيت التبعيات + أعد التشغيل:

```bash
pnpm install
pnpm build
openclaw gateway restart
```

إذا أردت العودة إلى أحدث إصدار لاحقًا:

```bash
git checkout main
git pull
```

## إذا علقت

-   شغل `openclaw doctor` مرة أخرى واقرأ الناتج بعناية (غالبًا ما يخبرك بالإصلاح).
-   تحقق من: [استكشاف الأخطاء وإصلاحها](../gateway/troubleshooting.md)
-   اسأل في Discord: [https://discord.gg/clawd](https://discord.gg/clawd)

[Bun (تجريبي)](./bun.md)[دليل الهجرة](./migrating.md)