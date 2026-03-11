

  المزودون

  
# Mistral

يدعم OpenClaw خدمة Mistral لكل من توجيه النماذج النصية/المرئية (`mistral/...`) وتحويل الصوت إلى نص عبر Voxtral في فهم الوسائط. يمكن أيضًا استخدام Mistral لتضمينات الذاكرة (`memorySearch.provider = "mistral"`).

## الإعداد عبر سطر الأوامر

```bash
openclaw onboard --auth-choice mistral-api-key
# أو بشكل غير تفاعلي
openclaw onboard --mistral-api-key "$MISTRAL_API_KEY"
```

## مقتطف التكوين (مزود LLM)

```json
{
  env: { MISTRAL_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "mistral/mistral-large-latest" } } },
}
```

## مقتطف التكوين (تحويل الصوت إلى نص باستخدام Voxtral)

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "mistral", model: "voxtral-mini-latest" }],
      },
    },
  },
}
```

## ملاحظات

-   يستخدم المصادقة مع Mistral المتغير `MISTRAL_API_KEY`.
-   عنوان URL الأساسي للمزود هو افتراضيًا `https://api.mistral.ai/v1`.
-   النموذج الافتراضي أثناء الإعداد الأولي هو `mistral/mistral-large-latest`.
-   النموذج الصوتي الافتراضي لفهم الوسائط باستخدام Mistral هو `voxtral-mini-latest`.
-   مسار تحويل الوسائط الصوتية يستخدم `/v1/audio/transcriptions`.
-   مسار تضمينات الذاكرة يستخدم `/v1/embeddings` (النموذج الافتراضي: `mistral-embed`).

[Moonshot AI](./moonshot.md)[NVIDIA](./nvidia.md)