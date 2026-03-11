

  المزودون

  
# OpenCode Zen

OpenCode Zen هي **قائمة مختارة من النماذج** التي يوصي بها فريق OpenCode لوكلاء الترميز. إنها مسار وصول اختياري للنماذج المستضافة يستخدم مفتاح API وموفر `opencode`. Zen حاليًا في مرحلة تجريبية.

## الإعداد عبر سطر الأوامر

```bash
openclaw onboard --auth-choice opencode-zen
# أو بشكل غير تفاعلي
openclaw onboard --opencode-zen-api-key "$OPENCODE_API_KEY"
```

## مقتطف التكوين

```json
{
  env: { OPENCODE_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-6" } } },
}
```

## ملاحظات

-   `OPENCODE_ZEN_API_KEY` مدعوم أيضًا.
-   تقوم بتسجيل الدخول إلى Zen، وإضافة تفاصيل الفوترة، ونسخ مفتاح API الخاص بك.
-   OpenCode Zen يتقاضى رسومًا حسب الطلب؛ تحقق من لوحة تحكم OpenCode للحصول على التفاصيل.

[OpenAI](./openai.md)[OpenRouter](./openrouter.md)

---