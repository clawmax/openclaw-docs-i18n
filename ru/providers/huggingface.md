

  Провайдеры

  
# Hugging Face (Inference)

[Hugging Face Inference Providers](https://huggingface.co/docs/inference-providers) предлагают OpenAI-совместимые чат-завершения через единый API маршрутизатора. Вы получаете доступ ко многим моделям (DeepSeek, Llama и другим) с одним токеном. OpenClaw использует **OpenAI-совместимую конечную точку** (только чат-завершения); для text-to-image, эмбеддингов или речи используйте [HF inference clients](https://huggingface.co/docs/api-inference/quicktour) напрямую.

-   Провайдер: `huggingface`
-   Аутентификация: `HUGGINGFACE_HUB_TOKEN` или `HF_TOKEN` (детализированный токен с разрешением **Make calls to Inference Providers**)
-   API: OpenAI-совместимый (`https://router.huggingface.co/v1`)
-   Тарификация: Единый HF токен; [цены](https://huggingface.co/docs/inference-providers/pricing) соответствуют тарифам провайдеров с бесплатным уровнем.

## Быстрый старт

1.  Создайте детализированный токен в [Hugging Face → Settings → Tokens](https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained) с разрешением **Make calls to Inference Providers**.
2.  Запустите онбординг и выберите **Hugging Face** в выпадающем списке провайдеров, затем введите свой API-ключ при запросе:

```bash
openclaw onboard --auth-choice huggingface-api-key
```

3.  В выпадающем списке **Default Hugging Face model** выберите желаемую модель (список загружается из Inference API при наличии валидного токена; в противном случае показывается встроенный список). Ваш выбор сохраняется как модель по умолчанию.
4.  Вы также можете установить или изменить модель по умолчанию позже в конфигурации:

```json
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1" },
    },
  },
}
```

## Неинтерактивный пример

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice huggingface-api-key \
  --huggingface-api-key "$HF_TOKEN"
```

Это установит `huggingface/deepseek-ai/DeepSeek-R1` в качестве модели по умолчанию.

## Примечание о среде

Если Шлюз работает как демон (launchd/systemd), убедитесь, что `HUGGINGFACE_HUB_TOKEN` или `HF_TOKEN` доступен этому процессу (например, в `~/.openclaw/.env` или через `env.shellEnv`).

## Обнаружение моделей и выпадающий список онбординга

OpenClaw обнаруживает модели, обращаясь **напрямую к конечной точке Inference**:

```bash
GET https://router.huggingface.co/v1/models
```

(Опционально: отправьте `Authorization: Bearer $HUGGINGFACE_HUB_TOKEN` или `$HF_TOKEN` для получения полного списка; некоторые конечные точки возвращают подмножество без аутентификации.) Ответ в стиле OpenAI: `{ "object": "list", "data": [ { "id": "Qwen/Qwen3-8B", "owned_by": "Qwen", ... }, ... ] }`. Когда вы настраиваете API-ключ Hugging Face (через онбординг, `HUGGINGFACE_HUB_TOKEN` или `HF_TOKEN`), OpenClaw использует этот GET-запрос для обнаружения доступных моделей для чат-завершений. Во время **интерактивного онбординга**, после ввода токена вы видите выпадающий список **Default Hugging Face model**, заполненный из этого списка (или из встроенного каталога, если запрос не удался). Во время выполнения (например, при запуске Шлюза), когда ключ присутствует, OpenClaw снова вызывает **GET** `https://router.huggingface.co/v1/models` для обновления каталога. Список объединяется со встроенным каталогом (для метаданных, таких как контекстное окно и стоимость). Если запрос не удался или ключ не установлен, используется только встроенный каталог.

## Имена моделей и редактируемые опции

-   **Имя из API:** Отображаемое имя модели **гидратируется из GET /v1/models**, когда API возвращает `name`, `title` или `display_name`; в противном случае оно выводится из идентификатора модели (например, `deepseek-ai/DeepSeek-R1` → “DeepSeek R1”).
-   **Переопределить отображаемое имя:** Вы можете установить пользовательскую метку для каждой модели в конфигурации, чтобы она отображалась так, как вы хотите, в CLI и UI:

```json
{
  agents: {
    defaults: {
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1 (быстрая)" },
        "huggingface/deepseek-ai/DeepSeek-R1:cheapest": { alias: "DeepSeek R1 (дешёвая)" },
      },
    },
  },
}
```

-   **Выбор провайдера / политики:** Добавьте суффикс к **идентификатору модели**, чтобы выбрать, как маршрутизатор выбирает бэкенд:
    
    -   **`:fastest`** — наивысшая пропускная способность (выбирает маршрутизатор; выбор провайдера **заблокирован** — интерактивный выбор бэкенда не показывается).
    -   **`:cheapest`** — самая низкая стоимость за выходной токен (выбирает маршрутизатор; выбор провайдера **заблокирован**).
    -   **`:provider`** — принудительно использовать конкретный бэкенд (например, `:sambanova`, `:together`).
    
    Когда вы выбираете **:cheapest** или **:fastest** (например, в выпадающем списке моделей при онбординге), провайдер блокируется: маршрутизатор решает по стоимости или скорости, и дополнительный шаг «предпочитать конкретный бэкенд» не показывается. Вы можете добавить их как отдельные записи в `models.providers.huggingface.models` или установить `model.primary` с суффиксом. Вы также можете установить порядок по умолчанию в [Inference Provider settings](https://hf.co/settings/inference-providers) (без суффикса = использовать этот порядок).
-   **Слияние конфигурации:** Существующие записи в `models.providers.huggingface.models` (например, в `models.json`) сохраняются при слиянии конфигураций. Таким образом, любые пользовательские `name`, `alias` или параметры модели, которые вы там установили, сохраняются.

## Идентификаторы моделей и примеры конфигурации

Ссылки на модели используют форму `huggingface//` (ID в стиле Hub). Список ниже взят из **GET** `https://router.huggingface.co/v1/models`; ваш каталог может включать больше. **Примеры ID (из конечной точки inference):**

| Модель | Ref (добавьте префикс `huggingface/`) |
| --- | --- |
| DeepSeek R1 | `deepseek-ai/DeepSeek-R1` |
| DeepSeek V3.2 | `deepseek-ai/DeepSeek-V3.2` |
| Qwen3 8B | `Qwen/Qwen3-8B` |
| Qwen2.5 7B Instruct | `Qwen/Qwen2.5-7B-Instruct` |
| Qwen3 32B | `Qwen/Qwen3-32B` |
| Llama 3.3 70B Instruct | `meta-llama/Llama-3.3-70B-Instruct` |
| Llama 3.1 8B Instruct | `meta-llama/Llama-3.1-8B-Instruct` |
| GPT-OSS 120B | `openai/gpt-oss-120b` |
| GLM 4.7 | `zai-org/GLM-4.7` |
| Kimi K2.5 | `moonshotai/Kimi-K2.5` |

Вы можете добавить `:fastest`, `:cheapest` или `:provider` (например, `:together`, `:sambanova`) к идентификатору модели. Установите порядок по умолчанию в [Inference Provider settings](https://hf.co/settings/inference-providers); см. [Inference Providers](https://huggingface.co/docs/inference-providers) и **GET** `https://router.huggingface.co/v1/models` для полного списка.

### Полные примеры конфигурации

**Основная DeepSeek R1 с резервной Qwen:**

```json
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-R1",
        fallbacks: ["huggingface/Qwen/Qwen3-8B"],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1" },
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
      },
    },
  },
}
```

**Qwen по умолчанию, с вариантами :cheapest и :fastest:**

```json
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen3-8B" },
      models: {
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
        "huggingface/Qwen/Qwen3-8B:cheapest": { alias: "Qwen3 8B (самая дешёвая)" },
        "huggingface/Qwen/Qwen3-8B:fastest": { alias: "Qwen3 8B (самая быстрая)" },
      },
    },
  },
}
```

**DeepSeek + Llama + GPT-OSS с псевдонимами:**

```json
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-V3.2",
        fallbacks: [
          "huggingface/meta-llama/Llama-3.3-70B-Instruct",
          "huggingface/openai/gpt-oss-120b",
        ],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-V3.2": { alias: "DeepSeek V3.2" },
        "huggingface/meta-llama/Llama-3.3-70B-Instruct": { alias: "Llama 3.3 70B" },
        "huggingface/openai/gpt-oss-120b": { alias: "GPT-OSS 120B" },
      },
    },
  },
}
```

**Принудительное использование конкретного бэкенда с :provider:**

```json
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1:together" },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1:together": { alias: "DeepSeek R1 (Together)" },
      },
    },
  },
}
```

**Несколько моделей Qwen и DeepSeek с суффиксами политик:**

```json
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest" },
      models: {
        "huggingface/Qwen/Qwen2.5-7B-Instruct": { alias: "Qwen2.5 7B" },
        "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest": { alias: "Qwen2.5 7B (дешёвая)" },
        "huggingface/deepseek-ai/DeepSeek-R1:fastest": { alias: "DeepSeek R1 (быстрая)" },
        "huggingface/meta-llama/Llama-3.1-8B-Instruct": { alias: "Llama 3.1 8B" },
      },
    },
  },
}
```

[GitHub Copilot](./github-copilot.md)[Kilocode](./kilocode.md)