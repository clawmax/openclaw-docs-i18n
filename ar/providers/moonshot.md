title: "تكوين موفر Moonshot AI و Kimi Coding لـ OpenClaw"
description: "تعلم كيفية إعداد موفر Moonshot AI و Kimi Coding في OpenClaw. قم بتكوين مفاتيح API والنقاط الطرفية ومراجع النماذج مثل moonshot/kimi-k2.5."
keywords: ["moonshot ai", "kimi api", "موفر openclaw", "kimi coding", "kimi k2.5", "api متوافق مع openai", "تكوين api", "إعداد النموذج"]
---

  الموفرون

  
# Moonshot AI

يوفر Moonshot واجهة برمجة تطبيقات Kimi مع نقاط طرفية متوافقة مع OpenAI. قم بتكوين الموفر وعيّن النموذج الافتراضي إلى `moonshot/kimi-k2.5`، أو استخدم Kimi Coding مع `kimi-coding/k2p5`. معرفات نموذج Kimi K2 الحالية:

-   `kimi-k2.5`
-   `kimi-k2-0905-preview`
-   `kimi-k2-turbo-preview`
-   `kimi-k2-thinking`
-   `kimi-k2-thinking-turbo`

```bash
openclaw onboard --auth-choice moonshot-api-key
```

Kimi Coding:

```bash
openclaw onboard --auth-choice kimi-code-api-key
```

ملاحظة: Moonshot و Kimi Coding هما موفران منفصلان. المفاتيح غير قابلة للتبادل، والنقاط الطرفية مختلفة، ومراجع النماذج مختلفة (يستخدم Moonshot `moonshot/...`، ويستخدم Kimi Coding `kimi-coding/...`).

## مقتطف التكوين (Moonshot API)

```json
{
  env: { MOONSHOT_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "moonshot/kimi-k2.5" },
      models: {
        // moonshot-kimi-k2-aliases:start
        "moonshot/kimi-k2.5": { alias: "Kimi K2.5" },
        "moonshot/kimi-k2-0905-preview": { alias: "Kimi K2" },
        "moonshot/kimi-k2-turbo-preview": { alias: "Kimi K2 Turbo" },
        "moonshot/kimi-k2-thinking": { alias: "Kimi K2 Thinking" },
        "moonshot/kimi-k2-thinking-turbo": { alias: "Kimi K2 Thinking Turbo" },
        // moonshot-kimi-k2-aliases:end
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [
          // moonshot-kimi-k2-models:start
          {
            id: "kimi-k2.5",
            name: "Kimi K2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
          {
            id: "kimi-k2-0905-preview",
            name: "Kimi K2 0905 Preview",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
          {
            id: "kimi-k2-turbo-preview",
            name: "Kimi K2 Turbo",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
          {
            id: "kimi-k2-thinking",
            name: "Kimi K2 Thinking",
            reasoning: true,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
          {
            id: "kimi-k2-thinking-turbo",
            name: "Kimi K2 Thinking Turbo",
            reasoning: true,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
          // moonshot-kimi-k2-models:end
        ],
      },
    },
  },
}
```

## Kimi Coding

```json
{
  env: { KIMI_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "kimi-coding/k2p5" },
      models: {
        "kimi-coding/k2p5": { alias: "Kimi K2.5" },
      },
    },
  },
}
```

## ملاحظات

-   تستخدم مراجع نماذج Moonshot `moonshot/`. تستخدم مراجع نماذج Kimi Coding `kimi-coding/`.
-   يمكنك تجاوز التسعير وبيانات وصف السياق في `models.providers` إذا لزم الأمر.
-   إذا نشر Moonshot حدود سياق مختلفة لنموذج ما، فاضبط `contextWindow` وفقًا لذلك.
-   استخدم `https://api.moonshot.ai/v1` للنقطة الطرفية الدولية، و `https://api.moonshot.cn/v1` للنقطة الطرفية في الصين.

[MiniMax](./minimax.md)[Mistral](./mistral.md)