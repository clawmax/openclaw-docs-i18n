

  الأمان والتجزئة

  
# التجزئة

يمكن لـ OpenClaw تشغيل **الأدوات داخل حاويات Docker** لتقليل نطاق التأثير. هذا **اختياري** ويتم التحكم به عبر التهيئة (`agents.defaults.sandbox` أو `agents.list[].sandbox`). إذا كانت التجزئة معطلة، تعمل الأدوات على المضيف. تبقى البوابة على المضيف؛ بينما يتم تنفيذ الأداة في تجزئة معزولة عند التمكين. هذا ليس حدًا أمنيًا مثاليًا، لكنه يحد بشكل ملموس من وصول نظام الملفات والعمليات عندما يقوم النموذج بشيء غير ذكي.

## ما الذي يتم تجزئته

-   تنفيذ الأداة (`exec`, `read`, `write`, `edit`, `apply_patch`, `process`, إلخ).
-   متصفح مجزأ اختياري (`agents.defaults.sandbox.browser`).
    -   افتراضيًا، يبدأ متصفح التجزئة تلقائيًا (يضمن الوصول إلى CDP) عندما تحتاجه أداة المتصفح. قم بالتهيئة عبر `agents.defaults.sandbox.browser.autoStart` و `agents.defaults.sandbox.browser.autoStartTimeoutMs`.
    -   افتراضيًا، تستخدم حاويات متصفح التجزئة شبكة Docker مخصصة (`openclaw-sandbox-browser`) بدلاً من الشبكة العامة `bridge`. قم بالتهيئة باستخدام `agents.defaults.sandbox.browser.network`.
    -   اختياريًا، `agents.defaults.sandbox.browser.cdpSourceRange` يقيد وصول CDP من حافة الحاوية باستخدام قائمة سماح CIDR (على سبيل المثال `172.21.0.1/32`).
    -   الوصول إلى مراقب noVNC محمي بكلمة مرور افتراضيًا؛ يصدر OpenClaw عنوان URL رمزيًا قصير العمر يقدم صفحة تمهيدية محلية ويفتح noVNC مع كلمة المرور في جزء العنوان (وليس في سجلات الاستعلام/الرأس).
    -   `agents.defaults.sandbox.browser.allowHostControl` يسمح لجلسات التجزئة باستهداف متصفح المضيف صراحةً.
    -   قوائم السماح الاختيارية تتحكم في `target: "custom"`: `allowedControlUrls`, `allowedControlHosts`, `allowedControlPorts`.

غير مجزأ:

-   عملية البوابة نفسها.
-   أي أداة مسموح لها صراحةً بالعمل على المضيف (مثل `tools.elevated`).
    -   **التنفيذ المرفوع يعمل على المضيف ويتجاوز التجزئة.**
    -   إذا كانت التجزئة معطلة، فإن `tools.elevated` لا يغير التنفيذ (أصلاً على المضيف). راجع [الوضع المرفوع](../tools/elevated.md).

## الأوضاع

`agents.defaults.sandbox.mode` يتحكم في **متى** يتم استخدام التجزئة:

-   `"off"`: لا تجزئة.
-   `"non-main"`: جزئ **الجلسات غير الرئيسية** فقط (الافتراضي إذا كنت تريد الدردشات العادية على المضيف).
-   `"all"`: كل جلسة تعمل في تجزئة. ملاحظة: `"non-main"` يعتمد على `session.mainKey` (الافتراضي `"main"`)، وليس معرف الوكيل. جلسات المجموعة/القناة تستخدم مفاتيحها الخاصة، لذا تعتبر غير رئيسية وسيتم تجزئتها.

## النطاق

`agents.defaults.sandbox.scope` يتحكم في **عدد الحاويات** التي يتم إنشاؤها:

-   `"session"` (الافتراضي): حاوية واحدة لكل جلسة.
-   `"agent"`: حاوية واحدة لكل وكيل.
-   `"shared"`: حاوية واحدة مشتركة بين جميع جلسات التجزئة.

## وصول مساحة العمل

`agents.defaults.sandbox.workspaceAccess` يتحكم في **ما يمكن للتجزئة رؤيته**:

-   `"none"` (الافتراضي): ترى الأدوات مساحة عمل مجزأة تحت `~/.openclaw/sandboxes`.
-   `"ro"`: يربط مساحة عمل الوكيل للقراءة فقط في `/agent` (يعطل `write`/`edit`/`apply_patch`).
-   `"rw"`: يربط مساحة عمل الوكيل للقراءة/الكتابة في `/workspace`.

يتم نسخ الوسائط الواردة إلى مساحة عمل التجزئة النشطة (`media/inbound/*`). ملاحظة حول المهارات: أداة `read` متجذرة في التجزئة. مع `workspaceAccess: "none"`، يقوم OpenClaw بعكس المهارات المؤهلة إلى مساحة عمل التجزئة (`.../skills`) حتى يمكن قراءتها. مع `"rw"`، يمكن قراءة مهارات مساحة العمل من `/workspace/skills`.

## نقاط الربط المخصصة

`agents.defaults.sandbox.docker.binds` يربط أدلة مضيف إضافية داخل الحاوية. التنسيق: `host:container:mode` (مثال: `"/home/user/source:/source:rw"`). يتم **دمج** نقاط الربط العامة ونقاط الربط لكل وكيل (وليس استبدالها). تحت `scope: "shared"`، يتم تجاهل نقاط الربط لكل وكيل. `agents.defaults.sandbox.browser.binds` يربط أدلة مضيف إضافية داخل حاوية **متصفح التجزئة** فقط.

-   عند تعيينها (بما في ذلك `[]`)، تحل محل `agents.defaults.sandbox.docker.binds` لحاوية المتصفح.
-   عند حذفها، تعود حاوية المتصفح إلى `agents.defaults.sandbox.docker.binds` (للتوافق مع الإصدارات السابقة).

مثال (مصدر للقراءة فقط + دليل بيانات إضافي):

```json
{
  agents: {
    defaults: {
      sandbox: {
        docker: {
          binds: ["/home/user/source:/source:ro", "/var/data/myapp:/data:ro"],
        },
      },
    },
    list: [
      {
        id: "build",
        sandbox: {
          docker: {
            binds: ["/mnt/cache:/cache:rw"],
          },
        },
      },
    ],
  },
}
```

ملاحظات أمنية:

-   نقاط الربط تتجاوز نظام ملفات التجزئة: تعرض مسارات المضيف بأي وضع قمت بتعيينه (`:ro` أو `:rw`).
-   يمنع OpenClaw مصادر الربط الخطرة (على سبيل المثال: `docker.sock`, `/etc`, `/proc`, `/sys`, `/dev`, ونقاط الربط الأصلية التي قد تعرضها).
-   يجب أن تكون نقاط الربط الحساسة (الأسرار، مفاتيح SSH، بيانات اعتماد الخدمة) `:ro` إلا إذا كانت مطلوبة تمامًا.
-   اجمع مع `workspaceAccess: "ro"` إذا كنت تحتاج فقط وصول قراءة لمساحة العمل؛ تبقى أوضاع الربط مستقلة.
-   راجع [التجزئة مقابل سياسة الأداة مقابل المرفوع](./sandbox-vs-tool-policy-vs-elevated.md) لمعرفة كيفية تفاعل نقاط الربط مع سياسة الأداة والتنفيذ المرفوع.

## الصور + الإعداد

الصورة الافتراضية: `openclaw-sandbox:bookworm-slim` قم ببنائها مرة واحدة:

```
scripts/sandbox-setup.sh
```

ملاحظة: الصورة الافتراضية **لا** تتضمن Node. إذا كانت المهارة تحتاج Node (أو بيئات تشغيل أخرى)، إما أن تصنع صورة مخصصة أو تقوم بالتثبيت عبر `sandbox.docker.setupCommand` (يتطلب خروج الشبكة + جذر قابل للكتابة + مستخدم root). إذا كنت تريد صورة تجزئة أكثر وظيفية مع أدوات شائعة (مثل `curl`, `jq`, `nodejs`, `python3`, `git`)، قم بالبناء:

```
scripts/sandbox-common-setup.sh
```

ثم عيّن `agents.defaults.sandbox.docker.image` إلى `openclaw-sandbox-common:bookworm-slim`. صورة متصفح التجزئة:

```
scripts/sandbox-browser-setup.sh
```

افتراضيًا، تعمل حاويات التجزئة **بدون شبكة**. تجاوز ذلك باستخدام `agents.defaults.sandbox.docker.network`. تطبق صورة متصفح التجزئة المضمنة أيضًا إعدادات بدء تشغيل Chromium محافظة لأحمال العمل المعتمدة على الحاويات. تتضمن إعدادات الحاوية الحالية الافتراضية:

-   `--remote-debugging-address=127.0.0.1`
-   `--remote-debugging-port=<مشتق من OPENCLAW_BROWSER_CDP_PORT>`
-   `--user-data-dir=${HOME}/.chrome`
-   `--no-first-run`
-   `--no-default-browser-check`
-   `--disable-3d-apis`
-   `--disable-gpu`
-   `--disable-dev-shm-usage`
-   `--disable-background-networking`
-   `--disable-extensions`
-   `--disable-features=TranslateUI`
-   `--disable-breakpad`
-   `--disable-crash-reporter`
-   `--disable-software-rasterizer`
-   `--no-zygote`
-   `--metrics-recording-only`
-   `--renderer-process-limit=2`
-   `--no-sandbox` و `--disable-setuid-sandbox` عند تمكين `noSandbox`.
-   علامات التحصين الثلاثة للرسومات (`--disable-3d-apis`, `--disable-software-rasterizer`, `--disable-gpu`) اختيارية ومفيدة عندما تفتقر الحاويات إلى دعم GPU. عيّن `OPENCLAW_BROWSER_DISABLE_GRAPHICS_FLAGS=0` إذا كان حمل عملك يتطلب WebGL أو ميزات متصفح/ثلاثية الأبعاد أخرى.
-   `--disable-extensions` مفعلة افتراضيًا ويمكن تعطيلها بـ `OPENCLAW_BROWSER_DISABLE_EXTENSIONS=0` للعمليات التي تعتمد على الامتدادات.
-   `--renderer-process-limit=2` يتم التحكم بها بواسطة `OPENCLAW_BROWSER_RENDERER_PROCESS_LIMIT=`، حيث `0` تحتفظ بإعداد Chromium الافتراضي.

إذا كنت تحتاج إلى ملف تعريف وقت تشغيل مختلف، استخدم صورة متصفح مخصصة ووفر نقطة دخولك الخاصة. لملفات تعريف Chromium المحلية (غير المعتمدة على الحاوية)، استخدم `browser.extraArgs` لإلحاق علامات بدء تشغيل إضافية. الإعدادات الأمنية الافتراضية:

-   `network: "host"` محظور.
-   `network: "container:"` محظور افتراضيًا (خطر تجاوز انضمام النطاق).
-   تجاوز طارئ: `agents.defaults.sandbox.docker.dangerouslyAllowContainerNamespaceJoin: true`.

تثبيتات Docker والبوابة المعتمدة على الحاويات موجودة هنا: [Docker](../install/docker.md) لنشرات بوابة Docker، يمكن لـ `docker-setup.sh` تهيئة إعدادات التجزئة. عيّن `OPENCLAW_SANDBOX=1` (أو `true`/`yes`/`on`) لتمكين هذا المسار. يمكنك تجاوز موقع المقبس باستخدام `OPENCLAW_DOCKER_SOCKET`. الإعداد الكامل ومرجع البيئة: [Docker](../install/docker.md#enable-agent-sandbox-for-docker-gateway-opt-in).

## setupCommand (إعداد الحاوية لمرة واحدة)

`setupCommand` يعمل **مرة واحدة** بعد إنشاء حاوية التجزئة (ليس في كل تشغيل). يتم تنفيذه داخل الحاوية عبر `sh -lc`. المسارات:

-   عام: `agents.defaults.sandbox.docker.setupCommand`
-   لكل وكيل: `agents.list[].sandbox.docker.setupCommand`

المزالق الشائعة:

-   الافتراضي `docker.network` هو `"none"` (لا خروج)، لذا ستفشل تثبيتات الحزم.
-   `docker.network: "container:"` يتطلب `dangerouslyAllowContainerNamespaceJoin: true` وهو للطوارئ فقط.
-   `readOnlyRoot: true` يمنع الكتابة؛ عيّن `readOnlyRoot: false` أو اصنع صورة مخصصة.
-   `user` يجب أن يكون root لتثبيتات الحزم (احذف `user` أو عيّن `user: "0:0"`).
-   تنفيذ التجزئة **لا** يرث `process.env` للمضيف. استخدم `agents.defaults.sandbox.docker.env` (أو صورة مخصصة) لمفاتيح API للمهارات.

## سياسة الأداة + مخارج الطوارئ

لا تزال سياسات السماح/الرفض للأدوات تنطبق قبل قواعد التجزئة. إذا تم رفض أداة عالميًا أو لكل وكيل، فإن التجزئة لا تعيدها. `tools.elevated` هو مخرج طوارئ صريح يعمل `exec` على المضيف. توجيهات `/exec` تنطبق فقط للمرسلين المصرح لهم وتستمر لكل جلسة؛ لتعطيل `exec` بشكل صارم، استخدم سياسة رفض الأداة (انظر [التجزئة مقابل سياسة الأداة مقابل المرفوع](./sandbox-vs-tool-policy-vs-elevated.md)). التصحيح:

-   استخدم `openclaw sandbox explain` لفحص وضع التجزئة الفعال، وسياسة الأداة، ومفاتيح التهيئة للإصلاح.
-   راجع [التجزئة مقابل سياسة الأداة مقابل المرفوع](./sandbox-vs-tool-policy-vs-elevated.md) لنموذج "لماذا تم حظر هذا؟". حافظ على إغلاقه.

## تجاوزات متعددة الوكلاء

يمكن لكل وكيل تجاوز التجزئة + الأدوات: `agents.list[].sandbox` و `agents.list[].tools` (بالإضافة إلى `agents.list[].tools.sandbox.tools` لسياسة أداة التجزئة). راجع [تجزئة وأدوات متعددة الوكلاء](../tools/multi-agent-sandbox-tools.md) للأسبقية.

## مثال تمكين بسيط

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none",
      },
    },
  },
}
```

## وثائق ذات صلة

-   [تهيئة التجزئة](./configuration.md#agentsdefaults-sandbox)
-   [تجزئة وأدوات متعددة الوكلاء](../tools/multi-agent-sandbox-tools.md)
-   [الأمان](./security.md)

[الأمان](./security.md)[التجزئة مقابل سياسة الأداة مقابل المرفوع](./sandbox-vs-tool-policy-vs-elevated.md)