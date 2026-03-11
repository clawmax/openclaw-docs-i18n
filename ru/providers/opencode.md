

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