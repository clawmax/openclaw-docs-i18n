title: "Библиотека схем TypeBox для протокола Gateway WebSocket"
description: "Узнайте, как TypeBox определяет протокол Gateway WebSocket для валидации во время выполнения, экспорта JSON Schema и генерации кода Swift. Следуйте пошаговому руководству, чтобы добавить новые методы."
keywords: ["typebox", "websocket протокол", "валидация схем", "typescript схема", "swift кодогенерация", "архитектура gateway", "json схема", "валидация во время выполнения"]
---

  Внутренние концепции

  
# TypeBox

Последнее обновление: 2026-01-10 TypeBox — это библиотека схем, ориентированная на TypeScript. Мы используем её для определения **протокола Gateway WebSocket** (рукопожатие, запрос/ответ, события сервера). Эти схемы служат основой для **валидации во время выполнения**, **экспорта JSON Schema** и **генерации кода Swift** для приложения macOS. Один источник истины; всё остальное генерируется. Если вам нужен контекст протокола более высокого уровня, начните с [Архитектуры Gateway](./architecture.md).

## Ментальная модель (30 секунд)

Каждое сообщение Gateway WS — это один из трёх фреймов:

-   **Запрос**: `{ type: "req", id, method, params }`
-   **Ответ**: `{ type: "res", id, ok, payload | error }`
-   **Событие**: `{ type: "event", event, payload, seq?, stateVersion? }`

Первый фрейм **должен** быть запросом `connect`. После этого клиенты могут вызывать методы (например, `health`, `send`, `chat.send`) и подписываться на события (например, `presence`, `tick`, `agent`). Поток соединения (минимальный):

```
Клиент                    Шлюз (Gateway)
  |---- req:connect -------->|
  |<---- res:hello-ok --------|
  |<---- event:tick ----------|
  |---- req:health ---------->|
  |<---- res:health ----------|
```

Распространённые методы + события:

| Категория | Примеры | Примечания |
| --- | --- | --- |
| Ядро | `connect`, `health`, `status` | `connect` должен быть первым |
| Обмен сообщениями | `send`, `poll`, `agent`, `agent.wait` | для побочных эффектов нужен `idempotencyKey` |
| Чат | `chat.history`, `chat.send`, `chat.abort`, `chat.inject` | Используются WebChat |
| Сессии | `sessions.list`, `sessions.patch`, `sessions.delete` | управление сессиями |
| Узлы | `node.list`, `node.invoke`, `node.pair.*` | Действия Gateway WS + узлов |
| События | `tick`, `presence`, `agent`, `chat`, `health`, `shutdown` | push-уведомления от сервера |

Авторитетный список находится в `src/gateway/server.ts` (`METHODS`, `EVENTS`).

## Где находятся схемы

-   Исходный код: `src/gateway/protocol/schema.ts`
-   Валидаторы времени выполнения (AJV): `src/gateway/protocol/index.ts`
-   Рукопожатие сервера + диспетчеризация методов: `src/gateway/server.ts`
-   Клиент Node: `src/gateway/client.ts`
-   Сгенерированная JSON Schema: `dist/protocol.schema.json`
-   Сгенерированные модели Swift: `apps/macos/Sources/OpenClawProtocol/GatewayModels.swift`

## Текущий пайплайн

-   `pnpm protocol:gen`
    -   записывает JSON Schema (черновик‑07) в `dist/protocol.schema.json`
-   `pnpm protocol:gen:swift`
    -   генерирует модели шлюза для Swift
-   `pnpm protocol:check`
    -   запускает оба генератора и проверяет, что вывод закоммичен

## Как схемы используются во время выполнения

-   **На стороне сервера**: каждый входящий фрейм валидируется с помощью AJV. Рукопожатие принимает только запрос `connect`, параметры которого соответствуют `ConnectParams`.
-   **На стороне клиента**: JS-клиент валидирует фреймы событий и ответов перед их использованием.
-   **Поверхность методов**: Шлюз объявляет поддерживаемые `methods` и `events` в `hello-ok`.

## Примеры фреймов

Connect (первое сообщение):

```json
{
  "type": "req",
  "id": "c1",
  "method": "connect",
  "params": {
    "minProtocol": 2,
    "maxProtocol": 2,
    "client": {
      "id": "openclaw-macos",
      "displayName": "macos",
      "version": "1.0.0",
      "platform": "macos 15.1",
      "mode": "ui",
      "instanceId": "A1B2"
    }
  }
}
```

Hello-ok ответ:

```json
{
  "type": "res",
  "id": "c1",
  "ok": true,
  "payload": {
    "type": "hello-ok",
    "protocol": 2,
    "server": { "version": "dev", "connId": "ws-1" },
    "features": { "methods": ["health"], "events": ["tick"] },
    "snapshot": {
      "presence": [],
      "health": {},
      "stateVersion": { "presence": 0, "health": 0 },
      "uptimeMs": 0
    },
    "policy": { "maxPayload": 1048576, "maxBufferedBytes": 1048576, "tickIntervalMs": 30000 }
  }
}
```

Запрос + ответ:

```json
{ "type": "req", "id": "r1", "method": "health" }
```

```json
{ "type": "res", "id": "r1", "ok": true, "payload": { "ok": true } }
```

Событие:

```json
{ "type": "event", "event": "tick", "payload": { "ts": 1730000000 }, "seq": 12 }
```

## Минимальный клиент (Node.js)

Минимальный полезный поток: connect + health.

```typescript
import { WebSocket } from "ws";

const ws = new WebSocket("ws://127.0.0.1:18789");

ws.on("open", () => {
  ws.send(
    JSON.stringify({
      type: "req",
      id: "c1",
      method: "connect",
      params: {
        minProtocol: 3,
        maxProtocol: 3,
        client: {
          id: "cli",
          displayName: "example",
          version: "dev",
          platform: "node",
          mode: "cli",
        },
      },
    }),
  );
});

ws.on("message", (data) => {
  const msg = JSON.parse(String(data));
  if (msg.type === "res" && msg.id === "c1" && msg.ok) {
    ws.send(JSON.stringify({ type: "req", id: "h1", method: "health" }));
  }
  if (msg.type === "res" && msg.id === "h1") {
    console.log("health:", msg.payload);
    ws.close();
  }
});
```

## Разобранный пример: добавление метода от начала до конца

Пример: добавление нового запроса `system.echo`, который возвращает `{ ok: true, text }`.

1.  **Схема (источник истины)**

Добавить в `src/gateway/protocol/schema.ts`:

```bash
export const SystemEchoParamsSchema = Type.Object(
  { text: NonEmptyString },
  { additionalProperties: false },
);

export const SystemEchoResultSchema = Type.Object(
  { ok: Type.Boolean(), text: NonEmptyString },
  { additionalProperties: false },
);
```

Добавить обе в `ProtocolSchemas` и экспортировать типы:

```yaml
SystemEchoParams: SystemEchoParamsSchema,
  SystemEchoResult: SystemEchoResultSchema,
```

```bash
export type SystemEchoParams = Static<typeof SystemEchoParamsSchema>;
export type SystemEchoResult = Static<typeof SystemEchoResultSchema>;
```

2.  **Валидация**

В `src/gateway/protocol/index.ts` экспортировать валидатор AJV:

```bash
export const validateSystemEchoParams = ajv.compile<SystemEchoParams>(SystemEchoParamsSchema);
```

3.  **Поведение сервера**

Добавить обработчик в `src/gateway/server-methods/system.ts`:

```bash
export const systemHandlers: GatewayRequestHandlers = {
  "system.echo": ({ params, respond }) => {
    const text = String(params.text ?? "");
    respond(true, { ok: true, text });
  },
};
```

Зарегистрировать его в `src/gateway/server-methods.ts` (уже объединяет `systemHandlers`), затем добавить `"system.echo"` в `METHODS` в `src/gateway/server.ts`.

4.  **Перегенерировать**

```bash
pnpm protocol:check
```

5.  **Тесты + документация**

Добавить серверный тест в `src/gateway/server.*.test.ts` и отметить метод в документации.

## Поведение генератора кода Swift

Генератор Swift создаёт:

-   Перечисление `GatewayFrame` со случаями `req`, `res`, `event` и `unknown`
-   Строго типизированные структуры/перечисления для полезных данных
-   Значения `ErrorCode` и `GATEWAY_PROTOCOL_VERSION`

Неизвестные типы фреймов сохраняются как сырые полезные данные для обратной совместимости.

## Версионирование + совместимость

-   `PROTOCOL_VERSION` находится в `src/gateway/protocol/schema.ts`.
-   Клиенты отправляют `minProtocol` + `maxProtocol`; сервер отклоняет несоответствия.
-   Модели Swift сохраняют неизвестные типы фреймов, чтобы не ломать старых клиентов.

## Паттерны и соглашения схем

-   Большинство объектов используют `additionalProperties: false` для строгих полезных данных.
-   `NonEmptyString` используется по умолчанию для ID и имён методов/событий.
-   Фрейм верхнего уровня `GatewayFrame` использует **дискриминатор** по полю `type`.
-   Методы с побочными эффектами обычно требуют `idempotencyKey` в параметрах (пример: `send`, `poll`, `agent`, `chat.send`).
-   `agent` принимает опциональный `internalEvents` для контекста оркестрации, сгенерированного во время выполнения (например, передача завершения подзадачи/задачи cron); рассматривайте это как внутреннюю поверхность API.

## Живая JSON Schema

Сгенерированная JSON Schema находится в репозитории по пути `dist/protocol.schema.json`. Опубликованный сырой файл обычно доступен по адресу:

-   [https://raw.githubusercontent.com/openclaw/openclaw/main/dist/protocol.schema.json](https://raw.githubusercontent.com/openclaw/openclaw/main/dist/protocol.schema.json)

## Когда вы меняете схемы

1.  Обновите схемы TypeBox.
2.  Запустите `pnpm protocol:check`.
3.  Закоммитьте перегенерированную схему + модели Swift.

[Дата и время](../date-time.md)[Форматирование Markdown](./markdown-formatting.md)