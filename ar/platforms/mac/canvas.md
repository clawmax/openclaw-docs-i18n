

  التطبيق المصاحب لنظام macOS

  
# Canvas

يضم تطبيق macOS **لوحة Canvas** يتم التحكم فيها بواسطة وكيل باستخدام `WKWebView`. إنها مساحة عمل مرئية خفيفة الوزن لـ HTML/CSS/JS وA2UI وأسطح واجهة المستخدم التفاعلية الصغيرة.

## مكان تخزين Canvas

يتم تخزين حالة Canvas ضمن Application Support:

-   `~/Library/Application Support/OpenClaw/canvas//...`

تقدم لوحة Canvas هذه الملفات عبر **مخطط URL مخصص**:

-   `openclaw-canvas:///`

أمثلة:

-   `openclaw-canvas://main/` → `/main/index.html`
-   `openclaw-canvas://main/assets/app.css` → `/main/assets/app.css`
-   `openclaw-canvas://main/widgets/todo/` → `/main/widgets/todo/index.html`

إذا لم يكن هناك ملف `index.html` في الجذر، يعرض التطبيق **صفحة سقالة مدمجة**.

## سلوك اللوحة

-   لوحة بلا حدود، قابلة لتغيير الحجم، مثبتة بالقرب من شريط القوائم (أو مؤشر الفأرة).
-   تتذكر الحجم/الموضع لكل جلسة.
-   تعيد التحميل تلقائيًا عند تغيير ملفات Canvas المحلية.
-   تظهر لوحة Canvas واحدة فقط في كل مرة (يتم تبديل الجلسة حسب الحاجة).

يمكن تعطيل Canvas من الإعدادات → **السماح بـ Canvas**. عند التعطيل، تعيد أوامر عقدة canvas `CANVAS_DISABLED`.

## سطح واجهة برمجة تطبيقات الوكيل

يتم عرض Canvas عبر **مقبس الويب Gateway**، بحيث يمكن للوكيل:

-   إظهار/إخفاء اللوحة
-   التنقل إلى مسار أو عنوان URL
-   تنفيذ JavaScript
-   التقاط صورة لقطة شاشة

أمثلة CLI:

```bash
openclaw nodes canvas present --node <id>
openclaw nodes canvas navigate --node <id> --url "/"
openclaw nodes canvas eval --node <id> --js "document.title"
openclaw nodes canvas snapshot --node <id>
```

ملاحظات:

-   `canvas.navigate` تقبل **مسارات Canvas المحلية**، وعناوين URL `http(s)`، وعناوين URL `file://`.
-   إذا قمت بتمرير `"/"`، فإن Canvas تعرض السقالة المحلية أو `index.html`.

## A2UI في Canvas

يتم استضافة A2UI بواسطة مضيف Canvas الخاص بـ Gateway ويتم عرضه داخل لوحة Canvas. عندما يعلن Gateway عن مضيف Canvas، يقوم تطبيق macOS تلقائيًا بالتنقل إلى صفحة مضيف A2UI عند الفتح الأول. عنوان URL الافتراضي لمضيف A2UI:

```
http://<gateway-host>:18789/__openclaw__/a2ui/
```

### أوامر A2UI (الإصدار 0.8)

يقبل Canvas حاليًا رسائل **A2UI الإصدار 0.8** من الخادم إلى العميل:

-   `beginRendering`
-   `surfaceUpdate`
-   `dataModelUpdate`
-   `deleteSurface`

`createSurface` (الإصدار 0.9) غير مدعوم. مثال CLI:

```bash
cat > /tmp/a2ui-v0.8.jsonl <<'EOFA2'
{"surfaceUpdate":{"surfaceId":"main","components":[{"id":"root","component":{"Column":{"children":{"explicitList":["title","content"]}}}},{"id":"title","component":{"Text":{"text":{"literalString":"Canvas (A2UI v0.8)"},"usageHint":"h1"}}},{"id":"content","component":{"Text":{"text":{"literalString":"If you can read this, A2UI push works."},"usageHint":"body"}}}]}}
{"beginRendering":{"surfaceId":"main","root":"root"}}
EOFA2

openclaw nodes canvas a2ui push --jsonl /tmp/a2ui-v0.8.jsonl --node <id>
```

اختبار سريع:

```bash
openclaw nodes canvas a2ui push --node <id> --text "Hello from A2UI"
```

## تشغيل عمليات الوكيل من Canvas

يمكن لـ Canvas تشغيل عمليات وكيل جديدة عبر الروابط العميقة:

-   `openclaw://agent?...`

مثال (في JS):

```bash
window.location.href = "openclaw://agent?message=Review%20this%20design";
```

سيطلب التطبيق تأكيدًا ما لم يتم تقديم مفتاح صالح.

## ملاحظات أمنية

-   يمنع مخطط Canvas اجتياز الدلائل؛ يجب أن توجد الملفات تحت جذر الجلسة.
-   يستخدم محتوى Canvas المحلي مخططًا مخصصًا (لا يلزم وجود خادم loopback).
-   يُسمح بعناوين URL الخارجية `http(s)` فقط عند التنقل إليها صراحةً.

[WebChat](./webchat.md)[دورة حياة Gateway](./child-process.md)

---