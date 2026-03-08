title: "Настройка модели ИИ MiniMax M2.5 в документации OpenClaw"
description: "Узнайте, как настроить и сконфигурировать модели ИИ MiniMax M2.5 и M2.5 Highspeed в OpenClaw через OAuth, API-ключ или локальный вывод. Включает примеры настройки и устранение неполадок."
keywords: ["minimax m2.5", "конфигурация openclaw", "провайдер модели ии", "minimax highspeed", "совместимый с anthropic api", "план coding", "локальный вывод lm studio", "настройка резервной модели"]
---

  Провайдеры

  
# MiniMax

MiniMax — это компания в области ИИ, разрабатывающая семейство моделей **M2/M2.5**. Текущий релиз, ориентированный на программирование, — **MiniMax M2.5** (23 декабря 2025 г.), созданный для сложных реальных задач. Источник: [Примечания к выпуску MiniMax M2.5](https://www.minimax.io/news/minimax-m25)

## Обзор модели (M2.5)

MiniMax выделяет следующие улучшения в M2.5:

-   Более сильное **многоязыковое программирование** (Rust, Java, Go, C++, Kotlin, Objective-C, TS/JS).
-   Улучшенная **веб/приложений разработка** и качество эстетического вывода (включая нативные мобильные приложения).
-   Улучшенная обработка **составных инструкций** для офисных рабочих процессов, основанная на переплетенном мышлении и интегрированном выполнении ограничений.
-   **Более лаконичные ответы** с меньшим использованием токенов и более быстрыми циклами итераций.
-   Улучшенная совместимость с **фреймворками инструментов/агентов** и управление контекстом (Claude Code, Droid/Factory AI, Cline, Kilo Code, Roo Code, BlackBox).
-   Более качественный вывод **диалогов и технических текстов**.

## MiniMax M2.5 vs MiniMax M2.5 Highspeed

-   **Скорость:** `MiniMax-M2.5-highspeed` — это официальный быстрый тариф в документации MiniMax.
-   **Стоимость:** В ценах MiniMax указана одинаковая стоимость ввода и более высокая стоимость вывода для highspeed.
-   **Совместимость:** OpenClaw по-прежнему принимает устаревшие конфигурации `MiniMax-M2.5-Lightning`, но для новой настройки предпочтительнее использовать `MiniMax-M2.5-highspeed`.

## Выбор способа настройки

### MiniMax OAuth (Coding Plan) — рекомендуется

**Лучше для:** быстрой настройки с MiniMax Coding Plan через OAuth, без необходимости в API-ключе. Включите встроенный плагин OAuth и пройдите аутентификацию:

```bash
openclaw plugins enable minimax-portal-auth  # пропустите, если уже загружен.
openclaw gateway restart  # перезапустите, если шлюз уже работает
openclaw onboard --auth-choice minimax-portal
```

Вам будет предложено выбрать конечную точку:

-   **Global** — международные пользователи (`api.minimax.io`)
-   **CN** — пользователи в Китае (`api.minimaxi.com`)

Подробности см. в [README плагина MiniMax OAuth](https://github.com/openclaw/openclaw/tree/main/extensions/minimax-portal-auth).

### MiniMax M2.5 (API-ключ)

**Лучше для:** размещенный MiniMax с API, совместимым с Anthropic. Настройте через CLI:

-   Выполните `openclaw configure`
-   Выберите **Model/auth**
-   Выберите **MiniMax M2.5**

```json
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "minimax/MiniMax-M2.5" } } },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.5",
            name: "MiniMax M2.5",
            reasoning: true,
            input: ["text"],
            cost: { input: 0.3, output: 1.2, cacheRead: 0.03, cacheWrite: 0.12 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
          {
            id: "MiniMax-M2.5-highspeed",
            name: "MiniMax M2.5 Highspeed",
            reasoning: true,
            input: ["text"],
            cost: { input: 0.3, output: 1.2, cacheRead: 0.03, cacheWrite: 0.12 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

### MiniMax M2.5 как резервная модель (пример)

**Лучше для:** сохранить вашу самую мощную модель последнего поколения в качестве основной, с переходом на MiniMax M2.5 в случае сбоя. В примере ниже в качестве конкретной основной используется Opus; замените на предпочитаемую вами основную модель последнего поколения.

```json
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "primary" },
        "minimax/MiniMax-M2.5": { alias: "minimax" },
      },
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["minimax/MiniMax-M2.5"],
      },
    },
  },
}
```

### Опционально: Локально через LM Studio (вручную)

**Лучше для:** локальный вывод с помощью LM Studio. Мы наблюдали сильные результаты с MiniMax M2.5 на мощном оборудовании (например, настольный ПК/сервер) с использованием локального сервера LM Studio. Настройте вручную через `openclaw.json`:

```json
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.5-gs32" },
      models: { "lmstudio/minimax-m2.5-gs32": { alias: "Minimax" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.5-gs32",
            name: "MiniMax M2.5 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## Настройка через openclaw configure

Используйте интерактивный мастер конфигурации для настройки MiniMax без редактирования JSON:

1.  Выполните `openclaw configure`.
2.  Выберите **Model/auth**.
3.  Выберите **MiniMax M2.5**.
4.  Выберите вашу модель по умолчанию при запросе.

## Параметры конфигурации

-   `models.providers.minimax.baseUrl`: предпочтительно `https://api.minimax.io/anthropic` (совместимый с Anthropic); `https://api.minimax.io/v1` — опционально для полезных данных, совместимых с OpenAI.
-   `models.providers.minimax.api`: предпочтительно `anthropic-messages`; `openai-completions` — опционально для полезных данных, совместимых с OpenAI.
-   `models.providers.minimax.apiKey`: API-ключ MiniMax (`MINIMAX_API_KEY`).
-   `models.providers.minimax.models`: определите `id`, `name`, `reasoning`, `contextWindow`, `maxTokens`, `cost`.
-   `agents.defaults.models`: задайте псевдонимы для моделей, которые вы хотите видеть в разрешенном списке.
-   `models.mode`: оставьте `merge`, если хотите добавить MiniMax вместе со встроенными моделями.

## Примечания

-   Ссылки на модели имеют вид `minimax/`.
-   Рекомендуемые идентификаторы моделей: `MiniMax-M2.5` и `MiniMax-M2.5-highspeed`.
-   API использования Coding Plan: `https://api.minimaxi.com/v1/api/openplatform/coding_plan/remains` (требуется ключ плана coding).
-   Обновите значения цен в `models.json`, если вам нужен точный учет затрат.
-   Реферальная ссылка на MiniMax Coding Plan (скидка 10%): [https://platform.minimax.io/subscribe/coding-plan?code=DbXJTRClnb&source=link](https://platform.minimax.io/subscribe/coding-plan?code=DbXJTRClnb&source=link)
-   См. [/concepts/model-providers](../concepts/model-providers.md) для правил провайдеров.
-   Используйте `openclaw models list` и `openclaw models set minimax/MiniMax-M2.5` для переключения.

## Устранение неполадок

### «Unknown model: minimax/MiniMax-M2.5»

Обычно это означает, что **провайдер MiniMax не настроен** (нет записи провайдера и не найден профиль аутентификации MiniMax или ключ окружения). Исправление для этого обнаружения находится в **2026.1.12** (на момент написания еще не выпущено). Исправьте путем:

-   Обновления до **2026.1.12** (или запуска из исходного кода `main`), затем перезапустите шлюз.
-   Выполнения `openclaw configure` и выбора **MiniMax M2.5**, или
-   Добавления блока `models.providers.minimax` вручную, или
-   Установки `MINIMAX_API_KEY` (или профиля аутентификации MiniMax), чтобы провайдер мог быть внедрен.

Убедитесь, что идентификатор модели **чувствителен к регистру**:

-   `minimax/MiniMax-M2.5`
-   `minimax/MiniMax-M2.5-highspeed`
-   `minimax/MiniMax-M2.5-Lightning` (устаревший)

Затем проверьте снова:

```bash
openclaw models list
```

[GLM Models](./glm.md)[Moonshot AI](./moonshot.md)