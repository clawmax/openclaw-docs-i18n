title: "قائمة أوامر الجلسات في OpenClaw CLI ودليل التنظيف"
description: "تعلم كيفية عرض وإدارة جلسات المحادثة المخزنة باستخدام OpenClaw CLI. اعرض بيانات الجلسات، وقم بتنفيذ صيانة التنظيف، واستخدم خيارات إخراج JSON."
keywords: ["جلسات openclaw", "أمر جلسات cli", "تنظيف الجلسات", "جلسات المحادثة", "صيانة الجلسات", "أوامر cli", "إدارة الجلسات", "إخراج json"]
---

  أوامر CLI

  
# sessions

اعرض جلسات المحادثة المخزنة.

```bash
openclaw sessions
openclaw sessions --agent work
openclaw sessions --all-agents
openclaw sessions --active 120
openclaw sessions --json
```

اختيار النطاق:

-   الافتراضي: مخزن الوكيل الافتراضي المُهيأ
-   `--agent `: مخزن وكيل واحد مُهيأ
-   `--all-agents`: تجميع جميع مخازن الوكلاء المُهيأة
-   `--store `: مسار مخزن صريح (لا يمكن دمجه مع `--agent` أو `--all-agents`)

أمثلة JSON: `openclaw sessions --all-agents --json`:

```json
{
  "path": null,
  "stores": [
    { "agentId": "main", "path": "/home/user/.openclaw/agents/main/sessions/sessions.json" },
    { "agentId": "work", "path": "/home/user/.openclaw/agents/work/sessions/sessions.json" }
  ],
  "allAgents": true,
  "count": 2,
  "activeMinutes": null,
  "sessions": [
    { "agentId": "main", "key": "agent:main:main", "model": "gpt-5" },
    { "agentId": "work", "key": "agent:work:main", "model": "claude-opus-4-5" }
  ]
}
```

## صيانة التنظيف

قم بتشغيل الصيانة الآن (بدلاً من انتظار دورة الكتابة التالية):

```bash
openclaw sessions cleanup --dry-run
openclaw sessions cleanup --agent work --dry-run
openclaw sessions cleanup --all-agents --dry-run
openclaw sessions cleanup --enforce
openclaw sessions cleanup --enforce --active-key "agent:main:telegram:dm:123"
openclaw sessions cleanup --json
```

يستخدم `openclaw sessions cleanup` إعدادات `session.maintenance` من التكوين:

-   ملاحظة النطاق: يحافظ `openclaw sessions cleanup` على مخازن/نصوص الجلسات فقط. لا يقوم بتقليم سجلات تشغيل cron (`cron/runs/.jsonl`)، والتي تتم إدارتها بواسطة `cron.runLog.maxBytes` و `cron.runLog.keepLines` في [تكوين Cron](../automation/cron-jobs.md#configuration) ويتم شرحها في [صيانة Cron](../automation/cron-jobs.md#maintenance).
-   `--dry-run`: معاينة عدد الإدخالات التي سيتم تقليمها/تحديدها دون الكتابة.
    -   في وضع النص، يطبع التشغيل التجريبي جدول إجراءات لكل جلسة (`Action`, `Key`, `Age`, `Model`, `Flags`) حتى تتمكن من رؤية ما سيتم الاحتفاظ به مقابل ما سيتم إزالته.
-   `--enforce`: تطبيق الصيانة حتى عندما يكون `session.maintenance.mode` هو `warn`.
-   `--active-key `: حماية مفتاح نشط محدد من الإزاحة بسبب ميزانية القرص.
-   `--agent `: تشغيل التنظيف لمخزن وكيل واحد مُهيأ.
-   `--all-agents`: تشغيل التنظيف لجميع مخازن الوكلاء المُهيأة.
-   `--store `: التشغيل ضد ملف `sessions.json` محدد.
-   `--json`: طباعة ملخص JSON. مع `--all-agents`، يتضمن الإخراج ملخصًا واحدًا لكل مخزن.

`openclaw sessions cleanup --all-agents --dry-run --json`:

```json
{
  "allAgents": true,
  "mode": "warn",
  "dryRun": true,
  "stores": [
    {
      "agentId": "main",
      "storePath": "/home/user/.openclaw/agents/main/sessions/sessions.json",
      "beforeCount": 120,
      "afterCount": 80,
      "pruned": 40,
      "capped": 0
    },
    {
      "agentId": "work",
      "storePath": "/home/user/.openclaw/agents/work/sessions/sessions.json",
      "beforeCount": 18,
      "afterCount": 18,
      "pruned": 0,
      "capped": 0
    }
  ]
}
```

ذات صلة:

-   تكوين الجلسة: [مرجع التكوين](../gateway/configuration-reference.md#session)

[security](./security.md)[setup](./setup.md)