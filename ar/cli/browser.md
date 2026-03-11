

  أوامر سطر الأوامر

  
# browser

إدارة خادم تحكم متصفح OpenClaw وتنفيذ إجراءات المتصفح (علامات التبويب، اللقطات، لقطات الشاشة، التنقل، النقرات، الكتابة). متعلق:

-   أداة المتصفح + API: [أداة المتصفح](../tools/browser.md)
-   ناقل امتداد Chrome: [امتداد Chrome](../tools/chrome-extension.md)

## الأعلام الشائعة

-   `--url `: عنوان WebSocket للبوابة (الافتراضي من التكوين).
-   `--token `: رمز البوابة (إذا كان مطلوبًا).
-   `--timeout `: مهلة الطلب (ميلي ثانية).
-   `--browser-profile `: اختر ملف تعريف متصفح (الافتراضي من التكوين).
-   `--json`: إخراج قابل للقراءة الآلية (حيث يكون مدعومًا).

## البدء السريع (محلي)

```bash
openclaw browser --browser-profile chrome tabs
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

## ملفات التعريف

ملفات التعريف هي تكوينات توجيه متصفح مسماة. عمليًا:

-   `openclaw`: يقوم بتشغيل/ربط مثيل Chrome مخصص تديره OpenClaw (دليل بيانات مستخدم معزول).
-   `chrome`: يتحكم في علامة/علامات تبويب Chrome الحالية الخاصة بك عبر ناقل امتداد Chrome.

```bash
openclaw browser profiles
openclaw browser create-profile --name work --color "#FF5A36"
openclaw browser delete-profile --name work
```

استخدم ملف تعريف محددًا:

```bash
openclaw browser --browser-profile work tabs
```

## علامات التبويب

```bash
openclaw browser tabs
openclaw browser open https://docs.openclaw.ai
openclaw browser focus <targetId>
openclaw browser close <targetId>
```

## لقطة / لقطة شاشة / إجراءات

لقطة:

```bash
openclaw browser snapshot
```

لقطة شاشة:

```bash
openclaw browser screenshot
```

التنقل/النقر/الكتابة (أتمتة واجهة المستخدم المعتمدة على المرجع):

```bash
openclaw browser navigate https://example.com
openclaw browser click <ref>
openclaw browser type <ref> "hello"
```

## ناقل امتداد Chrome (الربط عبر زر شريط الأدوات)

يتيح هذا الوضع للوكيل التحكم في علامة تبويب Chrome موجودة تقوم بربطها يدويًا (لا يتم الربط تلقائيًا). قم بتثبيت الامتداد غير المعبأ في مسار ثابت:

```bash
openclaw browser extension install
openclaw browser extension path
```

ثم Chrome → `chrome://extensions` → تمكين "وضع المطور" → "تحميل غير معبأ" → حدد المجلد المطبوع. الدليل الكامل: [امتداد Chrome](../tools/chrome-extension.md)

## التحكم عن بعد في المتصفح (وكيل مضيف العقدة)

إذا كان البوابة تعمل على جهاز مختلف عن المتصفح، قم بتشغيل **مضيف عقدة** على الجهاز الذي يحتوي على Chrome/Brave/Edge/Chromium. سيقوم البوابة بتمرير إجراءات المتصفح إلى تلك العقدة (لا يلزم وجود خادم تحكم متصفح منفصل). استخدم `gateway.nodes.browser.mode` للتحكم في التوجيه التلقائي و `gateway.nodes.browser.node` لتثبيت عقدة محددة إذا كانت هناك عقد متعددة متصلة. الأمان + الإعداد عن بعد: [أداة المتصفح](../tools/browser.md)، [الوصول عن بعد](../gateway/remote.md)، [Tailscale](../gateway/tailscale.md)، [الأمان](../gateway/security.md)

[الموافقات](./approvals.md)[القنوات](./channels.md)

---