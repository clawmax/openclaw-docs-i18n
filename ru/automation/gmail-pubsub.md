title: "Автоматизация Gmail в OpenClaw через PubSub Push-уведомления"
description: "Узнайте, как настроить автоматизацию Gmail с помощью Google PubSub. Пересылайте письма в OpenClaw через вебхуки для обработки ИИ и доставки в чат."
keywords: ["автоматизация gmail", "pubsub", "вебхук openclaw", "наблюдение gmail", "автоматизация почты", "gogcli", "tailscale funnel", "push-уведомления"]
---

  Автоматизация

  
# Gmail PubSub

Цель: Наблюдение Gmail -> Pub/Sub push -> `gog gmail watch serve` -> вебхук OpenClaw.

## Предварительные требования

-   Установленный и авторизованный `gcloud` ([руководство по установке](https://docs.cloud.google.com/sdk/docs/install-sdk)).
-   Установленный и авторизованный для аккаунта Gmail `gog` (gogcli) ([gogcli.sh](https://gogcli.sh/)).
-   Включённые хуки OpenClaw (см. [Вебхуки](./webhook.md)).
-   Авторизованный `tailscale` ([tailscale.com](https://tailscale.com/)). Поддерживаемая конфигурация использует Tailscale Funnel для публичной HTTPS-конечной точки. Другие туннельные сервисы могут работать, но требуют самостоятельной настройки и не поддерживаются. В настоящее время мы поддерживаем только Tailscale.

Пример конфигурации хука (включите пресет Gmail):

```json
{
  hooks: {
    enabled: true,
    token: "OPENCLAW_HOOK_TOKEN",
    path: "/hooks",
    presets: ["gmail"],
  },
}
```

Чтобы доставить сводку Gmail в чат, переопределите пресет с помощью маппинга, который устанавливает `deliver` + опционально `channel`/`to`:

```json
{
  hooks: {
    enabled: true,
    token: "OPENCLAW_HOOK_TOKEN",
    presets: ["gmail"],
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate: "Новое письмо от {{messages[0].from}}\nТема: {{messages[0].subject}}\n{{messages[0].snippet}}\n{{messages[0].body}}",
        model: "openai/gpt-5.2-mini",
        deliver: true,
        channel: "last",
        // to: "+15551234567"
      },
    ],
  },
}
```

Если нужен фиксированный канал, установите `channel` + `to`. Иначе `channel: "last"` использует последний маршрут доставки (резервный — WhatsApp). Чтобы принудительно использовать более дешёвую модель для Gmail, установите `model` в маппинге (`provider/model` или алиас). Если вы задали `agents.defaults.models`, включите её туда. Чтобы задать модель по умолчанию и уровень мышления именно для хуков Gmail, добавьте `hooks.gmail.model` / `hooks.gmail.thinking` в свою конфигурацию:

```json
{
  hooks: {
    gmail: {
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      thinking: "off",
    },
  },
}
```

Примечания:

-   `model`/`thinking` для конкретного хука в маппинге всё равно переопределяют эти значения по умолчанию.
-   Порядок отката: `hooks.gmail.model` → `agents.defaults.model.fallbacks` → основная модель (авторизация/лимиты/таймауты).
-   Если задан `agents.defaults.models`, модель Gmail должна быть в списке разрешённых.
-   Содержимое хука Gmail по умолчанию оборачивается границами безопасности для внешнего контента. Чтобы отключить (опасно), установите `hooks.gmail.allowUnsafeExternalContent: true`.

Для дальнейшей кастомизации обработки полезной нагрузки добавьте `hooks.mappings` или JS/TS модуль преобразования в `~/.openclaw/hooks/transforms` (см. [Вебхуки](./webhook.md)).

## Мастер (рекомендуется)

Используйте помощник OpenClaw, чтобы связать всё вместе (устанавливает зависимости на macOS через brew):

```bash
openclaw webhooks gmail setup \
  --account openclaw@gmail.com
```

Значения по умолчанию:

-   Использует Tailscale Funnel для публичной push-конечной точки.
-   Записывает конфигурацию `hooks.gmail` для `openclaw webhooks gmail run`.
-   Включает пресет Gmail (`hooks.presets: ["gmail"]`).

Примечание о пути: когда `tailscale.mode` включён, OpenClaw автоматически устанавливает `hooks.gmail.serve.path` в `/` и сохраняет публичный путь в `hooks.gmail.tailscale.path` (по умолчанию `/gmail-pubsub`), потому что Tailscale удаляет установленный префикс пути перед проксированием. Если нужно, чтобы бэкенд получал путь с префиксом, установите `hooks.gmail.tailscale.target` (или `--tailscale-target`) на полный URL, например `http://127.0.0.1:8788/gmail-pubsub`, и сопоставьте `hooks.gmail.serve.path`. Нужна пользовательская конечная точка? Используйте `--push-endpoint ` или `--tailscale off`. Примечание о платформе: на macOS мастер устанавливает `gcloud`, `gogcli` и `tailscale` через Homebrew; на Linux установите их вручную заранее. Автозапуск шлюза (рекомендуется):

-   Когда `hooks.enabled=true` и `hooks.gmail.account` задан, Шлюз запускает `gog gmail watch serve` при загрузке и автоматически обновляет наблюдение.
-   Установите `OPENCLAW_SKIP_GMAIL_WATCHER=1`, чтобы отказаться (полезно, если вы запускаете демон самостоятельно).
-   Не запускайте ручной демон одновременно, иначе получите ошибку `listen tcp 127.0.0.1:8788: bind: address already in use`.

Ручной демон (запускает `gog gmail watch serve` + автообновление):

```bash
openclaw webhooks gmail run
```

## Разовая настройка

1.  Выберите проект GCP, **которому принадлежит OAuth-клиент**, используемый `gog`.

```bash
gcloud auth login
gcloud config set project <project-id>
```

Примечание: Наблюдение Gmail требует, чтобы тема Pub/Sub находилась в том же проекте, что и OAuth-клиент.

2.  Включите API:

```bash
gcloud services enable gmail.googleapis.com pubsub.googleapis.com
```

3.  Создайте тему:

```bash
gcloud pubsub topics create gog-gmail-watch
```

4.  Разрешите Gmail push публиковать:

```
gcloud pubsub topics add-iam-policy-binding gog-gmail-watch \
  --member=serviceAccount:gmail-api-push@system.gserviceaccount.com \
  --role=roles/pubsub.publisher
```

## Запуск наблюдения

```
gog gmail watch start \
  --account openclaw@gmail.com \
  --label INBOX \
  --topic projects/<project-id>/topics/gog-gmail-watch
```

Сохраните `history_id` из вывода (для отладки).

## Запуск обработчика push-уведомлений

Локальный пример (аутентификация общим токеном):

```
gog gmail watch serve \
  --account openclaw@gmail.com \
  --bind 127.0.0.1 \
  --port 8788 \
  --path /gmail-pubsub \
  --token <shared> \
  --hook-url http://127.0.0.1:18789/hooks/gmail \
  --hook-token OPENCLAW_HOOK_TOKEN \
  --include-body \
  --max-bytes 20000
```

Примечания:

-   `--token` защищает push-конечную точку (`x-gog-token` или `?token=`).
-   `--hook-url` указывает на OpenClaw `/hooks/gmail` (сопоставлен; изолированный запуск + сводка в основной).
-   `--include-body` и `--max-bytes` управляют фрагментом тела, отправляемым в OpenClaw.

Рекомендуется: `openclaw webhooks gmail run` оборачивает тот же процесс и автоматически обновляет наблюдение.

## Публикация обработчика (продвинутый, не поддерживается)

Если нужен туннель не на Tailscale, настройте его вручную и используйте публичный URL в push-подписке (не поддерживается, без защитных механизмов):

```bash
cloudflared tunnel --url http://127.0.0.1:8788 --no-autoupdate
```

Используйте сгенерированный URL как push-конечную точку:

```
gcloud pubsub subscriptions create gog-gmail-watch-push \
  --topic gog-gmail-watch \
  --push-endpoint "https://<public-url>/gmail-pubsub?token=<shared>"
```

Продакшен: используйте стабильную HTTPS-конечную точку и настройте OIDC JWT для Pub/Sub, затем запустите:

```bash
gog gmail watch serve --verify-oidc --oidc-email <svc@...>
```

## Тестирование

Отправьте сообщение в отслеживаемый ящик:

```
gog gmail send \
  --account openclaw@gmail.com \
  --to openclaw@gmail.com \
  --subject "watch test" \
  --body "ping"
```

Проверьте состояние наблюдения и историю:

```bash
gog gmail watch status --account openclaw@gmail.com
gog gmail history --account openclaw@gmail.com --since <historyId>
```

## Устранение неполадок

-   `Invalid topicName`: несоответствие проекта (тема не в проекте OAuth-клиента).
-   `User not authorized`: отсутствует `roles/pubsub.publisher` для темы.
-   Пустые сообщения: Gmail push предоставляет только `historyId`; получайте через `gog gmail history`.

## Очистка

```bash
gog gmail watch stop --account openclaw@gmail.com
gcloud pubsub subscriptions delete gog-gmail-watch-push
gcloud pubsub topics delete gog-gmail-watch
```

[Вебхуки](./webhook.md)[Опросы](./poll.md)