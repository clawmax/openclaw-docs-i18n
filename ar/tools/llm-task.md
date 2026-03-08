title: "أداة مهمة LLM لإخراج JSON منظم في OpenClaw"
description: "تعلم كيفية استخدام البرنامج المساعد الاختياري LLM Task لتشغيل مهام الذكاء الاصطناعي التي تنتج JSON فقط مع التحقق من المخطط. مثالي لمحركات سير العمل مثل Lobster."
keywords: ["مهمة llm", "التحقق من مخطط json", "برنامج مساعد openclaw", "إخراج llm منظم", "سير عمل lobster", "أداة مهمة الذكاء الاصطناعي", "برنامج مساعد اختياري", "llm لـ json فقط"]
---

  أدوات مدمجة

  
# مهمة LLM

`llm-task` هي **أداة برنامج مساعد اختيارية** تشغل مهمة LLM تنتج JSON فقط وتعيد مخرجات منظمة (مع التحقق من مطابقتها لمخطط JSON بشكل اختياري). هذا مثالي لمحركات سير العمل مثل Lobster: يمكنك إضافة خطوة LLM واحدة دون الحاجة لكتابة كود OpenClaw مخصص لكل سير عمل.

## تمكين البرنامج المساعد

1.  قم بتمكين البرنامج المساعد:

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  }
}
```

2.  أضف الأداة إلى القائمة المسموح بها (يتم تسجيلها بـ `optional: true`):

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": { "allow": ["llm-task"] }
      }
    ]
  }
}
```

## التكوين (اختياري)

```json
{
  "plugins": {
    "entries": {
      "llm-task": {
        "enabled": true,
        "config": {
          "defaultProvider": "openai-codex",
          "defaultModel": "gpt-5.4",
          "defaultAuthProfileId": "main",
          "allowedModels": ["openai-codex/gpt-5.4"],
          "maxTokens": 800,
          "timeoutMs": 30000
        }
      }
    }
  }
}
```

`allowedModels` هي قائمة مسموح بها من سلاسل `provider/model`. إذا تم تعيينها، يتم رفض أي طلب خارج القائمة.

## معلمات الأداة

-   `prompt` (سلسلة نصية، مطلوبة)
-   `input` (أي نوع، اختياري)
-   `schema` (كائن، مخطط JSON اختياري)
-   `provider` (سلسلة نصية، اختياري)
-   `model` (سلسلة نصية، اختياري)
-   `authProfileId` (سلسلة نصية، اختياري)
-   `temperature` (رقم، اختياري)
-   `maxTokens` (رقم، اختياري)
-   `timeoutMs` (رقم، اختياري)

## المخرجات

تُرجع `details.json` الذي يحتوي على JSON المُحلّل (وتتحقق من مطابقته لـ `schema` عند توفيره).

## مثال: خطوة سير عمل Lobster

```
openclaw.invoke --tool llm-task --action json --args-json '{
  "prompt": "Given the input email, return intent and draft.",
  "input": {
    "subject": "Hello",
    "body": "Can you help?"
  },
  "schema": {
    "type": "object",
    "properties": {
      "intent": { "type": "string" },
      "draft": { "type": "string" }
    },
    "required": ["intent", "draft"],
    "additionalProperties": false
  }
}'
```

## ملاحظات أمانية

-   الأداة **تنتج JSON فقط** وتوجه النموذج لإخراج JSON فقط (بدون إطارات كود، بدون تعليقات).
-   لا يتم تعريض أي أدوات للنموذج خلال هذه التشغيلة.
-   تعامل مع المخرجات على أنها غير موثوقة ما لم تتحقق منها باستخدام `schema`.
-   ضع خطوات الموافقة قبل أي خطوة لها تأثيرات جانبية (إرسال، نشر، تنفيذ).

[Firecrawl](./firecrawl.md)[Lobster](./lobster.md)

---