

  وقت تشغيل Node

  
# Node.js

يتطلب OpenClaw **Node 22 أو أحدث**. سيكتشف [سكريبت التثبيت](../install.md#install-methods) ويقوم بتثبيت Node تلقائيًا — هذه الصفحة مخصصة عندما تريد إعداد Node بنفسك والتأكد من أن كل شيء موصل بشكل صحيح (الإصدارات، PATH، التثبيتات العامة).

## تحقق من إصدارك

```bash
node -v
```

إذا أظهر هذا `v22.x.x` أو أعلى، فأنت على ما يرام. إذا لم يكن Node مثبتًا أو كان الإصدار قديمًا جدًا، اختر طريقة تثبيت من الأسفل.

## تثبيت Node

 

تتيح لك مديري الإصدارات التبديل بين إصدارات Node بسهولة. الخيارات الشائعة:

-   [**fnm**](https://github.com/Schniz/fnm) — سريع، يعمل على جميع المنصات
-   [**nvm**](https://github.com/nvm-sh/nvm) — مستخدم على نطاق واسع على macOS/Linux
-   [**mise**](https://mise.jdx.dev/) — متعدد اللغات (Node، Python، Ruby، إلخ)

مثال باستخدام fnm:

```bash
fnm install 22
fnm use 22
```

تأكد من تهيئة مدير الإصدارات الخاص بك في ملف بدء تشغيل shell الخاص بك (`~/.zshrc` أو `~/.bashrc`). إذا لم يكن كذلك، فقد لا يتم العثور على `openclaw` في جلسات الطرفية الجديدة لأن PATH لن يتضمن دليل bin الخاص بـ Node.

## استكشاف الأخطاء وإصلاحها

### openclaw: command not found

هذا يعني دائمًا تقريبًا أن الدليل الثنائي العام لـ npm ليس على PATH الخاص بك. 

### الخطوة 1: ابحث عن البادئة العامة لـ npm

```bash
npm prefix -g
```

### الخطوة 2: تحقق مما إذا كانت على PATH الخاص بك

```bash
echo "$PATH"
```

ابحث عن `<npm-prefix>/bin` (macOS/Linux) أو `<npm-prefix>` (Windows) في الناتج.

### الخطوة 3: أضفه إلى ملف بدء تشغيل shell الخاص بك

```bash
export PATH="$(npm prefix -g)/bin:$PATH"
```

أضف ناتج `npm prefix -g` إلى PATH الخاص بالنظام عبر الإعدادات → النظام → متغيرات البيئة.

### أخطاء صلاحيات على npm install -g (Linux)

إذا رأيت أخطاء `EACCES`، قم بتبديل البادئة العامة لـ npm إلى دليل يمكن للمستخدم الكتابة فيه:

```bash
mkdir -p "$HOME/.npm-global"
npm config set prefix "$HOME/.npm-global"
export PATH="$HOME/.npm-global/bin:$PATH"
```

أضف سطر `export PATH=...` إلى ملف `~/.bashrc` أو `~/.zshrc` الخاص بك لجعله دائمًا.

[Diagnostics Flags](../diagnostics/flags.md)[Session Management Deep Dive](../reference/session-management-compaction.md)