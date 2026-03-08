

  الشبكات والاكتشاف

  
# اكتشاف Bonjour

يستخدم OpenClaw بروتوكول Bonjour (mDNS / DNS‑SD) كـ **وسيلة راحة تعمل على الشبكة المحلية فقط** لاكتشاف بوابة نشطة (نقطة نهاية WebSocket). يعمل هذا النظام بأقصى جهد ممكن ولا **يُعتبر** بديلاً عن الاتصال عبر SSH أو شبكة Tailnet.

## Bonjour للنطاق الواسع (Unicast DNS‑SD) عبر Tailscale

إذا كان العقدة والبوابة على شبكتين مختلفتين، فإن البث المتعدد (multicast) mDNS لن يعبر الحدود بينهما. يمكنك الحفاظ على نفس تجربة المستخدم للاكتشاف بالتبديل إلى **DNS‑SD أحادي البث** ("Bonjour للنطاق الواسع") عبر Tailscale. الخطوات العامة:

1.  تشغيل خادم DNS على مضيف البوابة (يمكن الوصول إليه عبر شبكة Tailnet).
2.  نشر سجلات DNS‑SD لـ `_openclaw-gw._tcp` تحت نطاق مخصص (مثال: `openclaw.internal.`).
3.  تكوين **DNS المقسم (split DNS)** في Tailscale بحيث يتم حل النطاق المختار عبر خادم DNS ذلك للعملاء (بما في ذلك iOS).

يدعم OpenClaw أي نطاق اكتشاف؛ `openclaw.internal.` مجرد مثال. تتصفح العقد على iOS/Android كلاً من `local.` والنطاق المخصص للنطاق الواسع الذي قمت بتكوينه.

### تكوين البوابة (موصى به)

```json
{
  gateway: { bind: "tailnet" }, // tailnet-only (recommended)
  discovery: { wideArea: { enabled: true } }, // enables wide-area DNS-SD publishing
}
```

### إعداد خادم DNS لمرة واحدة (على مضيف البوابة)

```bash
openclaw dns setup --apply
```

يقوم هذا الأمر بتثبيت CoreDNS وتكوينه لـ:

-   الاستماع على المنفذ 53 فقط على واجهات Tailscale الخاصة بالبوابة
-   خدمة النطاق المختار (مثال: `openclaw.internal.`) من `~/.openclaw/dns/.db`

التحقق من جهاز متصل بشبكة tailnet:

```bash
dns-sd -B _openclaw-gw._tcp openclaw.internal.
dig @<TAILNET_IPV4> -p 53 _openclaw-gw._tcp.openclaw.internal PTR +short
```

### إعدادات DNS في Tailscale

في وحدة تحكم إدارة Tailscale:

-   أضف خادم أسماء يشير إلى عنوان IP الخاص بشبكة tailnet للبوابة (UDP/TCP 53).
-   أضف DNS مقسم (split DNS) بحيث يستخدم نطاق الاكتشاف الخاص بك خادم الأسماء هذا.

بمجرد قبول العملاء لـ DNS الخاص بشبكة tailnet، يمكن للعقد على iOS تصفح `_openclaw-gw._tcp` في نطاق الاكتشاف الخاص بك دون الحاجة إلى البث المتعدد.

### أمان مستمع البوابة (موصى به)

منفذ Gateway WS (الافتراضي `18789`) يرتبط بحلقة العودة (loopback) افتراضيًا. للوصول عبر LAN/tailnet، قم بربطه بشكل صريح واحتفظ بتفعيل المصادقة. للإعدادات التي تعمل على tailnet فقط:

-   عيّن `gateway.bind: "tailnet"` في `~/.openclaw/openclaw.json`.
-   أعد تشغيل البوابة (أو أعد تشغيل تطبيق شريط القائمة على macOS).

## ما الذي يُعلن عنه

فقط البوابة تُعلن عن `_openclaw-gw._tcp`.

## أنواع الخدمات

-   `_openclaw-gw._tcp` — منارة نقل البوابة (تُستخدم من قبل العقد على macOS/iOS/Android).

## مفاتيح TXT (تلميحات غير سرية)

تُعلن البوابة عن تلميحات صغيرة غير سرية لتسهيل سير عمل واجهة المستخدم:

-   `role=gateway`
-   `displayName=`
-   `lanHost=.local`
-   `gatewayPort=` (Gateway WS + HTTP)
-   `gatewayTls=1` (فقط عندما يكون TLS مفعلاً)
-   `gatewayTlsSha256=` (فقط عندما يكون TLS مفعلاً وتتوفر بصمة)
-   `canvasPort=` (فقط عندما يكون مضيف اللوحة (canvas) مفعلاً؛ حاليًا نفس `gatewayPort`)
-   `sshPort=` (الافتراضي 22 عندما لا يتم تجاوزه)
-   `transport=gateway`
-   `cliPath=` (اختياري؛ المسار المطلق لنقطة الدخول القابلة للتشغيل `openclaw`)
-   `tailnetDns=` (تلميح اختياري عندما تكون شبكة Tailnet متاحة)

ملاحظات الأمان:

-   سجلات Bonjour/mDNS TXT **غير موثقة**. يجب ألا تعاملها العملاء كمعلومات توجيه موثوقة.
-   يجب على العملاء التوجيه باستخدام نقطة نهاية الخدمة التي تم حلها (SRV + A/AAAA). عالج `lanHost`، `tailnetDns`، `gatewayPort`، و `gatewayTlsSha256` كتلميحات فقط.
-   يجب ألا تسمح عملية تثبيت TLS أبدًا لـ `gatewayTlsSha256` المُعلن عنه بتجاوز بصمة مخزنة مسبقًا.
-   يجب أن تعامل العقد على iOS/Android الاتصالات المباشرة القائمة على الاكتشاف على أنها **تعمل بـ TLS فقط** وتتطلب موافقة مستخدم صريحة قبل الوثوق ببصمة لأول مرة.

## التصحيح على macOS

أدوات مفيدة مدمجة:

-   تصفح الحالات:
    
    نسخ
    
    ```bash
    dns-sd -B _openclaw-gw._tcp local.
    ```
    
-   حل حالة واحدة (استبدل ``):
    
    نسخ
    
    ```bash
    dns-sd -L "<instance>" _openclaw-gw._tcp local.
    ```
    

إذا كان التصفح يعمل ولكن الحل يفشل، فعادة ما تواجه مشكلة في سياسة الشبكة المحلية أو محلل mDNS.

## التصحيح في سجلات البوابة

تكتب البوابة ملف سجل متداول (يتم طباعته عند بدء التشغيل كـ `gateway log file: ...`). ابحث عن أسطر `bonjour:`، خاصة:

-   `bonjour: advertise failed ...`
-   `bonjour: ... name conflict resolved` / `hostname conflict resolved`
-   `bonjour: watchdog detected non-announced service ...`

## التصحيح على عقدة iOS

تستخدم عقدة iOS `NWBrowser` لاكتشاف `_openclaw-gw._tcp`. لتسجيل السجلات:

-   الإعدادات → البوابة → متقدم → **سجلات تصحيح الاكتشاف**
-   الإعدادات → البوابة → متقدم → **سجلات الاكتشاف** → أعد الخطأ → **نسخ**

يتضمن السجل انتقالات حالة المتصفح وتغييرات مجموعة النتائج.

## حالات الفشل الشائعة

-   **Bonjour لا يعبر الشبكات**: استخدم Tailnet أو SSH.
-   **البث المتعدد محظور**: بعض شبكات Wi‑Fi تعطل mDNS.
-   **السكون / تغيير الواجهة**: قد يتخلى macOS مؤقتًا عن نتائج mDNS؛ أعد المحاولة.
-   **التصفح يعمل ولكن الحل يفشل**: حافظ على أسماء الأجهزة بسيطة (تجنب الرموز التعبيرية أو علامات الترقيم)، ثم أعد تشغيل البوابة. اسم حالة الخدمة مشتق من اسم المضيف، لذا يمكن للأسماء المعقدة للغاية أن تربك بعض المحللين.

## أسماء الحالات المهربة (\\032)

غالبًا ما تهرب Bonjour/DNS‑SD البايتات في أسماء حالات الخدمة كتسلسلات عشرية `\DDD` (مثال: المسافات تصبح `\032`).

-   هذا طبيعي على مستوى البروتوكول.
-   يجب على واجهات المستخدم فك التشفير للعرض (يستخدم iOS `BonjourEscapes.decode`).

## التعطيل / التكوين

-   `OPENCLAW_DISABLE_BONJOUR=1` يعطل الإعلان (قديم: `OPENCLAW_DISABLE_BONJOUR`).
-   `gateway.bind` في `~/.openclaw/openclaw.json` يتحكم في وضع ربط البوابة.
-   `OPENCLAW_SSH_PORT` يتجاوز منفذ SSH المُعلن عنه في TXT (قديم: `OPENCLAW_SSH_PORT`).
-   `OPENCLAW_TAILNET_DNS` ينشر تلميح MagicDNS في TXT (قديم: `OPENCLAW_TAILNET_DNS`).
-   `OPENCLAW_CLI_PATH` يتجاوز مسار CLI المُعلن عنه (قديم: `OPENCLAW_CLI_PATH`).

## وثائق ذات صلة

-   سياسة الاكتشاف واختيار النقل: [الاكتشاف](./discovery.md)
-   إقران العقد + الموافقات: [إقران البوابة](./pairing.md)

[الاكتشاف والنقل](./discovery.md)[الوصول عن بُعد](./remote.md)