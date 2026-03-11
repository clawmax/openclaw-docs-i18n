

  المتصفح

  
# المتصفح (المدار من OpenClaw)

يمكن لـ OpenClaw تشغيل **ملف تعريف مخصص لـ Chrome/Brave/Edge/Chromium** يتحكم فيه الوكيل. إنه معزول عن متصفحك الشخصي ويتم إدارته من خلال خدمة تحكم محلية صغيرة داخل البوابة (حلقة العودة فقط). منظور المبتدئين:

-   فكر فيه على أنه **متصفح منفصل مخصص للوكيل فقط**.
-   ملف تعريف `openclaw` **لا** يلمس ملف تعريف متصفحك الشخصي.
-   يمكن للوكيل **فتح علامات تبويب، وقراءة الصفحات، والنقر، والكتابة** في مسار آمن.
-   يستخدم ملف تعريف `chrome` الافتراضي **متصفح Chromium الافتراضي للنظام** عبر ناقل الامتداد؛ انتقل إلى `openclaw` لاستخدام المتصفح المعزول المدار.

## ما تحصل عليه

-   ملف تعريف متصفح منفصل باسم **openclaw** (لون برتقالي مائل افتراضيًا).
-   تحكم حتمي في علامات التبويب (عرض/فتح/تركيز/إغلاق).
-   إجراءات الوكيل (نقر/كتابة/سحب/تحديد)، لقطات، صور شاشة، ملفات PDF.
-   دعم اختياري لملفات تعريف متعددة (`openclaw`, `work`, `remote`, …).

هذا المتصفح **ليس** متصفحك اليومي. إنه سطح آمن ومعزول لأتمتة الوكيل والتحقق منه.

## البدء السريع

```bash
openclaw browser --browser-profile openclaw status
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

إذا ظهرت رسالة "Browser disabled"، فعّله في التكوين (انظر أدناه) وأعد تشغيل البوابة.

## ملفات التعريف: openclaw مقابل chrome

-   `openclaw`: متصفح مدار ومعزول (لا يتطلب امتدادًا).
-   `chrome`: ناقل امتداد إلى **متصفح نظامك** (يتطلب إرفاق امتداد OpenClaw بعلامة تبويب).

اضبط `browser.defaultProfile: "openclaw"` إذا كنت تريد الوضع المدار افتراضيًا.

## التكوين

توجد إعدادات المتصفح في `~/.openclaw/openclaw.json`.

```json
{
  browser: {
    enabled: true, // الافتراضي: true
    ssrfPolicy: {
      dangerouslyAllowPrivateNetwork: true, // وضع الشبكة الموثوقة الافتراضي
      // allowPrivateNetwork: true, // اسم مستعار قديم
      // hostnameAllowlist: ["*.example.com", "example.com"],
      // allowedHostnames: ["localhost"],
    },
    // cdpUrl: "http://127.0.0.1:18792", // تجاوز ملف تعريف واحد قديم
    remoteCdpTimeoutMs: 1500, // مهلة HTTP لـ CDP عن بُعد (مللي ثانية)
    remoteCdpHandshakeTimeoutMs: 3000, // مهلة مصافحة WebSocket لـ CDP عن بُعد (مللي ثانية)
    defaultProfile: "chrome",
    color: "#FF4500",
    headless: false,
    noSandbox: false,
    attachOnly: false,
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" },
    },
  },
}
```

ملاحظات:

-   تربط خدمة التحكم في المتصفح بحلقة العودة على منفذ مشتق من `gateway.port` (الافتراضي: `18791`، وهو gateway + 2). يستخدم الناقل المنفذ التالي (`18792`).
-   إذا قمت بتجاوز منفذ البوابة (`gateway.port` أو `OPENCLAW_GATEWAY_PORT`)، فإن منافذ المتصفح المشتقة تتغير للبقاء في نفس "العائلة".
-   `cdpUrl` الافتراضي هو منفذ الناقل عندما لا يتم تعيينه.
-   `remoteCdpTimeoutMs` ينطبق على فحوصات وصول CDP عن بُعد (غير حلقة العودة).
-   `remoteCdpHandshakeTimeoutMs` ينطبق على فحوصات وصول WebSocket لـ CDP عن بُعد.
-   يتم حماية التنقل في المتصفح/فتح علامة التبويب من SSRF قبل التنقل ويتم إعادة التحقق منها بأفضل جهد على عنوان URL النهائي `http(s)` بعد التنقل.
-   `browser.ssrfPolicy.dangerouslyAllowPrivateNetwork` الافتراضي هو `true` (نموذج الشبكة الموثوقة). اضبطه على `false` للتصفح العام الصارم فقط.
-   `browser.ssrfPolicy.allowPrivateNetwork` لا يزال مدعومًا كاسم مستعار قديم للتطابق.
-   `attachOnly: true` تعني "عدم تشغيل متصفح محلي أبدًا؛ قم بالإرفاق فقط إذا كان قيد التشغيل بالفعل."
-   `color` + `color` لكل ملف تعريف تلوين واجهة المستخدم للمتصفح حتى تتمكن من رؤية الملف الشخصي النشط.
-   الملف الشخصي الافتراضي هو `openclaw` (متصفح مستقل مدار من OpenClaw). استخدم `defaultProfile: "chrome"` للانضمام إلى ناقل امتداد Chrome.
-   ترتيب الكشف التلقائي: متصفح النظام الافتراضي إذا كان يعتمد على Chromium؛ وإلا Chrome → Brave → Edge → Chromium → Chrome Canary.
-   تخصص ملفات تعريف `openclaw` المحلية تلقائيًا `cdpPort`/`cdpUrl` — اضبطها فقط لـ CDP عن بُعد.

## استخدام Brave (أو متصفح آخر يعتمد على Chromium)

إذا كان **متصفح نظامك الافتراضي** يعتمد على Chromium (Chrome/Brave/Edge/إلخ)، فإن OpenClaw يستخدمه تلقائيًا. اضبط `browser.executablePath` لتجاوز الكشف التلقائي: مثال CLI:

```bash
openclaw config set browser.executablePath "/usr/bin/google-chrome"
```

```
// macOS
{
  browser: {
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser"
  }
}

// Windows
{
  browser: {
    executablePath: "C:\\Program Files\\BraveSoftware\\Brave-Browser\\Application\\brave.exe"
  }
}

// Linux
{
  browser: {
    executablePath: "/usr/bin/brave-browser"
  }
}
```

## التحكم المحلي مقابل التحكم عن بُعد

-   **التحكم المحلي (الافتراضي):** تبدأ البوابة خدمة التحكم في حلقة العودة ويمكنها تشغيل متصفح محلي.
-   **التحكم عن بُعد (مضيف العقدة):** قم بتشغيل مضيف عقدة على الجهاز الذي يحتوي على المتصفح؛ تقوم البوابة بتمرير إجراءات المتصفح إليه.
-   **CDP عن بُعد:** اضبط `browser.profiles..cdpUrl` (أو `browser.cdpUrl`) للإرفاق بمتصفح يعتمد على Chromium عن بُعد. في هذه الحالة، لن يقوم OpenClaw بتشغيل متصفح محلي.

يمكن أن تتضمن عناوين URL لـ CDP عن بُعد مصادقة:

-   رموز الاستعلام (مثل `https://provider.example?token=`)
-   مصادقة HTTP الأساسية (مثل `https://user:pass@provider.example`)

يحافظ OpenClaw على المصادقة عند استدعاء نقاط النهاية `/json/*` وعند الاتصال بـ WebSocket الخاص بـ CDP. يُفضل استخدام متغيرات البيئة أو مديري الأسرار للرموز بدلاً من إضافتها إلى ملفات التكوين.

## وكيل متصفح العقدة (الافتراضي بدون تكوين)

إذا قمت بتشغيل **مضيف عقدة** على الجهاز الذي يحتوي على متصفحك، يمكن لـ OpenClaw توجيه مكالمات أدوات المتصفح تلقائيًا إلى تلك العقدة دون أي تكوين إضافي للمتصفح. هذا هو المسار الافتراضي للبوابات البعيدة. ملاحظات:

-   يعرض مضيف العقدة خادم التحكم في المتصفح المحلي عبر **أمر وكيل**.
-   تأتي ملفات التعريف من تكوين `browser.profiles` الخاص بالعقدة (نفس الشيء محليًا).
-   عطله إذا كنت لا تريده:
    -   على العقدة: `nodeHost.browserProxy.enabled=false`
    -   على البوابة: `gateway.nodes.browser.mode="off"`

## Browserless (CDP بعيد مستضاف)

[Browserless](https://browserless.io) هي خدمة Chromium مستضافة تعرض نقاط نهاية CDP عبر HTTPS. يمكنك توجيه ملف تعريف متصفح OpenClaw إلى نقطة نهاية منطقة Browserless والمصادقة باستخدام مفتاح API الخاص بك. مثال:

```json
{
  browser: {
    enabled: true,
    defaultProfile: "browserless",
    remoteCdpTimeoutMs: 2000,
    remoteCdpHandshakeTimeoutMs: 4000,
    profiles: {
      browserless: {
        cdpUrl: "https://production-sfo.browserless.io?token=<BROWSERLESS_API_KEY>",
        color: "#00AA00",
      },
    },
  },
}
```

ملاحظات:

-   استبدل `<BROWSERLESS_API_KEY>` برمز Browserless الحقيقي الخاص بك.
-   اختر نقطة النهاية الإقليمية التي تطابق حساب Browserless الخاص بك (انظر وثائقهم).

## الأمان

أفكار رئيسية:

-   التحكم في المتصفح يكون في حلقة العودة فقط؛ يتدفق الوصول من خلال مصادقة البوابة أو اقتران العقدة.
-   إذا تم تمكين التحكم في المتصفح ولم يتم تكوين أي مصادقة، فإن OpenClaw يولد تلقائيًا `gateway.auth.token` عند بدء التشغيل ويحفظه في التكوين.
-   احتفظ بالبوابة وأي مضيفي عقد على شبكة خاصة (Tailscale)؛ تجنب التعرض العام.
-   تعامل مع عناوين URL/رموز CDP البعيدة كأسرار؛ يُفضل استخدام متغيرات البيئة أو مدير الأسرار.

نصائح CDP عن بُعد:

-   يُفضل استخدام نقاط نهاية HTTPS والرموز قصيرة العمر حيثما أمكن.
-   تجنب تضمين الرموز طويلة العمر مباشرة في ملفات التكوين.

## ملفات التعريف (متعددة المتصفحات)

يدعم OpenClaw ملفات تعريف مسماة متعددة (تكوينات التوجيه). يمكن أن تكون ملفات التعريف:

-   **مدارة من openclaw**: مثيل متصفح مخصص يعتمد على Chromium مع دليل بيانات المستخدم الخاص به + منفذ CDP
-   **بعيد**: عنوان URL صريح لـ CDP (متصفح يعتمد على Chromium يعمل في مكان آخر)
-   **ناقل الامتداد**: علامات تبويب Chrome الحالية الخاصة بك عبر الناقل المحلي + امتداد Chrome

الافتراضيات:

-   يتم إنشاء ملف تعريف `openclaw` تلقائيًا إذا كان مفقودًا.
-   ملف تعريف `chrome` مضمن لناقل امتداد Chrome (يشير إلى `http://127.0.0.1:18792` افتراضيًا).
-   تخصص منافذ CDP المحلية من **18800–18899** افتراضيًا.
-   يؤدي حذف ملف تعريف إلى نقل دليل البيانات المحلي الخاص به إلى سلة المهملات.

تقبل جميع نقاط نهاية التحكم `?profile=`؛ يستخدم CLI `--browser-profile`.

## ناقل امتداد Chrome (استخدم Chrome الحالي الخاص بك)

يمكن لـ OpenClaw أيضًا تشغيل **علامات تبويب Chrome الحالية الخاصة بك** (بدون مثيل Chrome منفصل "openclaw") عبر ناقل CDP محلي + امتداد Chrome. دليل كامل: [امتداد Chrome](./chrome-extension.md) التدفق:

-   تعمل البوابة محليًا (نفس الجهاز) أو يعمل مضيف عقدة على جهاز المتصفح.
-   يستمع **خادم ناقل** محلي عند `cdpUrl` لحلقة العودة (الافتراضي: `http://127.0.0.1:18792`).
-   تنقر على أيقونة امتداد **OpenClaw Browser Relay** على علامة تبويب للإرفاق (لا يتم الإرفاق تلقائيًا).
-   يتحكم الوكيل في تلك علامة التبويب عبر أداة `browser` العادية، عن طريق تحديد الملف الشخصي الصحيح.

إذا كانت البوابة تعمل في مكان آخر، فقم بتشغيل مضيف عقدة على جهاز المتصفح حتى تتمكن البوابة من تمرير إجراءات المتصفح.

### الجلسات المعزولة

إذا كانت جلسة الوكيل معزولة، فقد تستخدم أداة `browser` افتراضيًا `target="sandbox"` (متصفح معزول). يتطلب استيلاء ناقل امتداد Chrome التحكم في متصفح المضيف، لذا إما:

-   قم بتشغيل الجلسة بدون عزل، أو
-   اضبط `agents.defaults.sandbox.browser.allowHostControl: true` واستخدم `target="host"` عند استدعاء الأداة.

### الإعداد

1.  قم بتحميل الامتداد (dev/unpacked):

```bash
openclaw browser extension install
```

-   Chrome → `chrome://extensions` → تمكين "وضع المطور"
-   "Load unpacked" → حدد الدليل المطبوع بواسطة `openclaw browser extension path`
-   ثبت الامتداد، ثم انقر عليه على علامة التبويب التي تريد التحكم فيها (تظهر الشارة `ON`).

2.  استخدمه:

-   CLI: `openclaw browser --browser-profile chrome tabs`
-   أداة الوكيل: `browser` مع `profile="chrome"`

اختياري: إذا كنت تريد اسمًا مختلفًا أو منفذ ناقل مختلف، فأنشئ ملف تعريفك الخاص:

```bash
openclaw browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

ملاحظات:

-   يعتمد هذا الوضع على Playwright-on-CDP لمعظم العمليات (صور الشاشة/اللقطات/الإجراءات).
-   افصل بالنقر على أيقونة الامتداد مرة أخرى.

## ضمانات العزل

-   **دليل بيانات مستخدم مخصص**: لا يلمس ملف تعريف متصفحك الشخصي أبدًا.
-   **منافذ مخصصة**: يتجنب `9222` لمنع التعارضات مع سير عمل التطوير.
-   **تحكم حتمي في علامات التبويب**: استهدف علامات التبويب بواسطة `targetId`، وليس "علامة التبويب الأخيرة".

## اختيار المتصفح

عند التشغيل محليًا، يختار OpenClaw أول متاح:

1.  Chrome
2.  Brave
3.  Edge
4.  Chromium
5.  Chrome Canary

يمكنك التجاوز باستخدام `browser.executablePath`. المنصات:

-   macOS: يتحقق من `/Applications` و `~/Applications`.
-   Linux: يبحث عن `google-chrome`, `brave`, `microsoft-edge`, `chromium`, إلخ.
-   Windows: يتحقق من مواقع التثبيت الشائعة.

## واجهة برمجة التطبيقات للتحكم (اختياري)

للتكاملات المحلية فقط، تعرض البواجة واجهة برمجة تطبيقات HTTP صغيرة لحلقة العودة:

-   الحالة/البدء/الإيقاف: `GET /`, `POST /start`, `POST /stop`
-   علامات التبويب: `GET /tabs`, `POST /tabs/open`, `POST /tabs/focus`, `DELETE /tabs/:targetId`
-   لقطة/صورة شاشة: `GET /snapshot`, `POST /screenshot`
-   إجراءات: `POST /navigate`, `POST /act`
-   خطاطيف: `POST /hooks/file-chooser`, `POST /hooks/dialog`
-   التنزيلات: `POST /download`, `POST /wait/download`
-   التصحيح: `GET /console`, `POST /pdf`
-   التصحيح: `GET /errors`, `GET /requests`, `POST /trace/start`, `POST /trace/stop`, `POST /highlight`
-   الشبكة: `POST /response/body`
-   الحالة: `GET /cookies`, `POST /cookies/set`, `POST /cookies/clear`
-   الحالة: `GET /storage/:kind`, `POST /storage/:kind/set`, `POST /storage/:kind/clear`
-   الإعدادات: `POST /set/offline`, `POST /set/headers`, `POST /set/credentials`, `POST /set/geolocation`, `POST /set/media`, `POST /set/timezone`, `POST /set/locale`, `POST /set/device`

تقبل جميع نقاط النهاية `?profile=`. إذا تم تكوين مصادقة البوابة، فإن مسارات HTTP للمتصفح تتطلب مصادقة أيضًا:

-   `Authorization: Bearer `
-   `x-openclaw-password: ` أو مصادقة HTTP الأساسية باستخدام تلك كلمة المرور

### متطلب Playwright

تتطلب بعض الميزات (التنقل/الإجراء/لقطة الذكاء الاصطناعي/لقطة الدور، صور شاشة العنصر، PDF) Playwright. إذا لم يكن Playwright مثبتًا، فترجع نقاط النهاية هذه خطأ 501 واضحًا. لا تزال لقطات ARIA وصور الشاشة الأساسية تعمل لمتصفح Chrome المدار من openclaw. بالنسبة لبرنامج تشغيل ناقل امتداد Chrome، تتطلب لقطات ARIA وصور الشاشة Playwright. إذا رأيت `Playwright is not available in this gateway build`، فقم بتثبيت حزمة Playwright الكاملة (وليس `playwright-core`) وأعد تشغيل البوابة، أو أعد تثبيت OpenClaw مع دعم المتصفح.

#### تثبيت Playwright في Docker

إذا كانت بوابتك تعمل في Docker، فتجنب `npx playwright` (تعارضات تجاوز npm). استخدم CLI المضمن بدلاً من ذلك:

```bash
docker compose run --rm openclaw-cli \
  node /app/node_modules/playwright-core/cli.js install chromium
```

لحفظ تنزيلات المتصفح، اضبط `PLAYWRIGHT_BROWSERS_PATH` (على سبيل المثال، `/home/node/.cache/ms-playwright`) وتأكد من استمرار `/home/node` عبر `OPENCLAW_HOME_VOLUME` أو ربط نقطة. انظر [Docker](../install/docker.md).

## كيف يعمل (داخليًا)

تدفق عالي المستوى:

-   يقبل **خادم تحكم** صغير طلبات HTTP.
-   يتصل بمتصفحات تعتمد على Chromium (Chrome/Brave/Edge/Chromium) عبر **CDP**.
-   للإجراءات المتقدمة (نقر/كتابة/لقطة/PDF)، يستخدم **Playwright** فوق CDP.
-   عندما يكون Playwright مفقودًا، تتوفر فقط العمليات غير Playwright.

يحافظ هذا التصميم على الوكيل على واجهة مستقرة وحتمية بينما يتيح لك تبديل المتصفحات والملفات الشخصية المحلية/البعيدة.

## مرجع سريع لـ CLI

تقبل جميع الأوامر `--browser-profile ` لاستهداف ملف تعريف محدد. تقبل جميع الأوامر أيضًا `--json` للإخراج الذي يمكن قراءته آليًا (حمولات مستقرة). الأساسيات:

-   `openclaw browser status`
-   `openclaw browser start`
-   `openclaw browser stop`
-   `openclaw browser tabs`
-   `openclaw browser tab`
-   `openclaw browser tab new`
-   `openclaw browser tab select 2`
-   `openclaw browser tab close 2`
-   `openclaw browser open https://example.com`
-   `openclaw browser focus abcd1234`
-   `openclaw browser close abcd1234`

التفتيش:

-   `openclaw browser screenshot`
-   `openclaw browser screenshot --full-page`
-   `openclaw browser screenshot --ref 12`
-   `openclaw browser screenshot --ref e12`
-   `openclaw browser snapshot`
-   `openclaw browser snapshot --format aria --limit 200`
-   `openclaw browser snapshot --interactive --compact --depth 6`
-   `openclaw browser snapshot --efficient`
-   `openclaw browser snapshot --labels`
-   `openclaw browser snapshot --selector "#main" --interactive`
-   `openclaw browser snapshot --frame "iframe#main" --interactive`
-   `openclaw browser console --level error`
-   `openclaw browser errors --clear`
-   `openclaw browser requests --filter api --clear`
-   `openclaw browser pdf`
-   `openclaw browser responsebody "**/api" --max-chars 5000`

الإجراءات:

-   `openclaw browser navigate https://example.com`
-   `openclaw browser resize 1280 720`
-   `openclaw browser click 12 --double`
-   `openclaw browser click e12 --double`
-   `openclaw browser type 23 "hello" --submit`
-   `openclaw browser press Enter`
-   `openclaw browser hover 44`
-   `openclaw browser scrollintoview e12`
-   `openclaw browser drag 10 11`
-   `openclaw browser select 9 OptionA OptionB`
-   `openclaw browser download e12 report.pdf`
-   `openclaw browser waitfordownload report.pdf`
-   `openclaw browser upload /tmp/openclaw/uploads/file.pdf`
-   `openclaw browser fill --fields '[{"ref":"1","type":"text","value":"Ada"}]'`
-   `openclaw browser dialog --accept`
-   `openclaw browser wait --text "Done"`
-   `openclaw browser wait "#main" --url "**/dash" --load networkidle --fn "window.ready===true"`
-   `openclaw browser evaluate --fn '(el) => el.textContent' --ref 7`
-   `openclaw browser highlight e12`
-   `openclaw browser trace start`
-   `openclaw browser trace stop`

الحالة:

-   `openclaw browser cookies`
-   `openclaw browser cookies set session abc123 --url "https://example.com"`
-   `openclaw browser cookies clear`
-   `openclaw browser storage local get`
-   `openclaw browser storage local set theme dark`
-   `openclaw browser storage session clear`
-   `openclaw browser set offline on`
-   `openclaw browser set headers --headers-json '{"X-Debug":"1"}'`
-   `openclaw browser set credentials user pass`
-   `openclaw browser set credentials --clear`
-   `openclaw browser set geo 37.7749 -122.4194 --origin "https://example.com"`
-   `openclaw browser set geo --clear`
-   `openclaw browser set media dark`
-   `openclaw browser set timezone America/New_York`
-   `openclaw browser set locale en-US`
-   `openclaw browser set device "iPhone 14"`

ملاحظات:

-   `upload` و `dialog` هما مكالمات **تسليح**؛ قم بتشغيلهما قبل النقرة/الضغطة التي تؤدي إلى تشغيل منتقي الملفات/الحوار.
-   مسارات إخراج التنزيل والتتبع مقيدة بجذور OpenClaw المؤقتة:
    -   التتبع: `/tmp/openclaw` (الاحتياطي: `${os.tmpdir()}/openclaw`)
    -   التنزيلات: `/tmp/openclaw/downloads` (الاحتياطي: `${os.tmpdir()}/openclaw/downloads`)
-   مسارات التحميل مقيدة بجذر تحميلات OpenClaw المؤقت:
    -   التحميلات: `/tmp/openclaw/uploads` (الاحتياطي: `${os.tmpdir()}/openclaw/uploads`)
-   يمكن لـ `upload` أيضًا تعيين مدخلات الملف مباشرة عبر `--input-ref` أو `--element`.
-   `snapshot`:
    -   `--format ai` (الافتراضي عند تثبيت Playwright): يُرجع لقطة ذكاء اصطناعي مع مراجع رقمية (`aria-ref=""`).
    -   `--format aria`: يُرجع شجرة إمكانية الوصول (بدون مراجع؛ للفحص فقط).
    -   `--efficient` (أو `--mode efficient`): إعداد مسبق مضغوط للقطة الدور (interactive + compact + depth + maxChars أقل).
    -   الافتراضي للتكوين (أداة/CLI فقط): اضبط `browser.snapshotDefaults.mode: "efficient"` لاستخدام لقطات فعالة عندما لا يمرر المتصل وضعًا (انظر [تكوين البوابة](../gateway/configuration.md#browser-openclaw-managed-browser)).
    -   تجبر خيارات لقطة الدور (`--interactive`, `--compact`, `--depth`, `--selector`) على لقطة قائمة على الدور مع مراجع مثل `ref=e12`.
    -   `--frame ""` يحدد نطاق لقطات الدور إلى إطار iframe (يقترن بمراجع دور مثل `e12`).
    -   `--interactive` يُخرج قائمة مسطحة وسهلة الاختيار للعناصر التفاعلية (الأفضل لدفع الإجراءات).
    -   `--labels` يضيف صورة شاشة لمنطقة العرض فقط مع ملصقات مراجع متراكبة (يطبع `MEDIA:`).
-   تتطلب `click`/`type`/إلخ `ref` من `snapshot` (إما رقمي `12` أو مرجع دور `e12`). لا يتم دعم محددات CSS عمدًا للإجراءات.

## اللقطات والمراجع

يدعم OpenClaw نمطين من "اللقطات":

-   **لقطة الذكاء الاصطناعي (مراجع رقمية)**: `openclaw browser snapshot` (الافتراضي؛ `--format ai`)
    -   الإخراج: لقطة نصية تتضمن مراجع رقمية.
    -   الإجراءات: `openclaw browser click 12`, `openclaw browser type 23 "hello"`.
    -   داخليًا، يتم حل المرجع عبر `aria-ref` الخاص بـ Playwright.
-   **لقطة الدور (مراجع دور مثل `e12`)**: `openclaw browser snapshot --interactive` (أو `--compact`, `--depth`, `--selector`, `--frame`)
    -   الإخراج: قائمة/شجرة قائمة على الدور مع `[ref=e12]` (واختياري `[nth=1]`).
    -   الإجراءات: `openclaw browser click e12`, `openclaw browser highlight e12`.
    -   داخليًا، يتم حل المرجع عبر `getByRole(...)` (بالإضافة إلى `nth()` للعناصر المكررة).
    -   أضف `--labels` لتضمين صورة شاشة لمنطقة العرض مع ملصقات `e12` متراكبة.

سلوك المرجع:

-   المراجع **ليست مستقرة عبر التنقلات**؛ إذا فشل شيء ما، أعد تشغيل `snapshot` واستخدم مرجعًا جديدًا.
-   إذا تم أخذ لقطة الدور باستخدام `--frame`، فإن مراجع الدور تكون محددة النطاق إلى إطار iframe ذلك حتى لقطة الدور التالية.

## تعزيزات الانتظار

يمكنك الانتظار لأكثر من مجرد وقت/نص:

-   انتظر عنوان URL (تدعم globs بواسطة Playwright):
    -   `openclaw browser wait --url "**/dash"`
-   انتظر حالة التحميل:
    -   `openclaw browser wait --load networkidle`
-   انتظر مسند JavaScript:
    -   `openclaw browser wait --fn "window.ready===true"`
-   انتظر حتى يصبح محدد مرئيًا:
    -   `openclaw browser wait "#main"`

يمكن دمج هذه:

```bash
openclaw browser wait "#main" \
  --url "**/dash" \
  --load networkidle \
  --fn "window.ready===true" \
  --timeout-ms 15000
```

## سير عمل التصحيح

عندما يفشل إجراء (مثل "غير مرئي"، "انتهاك الوضع الصارم"، "مغطى"):

1.  `openclaw browser snapshot --interactive`
2.  استخدم `click ` / `type ` (يُفضل مراجع الدور في الوضع التفاعلي)
3.  إذا استمر الفشل: `openclaw browser highlight ` لرؤية ما يستهدفه Playwright
4.  إذا تصرفت الصفحة بشكل غريب:
    -   `openclaw browser errors --clear`
    -   `openclaw browser requests --filter api --clear`
5.  للتصحيح العميق: سجل تتبع:
    -   `openclaw browser trace start`
    -   استنساخ المشكلة
    -   `openclaw browser trace stop` (يطبع `TRACE:`)

## إخراج JSON

`--json` مخصص للبرمجة النصية وأدوات منظمة. أمثلة:

```bash
openclaw browser status --json
openclaw browser snapshot --interactive --json
openclaw browser requests --filter api --json
openclaw browser cookies --json
```

تتضمن لقطات الدور في JSON `refs` بالإضافة إلى كتلة `stats` صغيرة (أسطر/أحرف/مراجع/تفاعلية) حتى تتمكن الأدوات من التفكير في حجم وكثافة الحمولة.

## الحالة ومفاتيح البيئة

هذه مفيدة لسير عمل "اجعل الموقع يتصرف مثل X":

-   ملفات تعريف الارتباط: `cookies`, `cookies set`, `cookies clear`
-   التخزين: `storage local|session get|set|clear`
-   غير متصل: `set offline on|off`
-   الرؤوس: `set headers --headers-json '{"X-Debug":"1"}'` (يبقى `set headers --json '{"X-Debug":"1"}'` القديم مدعومًا)
-   مصادقة HTTP الأساسية: `set credentials user pass` (أو `--clear`)
-   الموقع الجغرافي: `set geo   --origin "https://example.com"` (أو `--clear`)
-   الوسائط: `set media dark|light|no-preference|none`
-   المنطقة الزمنية / اللغة: `set timezone ...`, `set locale ...`
-   الجهاز / منطقة العرض:
    -   `set device "iPhone 14"` (إعدادات جهاز Playwright المسبقة)
    -   `set viewport 1280 720`

## الأمان والخصوصية

-   قد يحتوي ملف تعريف متصفح openclaw على جلسات تسجيل دخول؛ تعامل معه على أنه حساس.
-   `browser act kind=evaluate` / `openclaw browser evaluate` و `wait --fn` تنفذ JavaScript عشوائيًا في سياق الصفحة. يمكن لتوجيه الحقن توجيه هذا. عطله باستخدام `browser.evaluateEnabled=false` إذا كنت لا تحتاجه.
-   لتسجيلات الدخول وملاحظات مكافحة الروبوتات (X/Twitter، إلخ)، انظر [تسجيل دخول المتصفح + نشر X/Twitter](./browser-login.md).
-   احتفظ بالبوابة/مضيف العقدة خاصًا (حلقة العودة أو شبكة ذيل فقط).
-   نقاط نهاية CDP البعيدة قوية؛ أنفقها واحمها.

مثال على الوضع الصارم (حظر الوجهات الخاصة/الداخلية افتراضيًا):

```json
{
  browser: {
    ssrfPolicy: {
      dangerouslyAllowPrivateNetwork: false,
      hostnameAllowlist: ["*.example.com", "example.com"],
      allowedHostnames: ["localhost"], // سماح دقيق اختياري
    },
  },
}
```

## استكشاف الأخطاء وإصلاحها

لمشكلات Linux المحددة (خاصة snap Chromium)، انظر [استكشاف أخطاء المتصفح وإصلاحها](./browser-linux-troubleshooting.md).

## أد