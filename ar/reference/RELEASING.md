

  ملاحظات الإصدار

  
# قائمة التحقق للإصدار

استخدم `pnpm` (Node 22+) من جذر المستودع. حافظ على شجرة العمل نظيفة قبل وضع العلامات/النشر.

## تشغيل المشغل

عندما يقول المشغل "أطلق الإصدار"، قم فورًا بتنفيذ فحص ما قبل الإطلاق هذا (بدون أسئلة إضافية إلا إذا تم حظرك):

-   اقرأ هذا المستند و `docs/platforms/mac/release.md`.
-   قم بتحميل المتغيرات البيئية من `~/.profile` وتأكد من تعيين `SPARKLE_PRIVATE_KEY_FILE` + متغيرات App Store Connect (يجب أن يكون SPARKLE\_PRIVATE\_KEY\_FILE موجودًا في `~/.profile`).
-   استخدم مفاتيح Sparkle من `~/Library/CloudStorage/Dropbox/Backup/Sparkle` إذا لزم الأمر.

1.  **الإصدار والبيانات الوصفية**

-   [ ]  رفع رقم الإصدار في `package.json` (مثال: `2026.1.29`).
-   [ ]  تشغيل `pnpm plugins:sync` لمحاذاة إصدارات حزم الإضافات + سجلات التغيير.
-   [ ]  تحديث سلاسل إصدار CLI في [`src/version.ts`](https://github.com/openclaw/openclaw/blob/main/src/version.ts) وعامل مستخدم Baileys في [`src/web/session.ts`](https://github.com/openclaw/openclaw/blob/main/src/web/session.ts).
-   [ ]  تأكيد البيانات الوصفية للحزمة (الاسم، الوصف، المستودع، الكلمات المفتاحية، الترخيص) وأن خريطة `bin` تشير إلى [`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs) لـ `openclaw`.
-   [ ]  إذا تغيرت التبعيات، قم بتشغيل `pnpm install` حتى يكون `pnpm-lock.yaml` محدثًا.

2.  **البناء والقطع الأثرية**

-   [ ]  إذا تغيرت مدخلات A2UI، قم بتشغيل `pnpm canvas:a2ui:bundle` وقم بتسجيل أي تحديث لـ [`src/canvas-host/a2ui/a2ui.bundle.js`](https://github.com/openclaw/openclaw/blob/main/src/canvas-host/a2ui/a2ui.bundle.js).
-   [ ]  `pnpm run build` (يعيد إنشاء `dist/`).
-   [ ]  التحقق من أن حزمة npm `files` تتضمن جميع مجلدات `dist/*` المطلوبة (خاصة `dist/node-host/**` و `dist/acp/**` لـ node headless + ACP CLI).
-   [ ]  تأكيد وجود `dist/build-info.json` وأنه يتضمن تجزئة `commit` المتوقعة (يستخدمها شعار CLI لعمليات تثبيت npm).
-   [ ]  اختياري: `npm pack --pack-destination /tmp` بعد البناء؛ افحص محتويات الأرشيف واحتفظ به في متناول اليد لإصدار GitHub (لا تقم **بتسجيله**).

3.  **سجل التغييرات والوثائق**

-   [ ]  تحديث `CHANGELOG.md` بأبرز التغييرات للمستخدمين (أنشئ الملف إذا كان مفقودًا)؛ حافظ على الإدخالات مرتبة تنازليًا حسب الإصدار.
-   [ ]  التأكد من أن الأمثلة/الخيارات في README تتطابق مع سلوك CLI الحالي (خاصة الأوامر أو الخيارات الجديدة).

4.  **التحقق**

-   [ ]  `pnpm build`
-   [ ]  `pnpm check`
-   [ ]  `pnpm test` (أو `pnpm test:coverage` إذا كنت بحاجة إلى إخراج تغطية)
-   [ ]  `pnpm release:check` (يتحقق من محتويات حزمة npm)
-   [ ]  `OPENCLAW_INSTALL_SMOKE_SKIP_NONROOT=1 pnpm test:install:smoke` (اختبار دخان التثبيت باستخدام Docker، المسار السريع؛ مطلوب قبل الإصدار)
    -   إذا كان الإصدار السابق المباشر من npm معروفًا بأنه معطل، قم بتعيين `OPENCLAW_INSTALL_SMOKE_PREVIOUS=<last-good-version>` أو `OPENCLAW_INSTALL_SMOKE_SKIP_PREVIOUS=1` لخطوة ما قبل التثبيت.
-   [ ]  (اختياري) اختبار دخان المثبت الكامل (يضيف تغطية لغير الجذر + CLI): `pnpm test:install:smoke`
-   [ ]  (اختياري) اختبار E2E للمثبت (Docker، يشغل `curl -fsSL https://openclaw.ai/install.sh | bash`، ثم الإعداد الأولي، ثم تشغيل استدعاءات الأدوات الحقيقية):
    -   `pnpm test:install:e2e:openai` (يتطلب `OPENAI_API_KEY`)
    -   `pnpm test:install:e2e:anthropic` (يتطلب `ANTHROPIC_API_KEY`)
    -   `pnpm test:install:e2e` (يتطلب كلا المفتاحين؛ يشغل كلا المزودين)
-   [ ]  (اختياري) فحص سريع لبوابة الويب إذا كانت تغييراتك تؤثر على مسارات الإرسال/الاستقبال.

5.  **تطبيق macOS (Sparkle)**

-   [ ]  بناء + توقيع تطبيق macOS، ثم ضغطه للتوزيع.
-   [ ]  إنشاء تغذية Sparkle appcast (ملاحظات HTML عبر [`scripts/make_appcast.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/make_appcast.sh)) وتحديث `appcast.xml`.
-   [ ]  احتفظ بملف zip للتطبيق (واختياريًا ملف dSYM المضغوط) جاهزًا لإرفاقه بإصدار GitHub.
-   [ ]  اتبع [إصدار macOS](../platforms/mac/release.md) للحصول على الأوامر الدقيقة ومتغيرات البيئة المطلوبة.
    -   يجب أن يكون `APP_BUILD` رقميًا + رتيبًا (بدون `-beta`) حتى تتمكن Sparkle من مقارنة الإصدارات بشكل صحيح.
    -   إذا كنت تقوم بالتصديق، استخدم ملف تعريف keychain `openclaw-notary` المنشأ من متغيرات بيئة App Store Connect API (انظر [إصدار macOS](../platforms/mac/release.md)).

6.  **النشر (npm)**

-   [ ]  تأكيد أن حالة git نظيفة؛ قم بالتسجيل والدفع حسب الحاجة.
-   [ ]  `npm login` (تحقق من المصادقة الثنائية) إذا لزم الأمر.
-   [ ]  `npm publish --access public` (استخدم `--tag beta` للإصدارات التجريبية).
-   [ ]  التحقق من السجل: `npm view openclaw version`, `npm view openclaw dist-tags`, و `npx -y openclaw@X.Y.Z --version` (أو `--help`).

### استكشاف الأخطاء وإصلاحها (ملاحظات من إصدار 2.0.0-beta2)

-   **npm pack/publish يتعطل أو ينتج أرشيفًا ضخمًا**: حزمة تطبيق macOS في `dist/OpenClaw.app` (وأرشيفات الإصدار) يتم تضمينها في الحزمة. الإصلاح عن طريق تضمين محتويات النشر عبر `package.json` `files` (قم بتضمين مجلدات فرعية من dist، الوثائق، المهارات؛ استبعد حزم التطبيقات). تأكد باستخدام `npm pack --dry-run` أن `dist/OpenClaw.app` غير مدرج.
-   **حلقة مصادقة npm الويب لعلامات التوزيع (dist-tags)**: استخدم المصادقة القديمة للحصول على مطالبة OTP:
    -   `NPM_CONFIG_AUTH_TYPE=legacy npm dist-tag add openclaw@X.Y.Z latest`
-   **فشل التحقق بـ `npx` مع `ECOMPROMISED: Lock compromised`**: أعد المحاولة بذاكرة تخزين مؤقت جديدة:
    -   `NPM_CONFIG_CACHE=/tmp/npm-cache-$(date +%s) npx -y openclaw@X.Y.Z --version`
-   **تحتاج العلامة إلى إعادة توجيه بعد إصلاح متأخر**: قم بتحديث قسري ودفع العلامة، ثم تأكد من أن مرفقات إصدار GitHub لا تزال مطابقة:
    -   `git tag -f vX.Y.Z && git push -f origin vX.Y.Z`

7.  **إصدار GitHub + تغذية التطبيق (appcast)**

-   [ ]  وضع علامة ودفع: `git tag vX.Y.Z && git push origin vX.Y.Z` (أو `git push --tags`).
-   [ ]  إنشاء/تحديث إصدار GitHub لـ `vX.Y.Z` بعنوان **`openclaw X.Y.Z`** (ليس مجرد العلامة)؛ يجب أن يتضمن النص **كامل** قسم سجل التغييرات لذلك الإصدار (أبرز التغييرات + التغييرات + الإصلاحات)، مضمنًا (بدون روابط عارية)، ويجب **ألا يكرر العنوان داخل النص**.
-   [ ]  إرفاق القطع الأثرية: أرشيف `npm pack` (اختياري)، `OpenClaw-X.Y.Z.zip`، و `OpenClaw-X.Y.Z.dSYM.zip` (إذا تم إنشاؤه).
-   [ ]  تسجيل `appcast.xml` المحدث ودفعه (تقرأ Sparkle التغذية من الفرع الرئيسي main).
-   [ ]  من دليل مؤقت نظيف (بدون `package.json`)، قم بتشغيل `npx -y openclaw@X.Y.Z send --help` للتأكد من عمل نقاط دخول التثبيت/CLI.
-   [ ]  الإعلان/مشاركة ملاحظات الإصدار.

## نطاق نشر الإضافات (npm)

نحن ننشر فقط **الإضافات الموجودة على npm** تحت النطاق `@openclaw/*`. الإضافات المضمنة غير الموجودة على npm تبقى **على القرص فقط** (لا تزال تُشحن في `extensions/**`). العملية لاشتقاق القائمة:

1.  `npm search @openclaw --json` والتقاط أسماء الحزم.
2.  المقارنة مع أسماء `extensions/*/package.json`.
3.  انشر فقط **التقاطع** (الموجود بالفعل على npm).

قائمة إضافات npm الحالية (قم بالتحديث حسب الحاجة):

-   @openclaw/bluebubbles
-   @openclaw/diagnostics-otel
-   @openclaw/discord
-   @openclaw/feishu
-   @openclaw/lobster
-   @openclaw/matrix
-   @openclaw/msteams
-   @openclaw/nextcloud-talk
-   @openclaw/nostr
-   @openclaw/voice-call
-   @openclaw/zalo
-   @openclaw/zalouser

يجب أن تشير ملاحظات الإصدار أيضًا إلى **الإضافات المضمنة الاختيارية الجديدة** التي **ليست مفعلة افتراضيًا** (مثال: `tlon`).

[الاعتمادات](./credits.md)[الاختبارات](./test.md)