title: "Настройка OpenCode Zen для AI-агентов программирования OpenClaw"
description: "Узнайте, как настроить OpenCode Zen — курируемый список моделей для AI-агентов программирования. Настройте свой API-ключ и начните использовать провайдера opencode."
keywords: ["opencode zen", "агенты программирования", "провайдер моделей", "настройка api ключа", "конфигурация openclaw", "claude opus", "хостинг моделей", "бета-доступ"]
---

  Провайдеры

  
# OpenCode Zen

OpenCode Zen — это **курируемый список моделей**, рекомендованных командой OpenCode для агентов программирования. Это опциональный, хостируемый путь доступа к моделям, который использует API-ключ и провайдера `opencode`. Zen в настоящее время находится в бета-версии.

## Настройка через CLI

```bash
openclaw onboard --auth-choice opencode-zen
# или неинтерактивно
openclaw onboard --opencode-zen-api-key "$OPENCODE_API_KEY"
```

## Фрагмент конфигурации

```json
{
  env: { OPENCODE_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-6" } } },
}
```

## Примечания

-   `OPENCODE_ZEN_API_KEY` также поддерживается.
-   Вы входите в Zen, добавляете платежные реквизиты и копируете свой API-ключ.
-   OpenCode Zen тарифицируется за запрос; подробности смотрите в панели управления OpenCode.

[OpenAI](./openai.md)[OpenRouter](./openrouter.md)

---