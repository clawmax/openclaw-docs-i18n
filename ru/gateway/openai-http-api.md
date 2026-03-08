title: "HTTP API OpenAI для Chat Completions в OpenClaw Gateway"
description: "Узнайте, как включить и использовать совместимую с OpenAI конечную точку HTTP API для Chat Completions в OpenClaw Gateway, включая аутентификацию, безопасность и примеры."
keywords: ["api openai", "chat completions", "openclaw gateway", "http api", "ai агент", "совместимость с openai", "интеграция api", "bearer аутентификация"]
---

  Протоколы и API

  
# OpenAI Chat Completions

Шлюз OpenClaw может обслуживать небольшую совместимую с OpenAI конечную точку Chat Completions. Эта конечная точка **по умолчанию отключена**. Сначала включите её в конфигурации.

-   `POST /v1/chat/completions`
-   Тот же порт, что и у Шлюза (WS + HTTP мультиплексирование): `http://<gateway-host>:/v1/chat/completions`

Под капотом запросы выполняются как обычный запуск агента через Шлюз (тот же путь кода, что и `openclaw agent`), поэтому маршрутизация/разрешения/конфигурация соответствуют вашему Шлюзу.

## Аутентификация

Использует конфигурацию аутентификации Шлюза. Отправляйте bearer-токен:

-   `Authorization: Bearer `

Примечания:

-   Когда `gateway.auth.mode="token"`, используйте `gateway.auth.token` (или `OPENCLAW_GATEWAY_TOKEN`).
-   Когда `gateway.auth.mode="password"`, используйте `gateway.auth.password` (или `OPENCLAW_GATEWAY_PASSWORD`).
-   Если настроен `gateway.auth.rateLimit` и происходит слишком много неудачных попыток аутентификации, конечная точка вернёт `429` с заголовком `Retry-After`.

## Граница безопасности (важно)

Рассматривайте эту конечную точку как поверхность доступа с **полными правами оператора** для экземпляра шлюза.

-   HTTP bearer-аутентификация здесь — это не узкая модель области действия для конкретного пользователя.
-   Действительный токен/пароль Шлюза для этой конечной точки следует рассматривать как учётные данные владельца/оператора.
-   Запросы проходят по тому же пути агента плоскости управления, что и доверенные действия оператора.
-   Для этой конечной точки нет отдельной границы инструментов для не-владельца/пользователя; как только вызывающая сторона проходит аутентификацию Шлюза здесь, OpenClaw рассматривает эту сторону как доверенного оператора для данного шлюза.
-   Если политика целевого агента разрешает использование чувствительных инструментов, эта конечная точка может их использовать.
-   Держите эту конечную точку только на loopback/tailnet/частном входе; не выставляйте её напрямую в публичный интернет.

См. [Безопасность](./security.md) и [Удалённый доступ](./remote.md).

## Выбор агента

Пользовательские заголовки не требуются: идентификатор агента кодируется в поле `model` OpenAI:

-   `model: "openclaw:"` (пример: `"openclaw:main"`, `"openclaw:beta"`)
-   `model: "agent:"` (псевдоним)

Или укажите конкретного агента OpenClaw через заголовок:

-   `x-openclaw-agent-id: ` (по умолчанию: `main`)

Для продвинутых пользователей:

-   `x-openclaw-session-key: ` для полного контроля над маршрутизацией сессии.

## Включение конечной точки

Установите `gateway.http.endpoints.chatCompletions.enabled` в `true`:

```json
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: true },
      },
    },
  },
}
```

## Отключение конечной точки

Установите `gateway.http.endpoints.chatCompletions.enabled` в `false`:

```json
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: false },
      },
    },
  },
}
```

## Поведение сессии

По умолчанию конечная точка **не сохраняет состояние между запросами** (новый ключ сессии генерируется при каждом вызове). Если запрос включает строку `user` от OpenAI, Шлюз выводит стабильный ключ сессии из неё, чтобы повторные вызовы могли использовать одну сессию агента.

## Потоковая передача (SSE)

Установите `stream: true` для получения событий, отправляемых сервером (SSE):

-   `Content-Type: text/event-stream`
-   Каждая строка события имеет вид `data: `
-   Поток завершается строкой `data: [DONE]`

## Примеры

Без потоковой передачи:

```bash
curl -sS http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "messages": [{"role":"user","content":"hi"}]
  }'
```

С потоковой передачей:

```bash
curl -N http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "stream": true,
    "messages": [{"role":"user","content":"hi"}]
  }'
```

[Протокол Bridge](./bridge-protocol.md)[API OpenResponses](./openresponses-http-api.md)