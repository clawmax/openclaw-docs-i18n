title: "دليل أدوات وكيل OpenClaw لاستدعاء وظائف LLM"
description: "تعلم كيفية تسجيل أدوات الوكيل المطلوبة والاختيارية (وظائف JSON-schema) لـ LLMs في OpenClaw. قم بتكوين قوائم السماح للأدوات، وقوائم الحظر، وسياسات التنفيذ."
keywords: ["أدوات الوكيل", "إضافات openclaw", "استدعاء وظائف llm", "قائمة السماح للأدوات", "أدوات اختيارية", "تطوير الإضافات", "json-schema", "تكوين الوكيل"]
---

  الإضافات

  
# أدوات وكيل الإضافة

يمكن لإضافات OpenClaw تسجيل **أدوات وكيل** (وظائف JSON‑schema) يتم عرضها على LLM أثناء تشغيل الوكيل. يمكن أن تكون الأدوات **مطلوبة** (متاحة دائمًا) أو **اختيارية** (يتم تفعيلها يدويًا). يتم تكوين أدوات الوكيل تحت `tools` في الإعدادات الرئيسية، أو لكل وكيل تحت `agents.list[].tools`. تتحكم سياسة قائمة السماح/الحظر في الأدوات التي يمكن للوكيل استدعاؤها.

## أداة أساسية

```typescript
import { Type } from "@sinclair/typebox";

export default function (api) {
  api.registerTool({
    name: "my_tool",
    description: "قم بعمل شيء",
    parameters: Type.Object({
      input: Type.String(),
    }),
    async execute(_id, params) {
      return { content: [{ type: "text", text: params.input }] };
    },
  });
}
```

## أداة اختيارية (يتم تفعيلها يدويًا)

الأدوات الاختيارية **لا** يتم تمكينها تلقائيًا أبدًا. يجب على المستخدمين إضافتها إلى قائمة السماح للوكيل.

```bash
export default function (api) {
  api.registerTool(
    {
      name: "workflow_tool",
      description: "تشغيل سير عمل محلي",
      parameters: {
        type: "object",
        properties: {
          pipeline: { type: "string" },
        },
        required: ["pipeline"],
      },
      async execute(_id, params) {
        return { content: [{ type: "text", text: params.pipeline }] };
      },
    },
    { optional: true },
  );
}
```

قم بتمكين الأدوات الاختيارية في `agents.list[].tools.allow` (أو `tools.allow` العام):

```json
{
  agents: {
    list: [
      {
        id: "main",
        tools: {
          allow: [
            "workflow_tool", // اسم أداة محدد
            "workflow", // معرف الإضافة (يفعل جميع أدوات تلك الإضافة)
            "group:plugins", // جميع أدوات الإضافات
          ],
        },
      },
    ],
  },
}
```

مقابض تكوين أخرى تؤثر على توفر الأداة:

-   قوائم السماح التي تذكر فقط أدوات الإضافات تعامل كتفعيل للإضافة؛ تبقى أدوات النواة مفعلة ما لم تقم بتضمين أدوات النواة أو مجموعاتها في قائمة السماح.
-   `tools.profile` / `agents.list[].tools.profile` (قائمة السماح الأساسية)
-   `tools.byProvider` / `agents.list[].tools.byProvider` (السماح/الحظر الخاص بالمزود)
-   `tools.sandbox.tools.*` (سياسة أداة الحماية عند التشغيل في بيئة معزولة)

## القواعد + النصائح

-   يجب **ألا** تتعارض أسماء الأدوات مع أسماء أدوات النواة؛ يتم تخطي الأدوات المتعارضة.
-   يجب ألا تتعارض معرفات الإضافات المستخدمة في قوائم السماح مع أسماء أدوات النواة.
-   يُفضل استخدام `optional: true` للأدوات التي تسبب تأثيرات جانبية أو تتطلب ملفات تنفيذية/بيانات اعتماد إضافية.

[بيان الإضافة](./manifest.md)[OpenProse](../prose.md)