

  نظرة عامة على التثبيت

  
# التفاصيل الداخلية للمثبت

يُرفق OpenClaw بثلاثة نصوص تثبيت، يتم تقديمها من `openclaw.ai`.

| النص | المنصة | الوظيفة |
| --- | --- | --- |
| [`install.sh`](#installsh) | macOS / Linux / WSL | يثبت Node إذا لزم الأمر، يثبت OpenClaw عبر npm (الافتراضي) أو git، ويمكنه تشغيل الإعداد الأولي. |
| [`install-cli.sh`](#install-clish) | macOS / Linux / WSL | يثبت Node + OpenClaw في بادئة محلية (`~/.openclaw`). لا يتطلب صلاحيات root. |
| [`install.ps1`](#installps1) | Windows (PowerShell) | يثبت Node إذا لزم الأمر، يثبت OpenClaw عبر npm (الافتراضي) أو git، ويمكنه تشغيل الإعداد الأولي. |

## أوامر سريعة

 

> **ℹ️** إذا نجح التثبيت ولكن لم يتم العثور على `openclaw` في طرفية جديدة، راجع [استكشاف أخطاء Node.js](./node.md#troubleshooting).

* * *

## install.sh

> **💡** موصى به لمعظم عمليات التثبيت التفاعلية على macOS/Linux/WSL.

### التدفق (install.sh)

### الخطوة 1: اكتشاف نظام التشغيل

يدعم macOS و Linux (بما في ذلك WSL). إذا تم اكتشاف macOS، يقوم بتثبيت Homebrew إذا كان مفقودًا.

### الخطوة 2: التأكد من Node.js 22+

يفحص إصدار Node ويقوم بتثبيت Node 22 إذا لزم الأمر (Homebrew على macOS، نصوص إعداد NodeSource على Linux apt/dnf/yum).

### الخطوة 3: التأكد من Git

يقوم بتثبيت Git إذا كان مفقودًا.

### الخطوة 4: تثبيت OpenClaw

-   طريقة `npm` (الافتراضية): تثبيت npm عام
-   طريقة `git`: استنساخ/تحديث المستودع، تثبيت التبعيات باستخدام pnpm، البناء، ثم تثبيت الغلاف في `~/.local/bin/openclaw`

### الخطوة 5: مهام ما بعد التثبيت

-   يشغل `openclaw doctor --non-interactive` عند الترقية وعند التثبيت عبر git (بأقصى جهد ممكن)
-   يحاول تشغيل الإعداد الأولي عند الاقتضاء (TTY متاح، الإعداد الأولي غير معطل، وتمر فحوصات التمهيد/الإعداد)
-   يضبط الافتراضي `SHARP_IGNORE_GLOBAL_LIBVIPS=1`

### اكتشاف نسخة المصدر

إذا تم التشغيل داخل نسخة مستنسخة من OpenClaw (`package.json` + `pnpm-workspace.yaml`)، يعرض النص خيارين:

-   استخدام النسخة المستنسخة (`git`)، أو
-   استخدام التثبيت العام (`npm`)

إذا لم يكن TTY متاحًا ولم يتم تعيين طريقة تثبيت، فإنه يستخدم `npm` بشكل افتراضي ويصدر تحذيرًا. يخرج النص برمز `2` لاختيار طريقة غير صالحة أو قيم غير صالحة لـ `--install-method`.

### أمثلة (install.sh)

 

| العلم | الوصف |
| --- | --- |
| `--install-method npm\|git` | اختر طريقة التثبيت (الافتراضي: `npm`). اسم بديل: `--method` |
| `--npm` | اختصار لطريقة npm |
| `--git` | اختصار لطريقة git. اسم بديل: `--github` |
| `--version <version\|dist-tag>` | إصدار npm أو dist-tag (الافتراضي: `latest`) |
| `--beta` | استخدم dist-tag بيتا إذا كان متاحًا، وإلا فاستخدم `latest` |
| `--git-dir ` | دليل النسخة المستنسخة (الافتراضي: `~/openclaw`). اسم بديل: `--dir` |
| `--no-git-update` | تخطي `git pull` للنسخة المستنسخة الموجودة |
| `--no-prompt` | تعطيل المطالبات |
| `--no-onboard` | تخطي الإعداد الأولي |
| `--onboard` | تمكين الإعداد الأولي |
| `--dry-run` | طباعة الإجراءات دون تطبيق التغييرات |
| `--verbose` | تمكين إخراج التصحيح (`set -x`، سجلات npm بمستوى notice) |
| `--help` | عرض الاستخدام (`-h`) |

| المتغير | الوصف |
| --- | --- |
| `OPENCLAW_INSTALL_METHOD=git\|npm` | طريقة التثبيت |
| `OPENCLAW_VERSION=latest\|next\|` | إصدار npm أو dist-tag |
| `OPENCLAW_BETA=0\|1` | استخدام بيتا إذا كان متاحًا |
| `OPENCLAW_GIT_DIR=` | دليل النسخة المستنسخة |
| `OPENCLAW_GIT_UPDATE=0\|1` | تبديل تحديثات git |
| `OPENCLAW_NO_PROMPT=1` | تعطيل المطالبات |
| `OPENCLAW_NO_ONBOARD=1` | تخطي الإعداد الأولي |
| `OPENCLAW_DRY_RUN=1` | وضع التشغيل التجريبي |
| `OPENCLAW_VERBOSE=1` | وضع التصحيح |
| `OPENCLAW_NPM_LOGLEVEL=error\|warn\|notice` | مستوى سجل npm |
| `SHARP_IGNORE_GLOBAL_LIBVIPS=0\|1` | التحكم في سلوك sharp/libvips (الافتراضي: `1`) |

* * *

## install-cli.sh

> **ℹ️** مصمم للبيئات التي تريد فيها وضع كل شيء تحت بادئة محلية (الافتراضي `~/.openclaw`) دون الاعتماد على Node النظام.

### التدفق (install-cli.sh)

### الخطوة 1: تثبيت وقت تشغيل Node محلي

يقوم بتنزيل أرشيف Node (الافتراضي `22.22.0`) إلى `/tools/node-v` والتحقق من SHA-256.

### الخطوة 2: التأكد من Git

إذا كان Git مفقودًا، يحاول التثبيت عبر apt/dnf/yum على Linux أو Homebrew على macOS.

### الخطوة 3: تثبيت OpenClaw تحت البادئة

يقوم بالتثبيت باستخدام npm مع `--prefix `، ثم يكتب الغلاف في `/bin/openclaw`.

### أمثلة (install-cli.sh)

 

| العلم | الوصف |
| --- | --- |
| `--prefix ` | بادئة التثبيت (الافتراضي: `~/.openclaw`) |
| `--version ` | إصدار OpenClaw أو dist-tag (الافتراضي: `latest`) |
| `--node-version ` | إصدار Node (الافتراضي: `22.22.0`) |
| `--json` | إصدار أحداث NDJSON |
| `--onboard` | تشغيل `openclaw onboard` بعد التثبيت |
| `--no-onboard` | تخطي الإعداد الأولي (الافتراضي) |
| `--set-npm-prefix` | على Linux، فرض بادئة npm إلى `~/.npm-global` إذا كانت البادئة الحالية غير قابلة للكتابة |
| `--help` | عرض الاستخدام (`-h`) |

| المتغير | الوصف |
| --- | --- |
| `OPENCLAW_PREFIX=` | بادئة التثبيت |
| `OPENCLAW_VERSION=` | إصدار OpenClaw أو dist-tag |
| `OPENCLAW_NODE_VERSION=` | إصدار Node |
| `OPENCLAW_NO_ONBOARD=1` | تخطي الإعداد الأولي |
| `OPENCLAW_NPM_LOGLEVEL=error\|warn\|notice` | مستوى سجل npm |
| `OPENCLAW_GIT_DIR=` | مسار البحث لتنظيف الإرث (يستخدم عند إزالة النسخة المستنسخة القديمة للوحدة الفرعية `Peekaboo`) |
| `SHARP_IGNORE_GLOBAL_LIBVIPS=0\|1` | التحكم في سلوك sharp/libvips (الافتراضي: `1`) |

* * *

## install.ps1

### التدفق (install.ps1)

### الخطوة 1: التأكد من PowerShell + بيئة Windows

يتطلب PowerShell 5+.

### الخطوة 2: التأكد من Node.js 22+

إذا كان مفقودًا، يحاول التثبيت عبر winget، ثم Chocolatey، ثم Scoop.

### الخطوة 3: تثبيت OpenClaw

-   طريقة `npm` (الافتراضية): تثبيت npm عام باستخدام `-Tag` المحدد
-   طريقة `git`: استنساخ/تحديث المستودع، تثبيت/بناء باستخدام pnpm، وتثبيت الغلاف في `%USERPROFILE%\.local\bin\openclaw.cmd`

### الخطوة 4: مهام ما بعد التثبيت

يضيف دليل bin المطلوب إلى مسار PATH للمستخدم عندما يكون ذلك ممكنًا، ثم يشغل `openclaw doctor --non-interactive` عند الترقية وعند التثبيت عبر git (بأقصى جهد ممكن).

### أمثلة (install.ps1)

 

| العلم | الوصف |
| --- | --- |
| `-InstallMethod npm\|git` | طريقة التثبيت (الافتراضي: `npm`) |
| `-Tag ` | dist-tag لـ npm (الافتراضي: `latest`) |
| `-GitDir ` | دليل النسخة المستنسخة (الافتراضي: `%USERPROFILE%\openclaw`) |
| `-NoOnboard` | تخطي الإعداد الأولي |
| `-NoGitUpdate` | تخطي `git pull` |
| `-DryRun` | طباعة الإجراءات فقط |

| المتغير | الوصف |
| --- | --- |
| `OPENCLAW_INSTALL_METHOD=git\|npm` | طريقة التثبيت |
| `OPENCLAW_GIT_DIR=` | دليل النسخة المستنسخة |
| `OPENCLAW_NO_ONBOARD=1` | تخطي الإعداد الأولي |
| `OPENCLAW_GIT_UPDATE=0` | تعطيل git pull |
| `OPENCLAW_DRY_RUN=1` | وضع التشغيل التجريبي |

 

> **ℹ️** إذا تم استخدام `-InstallMethod git` وكان Git مفقودًا، يخرج النص ويطبع رابط Git for Windows.

* * *

## CI والأتمتة

استخدم أعلام/متغيرات بيئة غير تفاعلية لتشغيلات يمكن التنبؤ بها. 

* * *

## استكشاف الأخطاء وإصلاحها

Git مطلوب لطريقة التثبيت `git`. بالنسبة لتثبيتات `npm`، لا يزال يتم فحص/تثبيت Git لتجنب فشل `spawn git ENOENT` عندما تستخدم التبعيات عناوين URL لـ git.

بعض إعدادات Linux تشير إلى البادئة العامة لـ npm إلى مسارات مملوكة لـ root. يمكن لـ `install.sh` تبديل البادئة إلى `~/.npm-global` وإلحاق تصديرات PATH بملفات shell rc (عند وجود تلك الملفات).

تضبط النصوص الافتراضي `SHARP_IGNORE_GLOBAL_LIBVIPS=1` لتجنب بناء sharp مقابل libvips النظام. للتجاوز:

```
SHARP_IGNORE_GLOBAL_LIBVIPS=0 curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash
```

قم بتثبيت Git for Windows، أعد فتح PowerShell، أعد تشغيل المثبت.

شغل `npm config get prefix` وأضف ذلك الدليل إلى مسار PATH للمستخدم (لا حاجة للاحقة `\bin` على Windows)، ثم أعد فتح PowerShell.

لا يعرض `install.ps1` حاليًا مفتاح `-Verbose`. استخدم تتبع PowerShell للتشخيص على مستوى النص:

```powershell
Set-PSDebug -Trace 1
& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -NoOnboard
Set-PSDebug -Trace 0
```

عادة ما تكون مشكلة في PATH. راجع [استكشاف أخطاء Node.js](./node.md#troubleshooting).

[التثبيت](../install.md)[Docker](./docker.md)