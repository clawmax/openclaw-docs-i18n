

  تنسيق الوكلاء

  
# بيئة الحماية متعددة الوكلاء والأدوات

## نظرة عامة

يمكن لكل وكيل في إعداد متعدد الوكلاء الآن أن يكون له:

-   **تكوين بيئة الحماية** الخاص به (`agents.list[].sandbox` يتجاوز `agents.defaults.sandbox`)
-   **قيود الأدوات** (`tools.allow` / `tools.deny`، بالإضافة إلى `agents.list[].tools`)

هذا يسمح لك بتشغيل وكلاء متعددين بملفات تعريف أمان مختلفة:

-   مساعد شخصي مع وصول كامل
-   وكلاء عائلية/عمل مع أدوات مقيدة
-   وكلاء عامة في بيئات حماية

`setupCommand` يتبع `sandbox.docker` (عالميًا أو لكل وكيل) ويعمل مرة واحدة عند إنشاء الحاوية. المصادقة لكل وكيل: كل وكيل يقرأ من مخزن المصادقة الخاص بـ `agentDir` الخاص به على المسار:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

**لا** تتم مشاركة بيانات الاعتماد بين الوكلاء. لا تعيد استخدام `agentDir` عبر وكلاء مختلفين. إذا كنت تريد مشاركة بيانات الاعتماد، انسخ `auth-profiles.json` إلى `agentDir` الخاص بالوكيل الآخر. لمعرفة كيفية عمل الحماية أثناء التشغيل، راجع [الحماية](../gateway/sandboxing.md). لتصحيح "لماذا تم حظر هذا؟"، راجع [الحماية مقابل سياسة الأداة مقابل المرفوعة](../gateway/sandbox-vs-tool-policy-vs-elevated.md) و `openclaw sandbox explain`.

* * *

## أمثلة التكوين

### المثال 1: وكيل شخصي + وكيل عائلي مقيد

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "name": "المساعد الشخصي",
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "family",
        "name": "بوت العائلة",
        "workspace": "~/.openclaw/workspace-family",
        "sandbox": {
          "mode": "all",
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch", "process", "browser"]
        }
      }
    ]
  },
  "bindings": [
    {
      "agentId": "family",
      "match": {
        "provider": "whatsapp",
        "accountId": "*",
        "peer": {
          "kind": "group",
          "id": "120363424282127706@g.us"
        }
      }
    }
  ]
}
```

**النتيجة:**

-   وكيل `main`: يعمل على المضيف، وصول كامل للأدوات
-   وكيل `family`: يعمل في Docker (حاوية واحدة لكل وكيل)، أداة `read` فقط

* * *

### المثال 2: وكيل عمل مع بيئة حماية مشتركة

```json
{
  "agents": {
    "list": [
      {
        "id": "personal",
        "workspace": "~/.openclaw/workspace-personal",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "work",
        "workspace": "~/.openclaw/workspace-work",
        "sandbox": {
          "mode": "all",
          "scope": "shared",
          "workspaceRoot": "/tmp/work-sandboxes"
        },
        "tools": {
          "allow": ["read", "write", "apply_patch", "exec"],
          "deny": ["browser", "gateway", "discord"]
        }
      }
    ]
  }
}
```

* * *

### المثال 2ب: ملف تعريف برمجة عالمي + وكيل للرسائل فقط

```json
{
  "tools": { "profile": "coding" },
  "agents": {
    "list": [
      {
        "id": "support",
        "tools": { "profile": "messaging", "allow": ["slack"] }
      }
    ]
  }
}
```

**النتيجة:**

-   الوكلاء الافتراضية تحصل على أدوات البرمجة
-   وكيل `support` للرسائل فقط (+ أداة Slack)

* * *

### المثال 3: أوضاع حماية مختلفة لكل وكيل

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main", // الإعداد الافتراضي العالمي
        "scope": "session"
      }
    },
    "list": [
      {
        "id": "main",
        "workspace": "~/.openclaw/workspace",
        "sandbox": {
          "mode": "off" // تجاوز: الرئيسي لا يتم حمايته أبدًا
        }
      },
      {
        "id": "public",
        "workspace": "~/.openclaw/workspace-public",
        "sandbox": {
          "mode": "all", // تجاوز: العام يتم حمايته دائمًا
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch"]
        }
      }
    ]
  }
}
```

* * *

## أولوية التكوين

عند وجود تكوينات عالمية (`agents.defaults.*`) وخاصة بالوكيل (`agents.list[].*`) معًا:

### تكوين الحماية

الإعدادات الخاصة بالوكيل تتجاوز العالمية:

```
agents.list[].sandbox.mode > agents.defaults.sandbox.mode
agents.list[].sandbox.scope > agents.defaults.sandbox.scope
agents.list[].sandbox.workspaceRoot > agents.defaults.sandbox.workspaceRoot
agents.list[].sandbox.workspaceAccess > agents.defaults.sandbox.workspaceAccess
agents.list[].sandbox.docker.* > agents.defaults.sandbox.docker.*
agents.list[].sandbox.browser.* > agents.defaults.sandbox.browser.*
agents.list[].sandbox.prune.* > agents.defaults.sandbox.prune.*
```

**ملاحظات:**

-   `agents.list[].sandbox.{docker,browser,prune}.*` يتجاوز `agents.defaults.sandbox.{docker,browser,prune}.*` لذلك الوكيل (يتم تجاهله عندما يكون نطاق الحماية `"shared"`).

### قيود الأدوات

ترتيب التصفية هو:

1.  **ملف تعريف الأداة** (`tools.profile` أو `agents.list[].tools.profile`)
2.  **ملف تعريف أداة المزود** (`tools.byProvider[provider].profile` أو `agents.list[].tools.byProvider[provider].profile`)
3.  **سياسة الأداة العالمية** (`tools.allow` / `tools.deny`)
4.  **سياسة أداة المزود** (`tools.byProvider[provider].allow/deny`)
5.  **سياسة الأداة الخاصة بالوكيل** (`agents.list[].tools.allow/deny`)
6.  **سياسة مزود الوكيل** (`agents.list[].tools.byProvider[provider].allow/deny`)
7.  **سياسة أداة الحماية** (`tools.sandbox.tools` أو `agents.list[].tools.sandbox.tools`)
8.  **سياسة أداة الوكلاء الفرعية** (`tools.subagents.tools`، إذا كان مطبقًا)

يمكن لكل مستوى تقييد الأدوات بشكل أكبر، لكن لا يمكنه منح أدوات تم رفضها من مستويات سابقة. إذا تم تعيين `agents.list[].tools.sandbox.tools`، فإنه يحل محل `tools.sandbox.tools` لذلك الوكيل. إذا تم تعيين `agents.list[].tools.profile`، فإنه يتجاوز `tools.profile` لذلك الوكيل. تقبل مفاتيح أداة المزود إما `provider` (مثل `google-antigravity`) أو `provider/model` (مثل `openai/gpt-5.2`).

### مجموعات الأدوات (اختصارات)

تدعم سياسات الأدوات (العالمية، الوكيل، الحماية) إدخالات `group:*` التي تتوسع إلى أدوات متعددة ملموسة:

-   `group:runtime`: `exec`, `bash`, `process`
-   `group:fs`: `read`, `write`, `edit`, `apply_patch`
-   `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
-   `group:memory`: `memory_search`, `memory_get`
-   `group:ui`: `browser`, `canvas`
-   `group:automation`: `cron`, `gateway`
-   `group:messaging`: `message`
-   `group:nodes`: `nodes`
-   `group:openclaw`: جميع أدوات OpenClaw المدمجة (تستثني إضافات المزود)

### الوضع المرفوع

`tools.elevated` هو الأساس العالمي (قائمة السماح المستندة إلى المرسل). يمكن لـ `agents.list[].tools.elevated` تقييد الوضع المرفوع بشكل أكبر لوكلاء محددين (يجب أن يسمح كلاهما). أنماط التخفيف:

-   ارفض `exec` للوكلاء غير الموثوقة (`agents.list[].tools.deny: ["exec"]`)
-   تجنب إدراج المرسلين الذين يوجهون إلى وكلاء مقيدة
-   عطل الوضع المرفوع عالميًا (`tools.elevated.enabled: false`) إذا كنت تريد تنفيذًا محميًا فقط
-   عطل الوضع المرفوع لكل وكيل (`agents.list[].tools.elevated.enabled: false`) لملفات تعريف حساسة

* * *

## الانتقال من وكيل واحد

**قبل (وكيل واحد):**

```json
{
  "agents": {
    "defaults": {
      "workspace": "~/.openclaw/workspace",
      "sandbox": {
        "mode": "non-main"
      }
    }
  },
  "tools": {
    "sandbox": {
      "tools": {
        "allow": ["read", "write", "apply_patch", "exec"],
        "deny": []
      }
    }
  }
}
```

**بعد (متعدد الوكلاء بملفات تعريف مختلفة):**

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      }
    ]
  }
}
```

يتم ترحيل تكوينات `agent.*` القديمة بواسطة `openclaw doctor`؛ يُفضل استخدام `agents.defaults` + `agents.list` في المستقبل.

* * *

## أمثلة تقييد الأدوات

### وكيل للقراءة فقط

```json
{
  "tools": {
    "allow": ["read"],
    "deny": ["exec", "write", "edit", "apply_patch", "process"]
  }
}
```

### وكيل تنفيذ آمن (بدون تعديلات على الملفات)

```json
{
  "tools": {
    "allow": ["read", "exec", "process"],
    "deny": ["write", "edit", "apply_patch", "browser", "gateway"]
  }
}
```

### وكيل للاتصالات فقط

```json
{
  "tools": {
    "sessions": { "visibility": "tree" },
    "allow": ["sessions_list", "sessions_send", "sessions_history", "session_status"],
    "deny": ["exec", "write", "edit", "apply_patch", "read", "browser"]
  }
}
```

* * *

## خطأ شائع: "non-main"

`agents.defaults.sandbox.mode: "non-main"` يعتمد على `session.mainKey` (الافتراضي `"main"`)، وليس معرف الوكيل. جلسات المجموعة/القناة تحصل دائمًا على مفاتيح خاصة بها، لذلك يتم التعامل معها على أنها non-main وسيتم حمايتها. إذا كنت تريد أن لا يتم حماية وكيل أبدًا، عيّن `agents.list[].sandbox.mode: "off"`.

* * *

## الاختبار

بعد تكوين الحماية متعددة الوكلاء والأدوات:

1.  **تحقق من تحديد الوكيل:**
    
    نسخ
    
    ```bash
    openclaw agents list --bindings
    ```
    
2.  **تحقق من حاويات الحماية:**
    
    نسخ
    
    ```bash
    docker ps --filter "name=openclaw-sbx-"
    ```
    
3.  **اختبر قيود الأدوات:**
    -   أرسل رسالة تتطلب أدوات مقيدة
    -   تحقق من أن الوكيل لا يمكنه استخدام الأدوات المرفوضة
4.  **راقب السجلات:**
    
    نسخ
    
    ```bash
    tail -f "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}/logs/gateway.log" | grep -E "routing|sandbox|tools"
    ```
    

* * *

## استكشاف الأخطاء وإصلاحها

### الوكيل غير محمي رغم mode: "all"

-   تحقق مما إذا كان هناك `agents.defaults.sandbox.mode` عالمي يتجاوزه
-   التكوين الخاص بالوكيل له الأولوية، لذا عيّن `agents.list[].sandbox.mode: "all"`

### الأدوات لا تزال متاحة رغم قائمة الرفض

-   تحقق من ترتيب تصفية الأدوات: عالمي → وكيل → حماية → وكيل فرعي
-   يمكن لكل مستوى فقط تقييد المزيد، وليس منح ما تم رفضه مرة أخرى
-   تحقق من السجلات: `[tools] filtering tools for agent:${agentId}`

### الحاوية غير معزولة لكل وكيل

-   عيّن `scope: "agent"` في تكوين الحماية الخاص بالوكيل
-   الافتراضي هو `"session"` والذي ينشئ حاوية واحدة لكل جلسة

* * *

## انظر أيضًا

-   [التوجيه متعدد الوكلاء](../concepts/multi-agent.md)
-   [تكوين الحماية](../gateway/configuration.md#agentsdefaults-sandbox)
-   [إدارة الجلسات](../concepts/session.md)

[وكلاء ACP](./acp-agents.md)[إنشاء المهارات](./creating-skills.md)