

  البوابة

  
# كتاب تشغيل البوابة

استخدم هذه الصفحة لتشغيل اليوم الأول وعمليات اليوم الثاني لخدمة البوابة.

## بدء تشغيل محلي في 5 دقائق

### الخطوة 1: تشغيل البوابة

```bash
openclaw gateway --port 18789
# debug/trace mirrored to stdio
openclaw gateway --port 18789 --verbose
# force-kill listener on selected port, then start
openclaw gateway --force
```

### الخطوة 2: التحقق من صحة الخدمة

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
```

المستوى الأساسي الصحي: `Runtime: running` و `RPC probe: ok`.

### الخطوة 3: التحقق من جاهزية القنوات

```bash
openclaw channels status --probe
```

 

> **ℹ️** يراقب إعادة تحميل تكوين البوابة مسار ملف التكوين النشط (المحل من إعدادات الملف الشخصي/الحالة الافتراضية، أو `OPENCLAW_CONFIG_PATH` عند تعيينه). الوضع الافتراضي هو `gateway.reload.mode="hybrid"`.

## نموذج وقت التشغيل

-   عملية واحدة دائمة التشغيل للتوجيه، ومستوى التحكم، واتصالات القنوات.
-   منفذ واحد متعدد الإرسال لـ:
    -   WebSocket للتحكم/RPC
    -   واجهات برمجة التطبيقات HTTP (متوافقة مع OpenAI، Responses، استدعاء الأدوات)
    -   واجهة التحكم والخطافات
-   وضع الربط الافتراضي: `loopback`.
-   المصادقة مطلوبة افتراضيًا (`gateway.auth.token` / `gateway.auth.password`، أو `OPENCLAW_GATEWAY_TOKEN` / `OPENCLAW_GATEWAY_PASSWORD`).

### أولوية المنفذ والربط

| الإعداد | ترتيب الحل |
| --- | --- |
| منفذ البوابة | `--port` → `OPENCLAW_GATEWAY_PORT` → `gateway.port` → `18789` |
| وضع الربط | CLI/تجاوز → `gateway.bind` → `loopback` |

### أوضاع إعادة التحميل الساخنة

| `gateway.reload.mode` | السلوك |
| --- | --- |
| `off` | لا يوجد إعادة تحميل للتكوين |
| `hot` | تطبيق التغييرات الآمنة للساخنة فقط |
| `restart` | إعادة التشغيل عند التغييرات التي تتطلب إعادة تحميل |
| `hybrid` (الافتراضي) | التطبيق الساخن عندما يكون آمنًا، إعادة التشغيل عندما يكون مطلوبًا |

## مجموعة أوامر المشغل

```bash
openclaw gateway status
openclaw gateway status --deep
openclaw gateway status --json
openclaw gateway install
openclaw gateway restart
openclaw gateway stop
openclaw secrets reload
openclaw logs --follow
openclaw doctor
```

## الوصول عن بُعد

الطريقة المفضلة: Tailscale/VPN. الطريقة الاحتياطية: نفق SSH.

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

ثم قم بتوصيل العملاء بـ `ws://127.0.0.1:18789` محليًا.

> **⚠️** إذا تم تكوين مصادقة البوابة، فلا يزال على العملاء إرسال بيانات المصادقة (`token`/`password`) حتى عبر أنفاق SSH.

 انظر: [البوابة البعيدة](./gateway/remote.md)، [المصادقة](./gateway/authentication.md)، [Tailscale](./gateway/tailscale.md).

## الإشراف ودورة حياة الخدمة

استخدم عمليات التشغيل الخاضعة للإشراف لموثوقية تشبه الإنتاج.

].service\nopenclaw gateway status', lang: 'bash' }, { label: 'Linux (system service)', code: 'sudo systemctl daemon-reload\nsudo systemctl enable --now openclaw-gateway[-].service', lang: 'bash' }]} />

## بوابات متعددة على مضيف واحد

يجب أن تعمل معظم الإعدادات **بوابة واحدة**. استخدم متعددة فقط للعزل/التكرار الصارم (على سبيل المثال ملف تعريف إنقاذ). قائمة التحقق لكل مثيل:

-   `gateway.port` فريد
-   `OPENCLAW_CONFIG_PATH` فريد
-   `OPENCLAW_STATE_DIR` فريد
-   `agents.defaults.workspace` فريد

مثال:

```
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

انظر: [بوابات متعددة](./gateway/multiple-gateways.md).

### المسار السريع لملف تعريف التطوير

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
openclaw --dev status
```

تتضمن الإعدادات الافتراضية حالة/تكوين معزول ومنفذ البوابة الأساسي `19001`.

## مرجع سريع للبروتوكول (من وجهة نظر المشغل)

-   يجب أن تكون أول إطار للعميل هو `connect`.
-   تُرجع البوابة لقطة `hello-ok` (`presence`, `health`, `stateVersion`, `uptimeMs`, حدود/سياسة).
-   الطلبات: `req(method, params)` → `res(ok/payload|error)`.
-   الأحداث الشائعة: `connect.challenge`, `agent`, `chat`, `presence`, `tick`, `health`, `heartbeat`, `shutdown`.

تشغيلات الوكيل ذات مرحلتين:

1.  قبول فوري (`status:"accepted"`)
2.  استجابة إكمال نهائية (`status:"ok"|"error"`)، مع أحداث `agent` مُتدفقة فيما بينها.

انظر وثائق البروتوكول الكاملة: [بروتوكول البوابة](./gateway/protocol.md).

## فحوصات التشغيل

### الحيوية

-   افتح WS وأرسل `connect`.
-   توقع استجابة `hello-ok` مع لقطة.

### الجاهزية

```bash
openclaw gateway status
openclaw channels status --probe
openclaw health
```

### استعادة الفجوة

لا يتم إعادة تشغيل الأحداث. عند وجود فجوات في التسلسل، قم بتحديث الحالة (`health`, `system-presence`) قبل المتابعة.

## توقعات الأعطال الشائعة

| التوقيع | المشكلة المحتملة |
| --- | --- |
| `refusing to bind gateway ... without auth` | ربط غير loopback بدون رمز/كلمة مرور |
| `another gateway instance is already listening` / `EADDRINUSE` | تعارض في المنفذ |
| `Gateway start blocked: set gateway.mode=local` | التكوين مضبوط على الوضع البعيد |
| `unauthorized` أثناء الاتصال | عدم تطابق المصادقة بين العميل والبوابة |

للسلالم التشخيصية الكاملة، استخدم [استكشاف أخطاء البوابة وإصلاحها](./gateway/troubleshooting.md).

## ضمانات السلامة

-   يفشل عملاء بروتوكول البوابة بسرعة عندما تكون البوابة غير متاحة (لا يوجد مسار احتياطي مباشر للقناة ضمنيًا).
-   يتم رفض الإطارات الأولى غير الصالحة/غير المتصلة وإغلاقها.
-   يرسل الإغلاق المهذب حدث `shutdown` قبل إغلاق المقبس.

* * *

ذات صلة:

-   [استكشاف الأخطاء وإصلاحها](./gateway/troubleshooting.md)
-   [عملية الخلفية](./gateway/background-process.md)
-   [التكوين](./gateway/configuration.md)
-   [الصحة](./gateway/health.md)
-   [الطبيب](./gateway/doctor.md)
-   [المصادقة](./gateway/authentication.md)

[التكوين](./gateway/configuration.md)