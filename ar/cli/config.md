

  أوامر CLI

  
# config

مساعدو الإعدادات: الحصول على/تعيين/إلغاء تعيين/التحقق من صحة القيم عبر المسار وطباعة ملف الإعدادات النشط. التشغيل دون أمر فرعي لفتح معالج الإعدادات (مثل `openclaw configure`).

## أمثلة

```bash
openclaw config file
openclaw config get browser.executablePath
openclaw config set browser.executablePath "/usr/bin/google-chrome"
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
openclaw config unset tools.web.search.apiKey
openclaw config validate
openclaw config validate --json
```

## المسارات

تستخدم المسارات ترميز النقطة أو الأقواس:

```bash
openclaw config get agents.defaults.workspace
openclaw config get agents.list[0].id
```

استخدم فهرس قائمة الوكيل لاستهداف وكيل محدد:

```bash
openclaw config get agents.list
openclaw config set agents.list[1].tools.exec.node "node-id-or-name"
```

## القيم

يتم تحليل القيم كـ JSON5 عندما يكون ذلك ممكنًا؛ وإلا يتم التعامل معها كسلاسل نصية. استخدم `--strict-json` لإلزام التحليل كـ JSON5. `--json` لا يزال مدعومًا كاسم قديم.

```bash
openclaw config set agents.defaults.heartbeat.every "0m"
openclaw config set gateway.port 19001 --strict-json
openclaw config set channels.whatsapp.groups '["*"]' --strict-json
```

## الأوامر الفرعية

-   `config file`: طباعة مسار ملف الإعدادات النشط (يتم حله من `OPENCLAW_CONFIG_PATH` أو الموقع الافتراضي).

أعد تشغيل البوابة بعد التعديلات.

## التحقق من الصحة

التحقق من صحة الإعدادات الحالية مقابل المخطط النشط دون بدء تشغيل البوابة.

```bash
openclaw config validate
openclaw config validate --json
```

[completion](./completion.md)[configure](./configure.md)

---