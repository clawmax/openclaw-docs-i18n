

  أوامر CLI

  
# plugins

إدارة إضافات/امتدادات البوابة (يتم تحميلها داخل العملية). ذات صلة:

-   نظام الإضافات: [الإضافات](../tools/plugin.md)
-   بيان الإضافة + المخطط: [بيان الإضافة](../plugins/manifest.md)
-   تعزيز الأمان: [الأمان](../gateway/security.md)

## الأوامر

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins uninstall <id>
openclaw plugins doctor
openclaw plugins update <id>
openclaw plugins update --all
```

الإضافات المضمنة تأتي مع OpenClaw ولكنها تبدأ معطلة. استخدم `plugins enable` لتفعيلها. يجب أن تقدم جميع الإضافات ملف `openclaw.plugin.json` مع مخطط JSON مضمن (`configSchema`، حتى لو كان فارغًا). يؤدي فقدان أو عدم صلاحية البيانات الوصفية أو المخططات إلى منع تحميل الإضافة وفشل التحقق من صحة التكوين.

### التثبيت

```bash
openclaw plugins install <path-or-spec>
openclaw plugins install <npm-spec> --pin
```

ملاحظة أمنية: عامل تثبيت الإضافات كتشغيل للكود. يُفضل استخدام الإصدارات المثبتة. مواصفات Npm هي **للسجل فقط** (اسم الحزمة + **الإصدار المحدد** الاختياري أو **علامة التوزيع**). يتم رفض مواصفات Git/URL/الملف ونطاقات semver. يتم تشغيل تثبيت التبعيات باستخدام `--ignore-scripts` للسلامة. تبقى المواصفات العارية و `@latest` على المسار المستقر. إذا قام npm بحل أي منهما إلى إصدار أولي، يتوقف OpenClaw ويطلب منك الموافقة صراحة باستخدام علامة إصدار أولي مثل `@beta`/`@rc` أو إصدار أولي محدد مثل `@1.2.3-beta.4`. إذا تطابق مواصفات تثبيت عارية مع معرف إضافة مضمنة (على سبيل المثال `diffs`)، يقوم OpenClaw بتثبيت الإضافة المضمنة مباشرة. لتثبيت حزمة npm بنفس الاسم، استخدم مواصفات محددة بالنطاق (على سبيل المثال `@scope/diffs`). الأرشيفات المدعومة: `.zip`, `.tgz`, `.tar.gz`, `.tar`. استخدم `--link` لتجنب نسخ دليل محلي (يضاف إلى `plugins.load.paths`):

```bash
openclaw plugins install -l ./my-plugin
```

استخدم `--pin` في عمليات تثبيت npm لحفظ المواصفات المحددة المحلولة (`name@version`) في `plugins.installs` مع الحفاظ على السلوك الافتراضي غير المثبت.

### الإزالة

```bash
openclaw plugins uninstall <id>
openclaw plugins uninstall <id> --dry-run
openclaw plugins uninstall <id> --keep-files
```

يزيل `uninstall` سجلات الإضافة من `plugins.entries`, `plugins.installs`، قائمة السماح بالإضافة، وإدخالات `plugins.load.paths` المرتبطة عند الاقتضاء. بالنسبة للإضافات النشطة في الذاكرة، تتم إعادة تعيين فتحة الذاكرة إلى `memory-core`. بشكل افتراضي، تقوم الإزالة أيضًا بإزالة دليل تثبيت الإضافة تحت دليل الامتدادات الجذر للحالة النشطة (`$OPENCLAW_STATE_DIR/extensions/`). استخدم `--keep-files` للاحتفاظ بالملفات على القرص. `--keep-config` مدعوم كاسم مستعار مهمل لـ `--keep-files`.

### التحديث

```bash
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins update <id> --dry-run
```

تطبق التحديثات فقط على الإضافات المثبتة من npm (المتتبعة في `plugins.installs`). عندما يكون هناك تجزئة سلامة مخزنة ويتغير تجزئة الأداة التي تم جلبها، يطبع OpenClaw تحذيرًا ويطلب التأكيد قبل المتابعة. استخدم `--yes` العام لتجاوز المطالبات في عمليات التشغيل غير التفاعلية أو في CI.

[pairing](./pairing.md)[qr](./qr.md)