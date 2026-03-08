

  الوسائط والأجهزة

  
# العُقَد

**العُقدة** هي جهاز مُرافق (macOS/iOS/Android/بدون واجهة) يتصل بـ **WebSocket** الخاص بالبوابة (نفس المنفذ الخاص بالمشغِّلين) باستخدام `role: "node"` ويعرض سطحًا للأوامر (مثل `canvas.*`, `camera.*`, `device.*`, `notifications.*`, `system.*`) عبر `node.invoke`. تفاصيل البروتوكول: [بروتوكول البوابة](./gateway/protocol.md). النقل القديم: [بروتوكول الجسر](./gateway/bridge-protocol.md) (TCP JSONL؛ مُهمَل/مُزال للعُقَد الحالية). يمكن لـ macOS أيضًا التشغيل في **وضع العُقدة**: يتصل تطبيق شريط القائمة بخادم WS الخاص بالبوابة ويعرض أوامر canvas/الكاميرا المحلية الخاصة به كعُقدة (وبالتالي تعمل `openclaw nodes …` ضد هذا الجهاز Mac). ملاحظات:

-   العُقَد هي **أجهزة طرفية**، وليست بوابات. إنها لا تشغِّل خدمة البوابة.
-   تصل رسائل Telegram/WhatsApp/إلخ إلى **البوابة**، وليس إلى العُقَد.
-   دليل استكشاف الأخطاء وإصلاحها: [/nodes/troubleshooting](./nodes/troubleshooting.md)

## الإقران + الحالة

**تستخدم عُقَد WS إقران الأجهزة.** تقدم العُقَد هوية جهاز أثناء `connect`؛ تنشئ البوابة طلب إقران جهاز لـ `role: node`. قم بالموافقة عبر واجهة سطر أوامر الأجهزة (أو واجهة المستخدم). واجهة سطر الأوامر السريعة:

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
```

ملاحظات:

-   `nodes status` يضع علامة على العُقدة على أنها **مُقرَنة** عندما يتضمن دور إقران الجهاز الخاص بها `node`.
-   `node.pair.*` (CLI: `openclaw nodes pending/approve/reject`) هو مخزن إقران عُقَد منفصل تملكه البوابة؛ إنه **لا** يتحكم في مصافحة `connect` الخاصة بـ WS.

## مضيف العُقدة البعيدة (system.run)

استخدم **مضيف عُقدة** عندما يعمل البوابة على جهاز واحد وتريد تنفيذ الأوامر على جهاز آخر. لا يزال النموذج يتحدث إلى **البوابة**؛ تقوم البوابة بتوجيه مكالمات `exec` إلى **مضيف العُقدة** عند تحديد `host=node`.

### ما الذي يعمل أين

-   **مضيف البوابة**: يستقبل الرسائل، يشغِّل النموذج، يُوجِّه مكالمات الأدوات.
-   **مضيف العُقدة**: ينفذ `system.run`/`system.which` على جهاز العُقدة.
-   **الموافقات**: يتم فرضها على مضيف العُقدة عبر `~/.openclaw/exec-approvals.json`.

### بدء تشغيل مضيف عُقدة (في المقدمة)

على جهاز العُقدة:

```bash
openclaw node run --host <gateway-host> --port 18789 --display-name "Build Node"
```

### بوابة بعيدة عبر نفق SSH (ربط loopback)

إذا كانت البوابة تربط loopback (`gateway.bind=loopback`، الافتراضي في الوضع المحلي)، لا يمكن لمضيفي العُقدة البعيدة الاتصال مباشرة. أنشئ نفق SSH وأشر مضيف العُقدة إلى الطرف المحلي للنفق. مثال (مضيف العُقدة -> مضيف البوابة):

```bash
# Terminal A (استمر في التشغيل): توجيه المنفذ المحلي 18790 -> بوابة 127.0.0.1:18789
ssh -N -L 18790:127.0.0.1:18789 user@gateway-host

# Terminal B: تصدير رمز البوابة والاتصال عبر النفق
export OPENCLAW_GATEWAY_TOKEN="<gateway-token>"
openclaw node run --host 127.0.0.1 --port 18790 --display-name "Build Node"
```

ملاحظات:

-   الرمز هو `gateway.auth.token` من تكوين البوابة (`~/.openclaw/openclaw.json` على مضيف البوابة).
-   يقرأ `openclaw node run` `OPENCLAW_GATEWAY_TOKEN` للمصادقة.

### بدء تشغيل مضيف عُقدة (خدمة)

```bash
openclaw node install --host <gateway-host> --port 18789 --display-name "Build Node"
openclaw node restart
```

### الإقران + التسمية

على مضيف البوابة:

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw nodes status
```

خيارات التسمية:

-   `--display-name` على `openclaw node run` / `openclaw node install` (يستمر في `~/.openclaw/node.json` على العُقدة).
-   `openclaw nodes rename --node <id|name|ip> --name "Build Node"` (تجاوز من البوابة).

### قائمة السماح للأوامر

موافقات التنفيذ هي **لكل مضيف عُقدة**. أضف مدخلات قائمة السماح من البوابة:

```bash
openclaw approvals allowlist add --node <id|name|ip> "/usr/bin/uname"
openclaw approvals allowlist add --node <id|name|ip> "/usr/bin/sw_vers"
```

تعيش الموافقات على مضيف العُقدة في `~/.openclaw/exec-approvals.json`.

### توجيه exec إلى العُقدة

تكوين الإعدادات الافتراضية (تكوين البوابة):

```bash
openclaw config set tools.exec.host node
openclaw config set tools.exec.security allowlist
openclaw config set tools.exec.node "<id-or-name>"
```

أو لكل جلسة:

```bash
/exec host=node security=allowlist node=<id-or-name>
```

بمجرد التعيين، أي استدعاء `exec` مع `host=node` يعمل على مضيف العُقدة (خاضع لقائمة السماح/الموافقات الخاصة بالعُقدة). متعلق:

-   [واجهة سطر أوامر مضيف العُقدة](./cli/node.md)
-   [أداة Exec](./tools/exec.md)
-   [موافقات Exec](./tools/exec-approvals.md)

## استدعاء الأوامر

مستوى منخفض (RPC خام):

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command canvas.eval --params '{"javaScript":"location.href"}'
```

توجد مساعدات ذات مستوى أعلى لسير عمل "إرفاق ملف MEDIA للوكيل" الشائعة.

## لقطات الشاشة (لقطات canvas)

إذا كانت العُقدة تعرض Canvas (WebView)، فإن `canvas.snapshot` تُرجع `{ format, base64 }`. مساعد CLI (يكتب إلى ملف مؤقت ويطبع `MEDIA:`):

```bash
openclaw nodes canvas snapshot --node <idOrNameOrIp> --format png
openclaw nodes canvas snapshot --node <idOrNameOrIp> --format jpg --max-width 1200 --quality 0.9
```

### عناصر تحكم Canvas

```bash
openclaw nodes canvas present --node <idOrNameOrIp> --target https://example.com
openclaw nodes canvas hide --node <idOrNameOrIp>
openclaw nodes canvas navigate https://example.com --node <idOrNameOrIp>
openclaw nodes canvas eval --node <idOrNameOrIp> --js "document.title"
```

ملاحظات:

-   `canvas present` تقبل عناوين URL أو مسارات الملفات المحلية (`--target`)، بالإضافة إلى `--x/--y/--width/--height` اختيارية لوضعها.
-   `canvas eval` تقبل JS مضمن (`--js`) أو وسيطة موضعية.

### A2UI (Canvas)

```bash
openclaw nodes canvas a2ui push --node <idOrNameOrIp> --text "Hello"
openclaw nodes canvas a2ui push --node <idOrNameOrIp> --jsonl ./payload.jsonl
openclaw nodes canvas a2ui reset --node <idOrNameOrIp>
```

ملاحظات:

-   فقط A2UI v0.8 JSONL مدعوم (يتم رفض v0.9/createSurface).

## الصور + مقاطع الفيديو (كاميرا العُقدة)

الصور (`jpg`):

```bash
openclaw nodes camera list --node <idOrNameOrIp>
openclaw nodes camera snap --node <idOrNameOrIp>            # الافتراضي: كلا الاتجاهين (سطرين MEDIA)
openclaw nodes camera snap --node <idOrNameOrIp> --facing front
```

مقاطع فيديو (`mp4`):

```bash
openclaw nodes camera clip --node <idOrNameOrIp> --duration 10s
openclaw nodes camera clip --node <idOrNameOrIp> --duration 3000 --no-audio
```

ملاحظات:

-   يجب أن تكون العُقدة **في المقدمة** لـ `canvas.*` و `camera.*` (تُرجع المكالمات في الخلفية `NODE_BACKGROUND_UNAVAILABLE`).
-   يتم تحديد مدة المقطع (حاليًا `<= 60s`) لتجنب حمولات base64 كبيرة الحجم.
-   سيطالب Android بأذونات `CAMERA`/`RECORD_AUDIO` عندما يكون ذلك ممكنًا؛ تفشل الأذونات المرفوضة بـ `*_PERMISSION_REQUIRED`.

## تسجيلات الشاشة (العُقَد)

تعرض العُقَد `screen.record` (mp4). مثال:

```bash
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10 --no-audio
```

ملاحظات:

-   تتطلب `screen.record` أن يكون تطبيق العُقدة في المقدمة.
-   سيعرض Android مطالبة تسجيل الشاشة للنظام قبل التسجيل.
-   يتم تحديد تسجيلات الشاشة إلى `<= 60s`.
-   `--no-audio` تعطيل التقاط الميكروفون (مدعوم على iOS/Android؛ يستخدم macOS تقاطعات صوت النظام).
-   استخدم `--screen ` لتحديد شاشة عند توفر شاشات متعددة.

## الموقع (العُقَد)

تعرض العُقدة `location.get` عندما يكون الموقع مفعلًا في الإعدادات. مساعد CLI:

```bash
openclaw nodes location get --node <idOrNameOrIp>
openclaw nodes location get --node <idOrNameOrIp> --accuracy precise --max-age 15000 --location-timeout 10000
```

ملاحظات:

-   الموقع **معطَّل افتراضيًا**.
-   يتطلب "دائمًا" إذنًا من النظام؛ جلب الخلفية هو أفضل جهد.
-   تتضمن الاستجابة خط العرض/خط الطول والدقة (بالأمتار) والطابع الزمني.

## الرسائل النصية (عُقَد Android)

يمكن لعُقَد Android عرض `sms.send` عندما يمنح المستخدم إذن **SMS** ويدعم الجهاز الاتصالات الهاتفية. استدعاء منخفض المستوى:

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command sms.send --params '{"to":"+15555550123","message":"Hello from OpenClaw"}'
```

ملاحظات:

-   يجب قبول مطالبة الإذن على جهاز Android قبل الإعلان عن القدرة.
-   لن تعلن الأجهزة التي تعمل فقط عبر Wi-Fi بدون اتصالات هاتفية عن `sms.send`.

## أوامر جهاز Android والبيانات الشخصية

يمكن لعُقَد Android الإعلان عن عائلات أوامر إضافية عند تمكين القدرات المقابلة. العائلات المتاحة:

-   `device.status`, `device.info`, `device.permissions`, `device.health`
-   `notifications.list`, `notifications.actions`
-   `photos.latest`
-   `contacts.search`, `contacts.add`
-   `calendar.events`, `calendar.add`
-   `motion.activity`, `motion.pedometer`
-   `app.update`

أمثلة استدعاء:

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command device.status --params '{}'
openclaw nodes invoke --node <idOrNameOrIp> --command notifications.list --params '{}'
openclaw nodes invoke --node <idOrNameOrIp> --command photos.latest --params '{"limit":1}'
```

ملاحظات:

-   أوامر الحركة مقيدة بالقدرة بواسطة أجهزة الاستشعار المتاحة.
-   `app.update` مقيدة بالإذن والسياسة بواسطة وقت تشغيل العُقدة.

## أوامر النظام (مضيف العُقدة / عُقدة mac)

تعرض عُقدة macOS `system.run`, `system.notify`, و `system.execApprovals.get/set`. يعرض مضيف العُقدة بدون واجهة `system.run`, `system.which`, و `system.execApprovals.get/set`. أمثلة:

```bash
openclaw nodes run --node <idOrNameOrIp> -- echo "Hello from mac node"
openclaw nodes notify --node <idOrNameOrIp> --title "Ping" --body "Gateway ready"
```

ملاحظات:

-   تُرجع `system.run` stdout/stderr/رمز الخروج في الحمولة.
-   تحترم `system.notify` حالة إذن الإشعارات في تطبيق macOS.
-   يستخدم `platform` / `deviceFamily` للعُقدة غير المعروفة قائمة سماح افتراضية محافظة تستثني `system.run` و `system.which`. إذا كنت تحتاج عمدًا إلى تلك الأوامر لمنصة غير معروفة، فأضفها صراحةً عبر `gateway.nodes.allowCommands`.
-   تدعم `system.run` `--cwd`, `--env KEY=VAL`, `--command-timeout`, و `--needs-screen-recording`.
-   بالنسبة لبرامج غلاف shell (`bash|sh|zsh ... -c/-lc`)، يتم تقليل قيم `--env` ذات النطاق المطلوب إلى قائمة سماح صريحة (`TERM`, `LANG`, `LC_*`, `COLORTERM`, `NO_COLOR`, `FORCE_COLOR`).
-   لقرارات السماح دائمًا في وضع قائمة السماح، تستمر برامج غلاف الإرسال المعروفة (`env`, `nice`, `nohup`, `stdbuf`, `timeout`) في مسارات الملفات القابلة للتنفيذ الداخلية بدلاً من مسارات الغلاف. إذا كان فك الغلاف غير آمن، فلن يستمر أي مدخل في قائمة السماح تلقائيًا.
-   على مضيفي العُقدة Windows في وضع قائمة السماح، تتطلب عمليات التشغيل عبر غلاف shell باستخدام `cmd.exe /c` موافقة (المدخل في قائمة السماح وحده لا يسمح تلقائيًا بصيغة الغلاف).
-   تدعم `system.notify` `--priority <passive|active|timeSensitive>` و `--delivery <system|overlay|auto>`.
-   يتجاهل مضيفو العُقدة تجاوزات `PATH` ويزيل مفاتيح بدء التشغيل/الغلاف الخطرة (`DYLD_*`, `LD_*`, `NODE_OPTIONS`, `PYTHON*`, `PERL*`, `RUBYOPT`, `SHELLOPTS`, `PS4`). إذا كنت بحاجة إلى مدخلات PATH إضافية، فقم بتكوين بيئة خدمة مضيف العُقدة (أو تثبيت الأدوات في مواقع قياسية) بدلاً من تمرير `PATH` عبر `--env`.
-   في وضع عُقدة macOS، تكون `system.run` مقيدة بموافقات التنفيذ في تطبيق macOS (الإعدادات → موافقات التنفيذ). تتصرف ask/allowlist/full بنفس طريقة مضيف العُقدة بدون واجهة؛ تُرجع المطالبات المرفوضة `SYSTEM_RUN_DENIED`.
-   على مضيف العُقدة بدون واجهة، تكون `system.run` مقيدة بموافقات التنفيذ (`~/.openclaw/exec-approvals.json`).

## ربط exec بالعُقدة

عند توفر عُقَد متعددة، يمكنك ربط exec بعُقدة محددة. هذا يحدد العُقدة الافتراضية لـ `exec host=node` (ويمكن تجاوزها لكل وكيل). الافتراضي العام:

```bash
openclaw config set tools.exec.node "node-id-or-name"
```

تجاوز لكل وكيل:

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

إلغاء التعيين للسماح بأي عُقدة:

```bash
openclaw config unset tools.exec.node
openclaw config unset agents.list[0].tools.exec.node
```

## خريطة الأذونات

قد تتضمن العُقَد خريطة `permissions` في `node.list` / `node.describe`، مفتاحها اسم الإذن (مثل `screenRecording`, `accessibility`) بقيم منطقية (`true` = مُمنَح).

## مضيف العُقدة بدون واجهة (عبر المنصات)

يمكن لـ OpenClaw تشغيل **مضيف عُقدة بدون واجهة** (بدون واجهة مستخدم) يتصل بـ WebSocket الخاص بالبوابة ويعرض `system.run` / `system.which`. هذا مفيد على Linux/Windows أو لتشغيل عُقدة صغيرة بجانب خادم. ابدأه:

```bash
openclaw node run --host <gateway-host> --port 18789
```

ملاحظات:

-   لا يزال الإقران مطلوبًا (ستعرض البوابة مطالبة إقران جهاز).
-   يخزن مضيف العُقدة معرف العُقدة والرمز واسم العرض ومعلومات اتصال البوابة في `~/.openclaw/node.json`.
-   يتم فرض موافقات التنفيذ محليًا عبر `~/.openclaw/exec-approvals.json` (انظر [موافقات Exec](./tools/exec-approvals.md)).
-   على macOS، ينفذ مضيف العُقدة بدون واجهة `system.run` محليًا افتراضيًا. عيّن `OPENCLAW_NODE_EXEC_HOST=app` لتوجيه `system.run` عبر مضيف تنفيذ تطبيق المرافق؛ أضف `OPENCLAW_NODE_EXEC_FALLBACK=0` لتطلب مضيف التطبيق والفشل مغلقًا إذا كان غير متاح.
-   أضف `--tls` / `--tls-fingerprint` عندما يستخدم Gateway WS TLS.

## وضع عُقدة Mac

-   يتصل تطبيق شريط القائمة macOS بخادم WS الخاص بالبوابة كعُقدة (وبالتالي تعمل `openclaw nodes …` ضد هذا الجهاز Mac).
-   في الوضع البعيد، يفتح التطبيق نفق SSH لمنفذ البوابة ويتصل بـ `localhost`.

[مراقبة المصادقة](./automation/auth-monitoring.md)[استكشاف أخطاء العُقدة وإصلاحها](./nodes/troubleshooting.md)