title: "Документация по протоколу WebSocket API OpenClaw Gateway"
description: "Изучите протокол WebSocket OpenClaw Gateway для подключения клиентов, процесса рукопожатия, ролей, областей видимости, аутентификации и структуры API."
keywords: ["openclaw gateway", "websocket протокол", "документация api", "рукопожатие клиента", "аутентификация устройства", "возможности узла", "области видимости оператора", "транспорт шлюза"]
---

  Протоколы и API

  
# Протокол Gateway

Протокол Gateway WS — это **единая плоскость управления + транспорт для узлов** в OpenClaw. Все клиенты (CLI, веб-интерфейс, приложение macOS, узлы iOS/Android, автономные узлы) подключаются через WebSocket и объявляют свою **роль** + **область видимости** во время рукопожатия.

## Транспорт

-   WebSocket, текстовые фреймы с полезной нагрузкой JSON.
-   Первый фрейм **обязательно** должен быть запросом `connect`.

## Рукопожатие (connect)

Шлюз → Клиент (запрос перед подключением):

```json
{
  "type": "event",
  "event": "connect.challenge",
  "payload": { "nonce": "…", "ts": 1737264000000 }
}
```

Клиент → Шлюз:

```json
{
  "type": "req",
  "id": "…",
  "method": "connect",
  "params": {
    "minProtocol": 3,
    "maxProtocol": 3,
    "client": {
      "id": "cli",
      "version": "1.2.3",
      "platform": "macos",
      "mode": "operator"
    },
    "role": "operator",
    "scopes": ["operator.read", "operator.write"],
    "caps": [],
    "commands": [],
    "permissions": {},
    "auth": { "token": "…" },
    "locale": "en-US",
    "userAgent": "openclaw-cli/1.2.3",
    "device": {
      "id": "device_fingerprint",
      "publicKey": "…",
      "signature": "…",
      "signedAt": 1737264000000,
      "nonce": "…"
    }
  }
}
```

Шлюз → Клиент:

```json
{
  "type": "res",
  "id": "…",
  "ok": true,
  "payload": { "type": "hello-ok", "protocol": 3, "policy": { "tickIntervalMs": 15000 } }
}
```

Когда токен устройства выдан, `hello-ok` также включает:

```json
{
  "auth": {
    "deviceToken": "…",
    "role": "operator",
    "scopes": ["operator.read", "operator.write"]
  }
}
```

### Пример узла

```json
{
  "type": "req",
  "id": "…",
  "method": "connect",
  "params": {
    "minProtocol": 3,
    "maxProtocol": 3,
    "client": {
      "id": "ios-node",
      "version": "1.2.3",
      "platform": "ios",
      "mode": "node"
    },
    "role": "node",
    "scopes": [],
    "caps": ["camera", "canvas", "screen", "location", "voice"],
    "commands": ["camera.snap", "canvas.navigate", "screen.record", "location.get"],
    "permissions": { "camera.capture": true, "screen.record": false },
    "auth": { "token": "…" },
    "locale": "en-US",
    "userAgent": "openclaw-ios/1.2.3",
    "device": {
      "id": "device_fingerprint",
      "publicKey": "…",
      "signature": "…",
      "signedAt": 1737264000000,
      "nonce": "…"
    }
  }
}
```

## Структура фреймов

-   **Запрос**: `{type:"req", id, method, params}`
-   **Ответ**: `{type:"res", id, ok, payload|error}`
-   **Событие**: `{type:"event", event, payload, seq?, stateVersion?}`

Методы с побочными эффектами требуют **ключей идемпотентности** (см. схему).

## Роли + области видимости

### Роли

-   `operator` = клиент плоскости управления (CLI/UI/автоматизация).
-   `node` = хост возможностей (камера/экран/холст/system.run).

### Области видимости (оператор)

Распространённые области:

-   `operator.read`
-   `operator.write`
-   `operator.admin`
-   `operator.approvals`
-   `operator.pairing`

Область видимости метода — это только первый барьер. Некоторые команды слэша, доступные через `chat.send`, применяют более строгие проверки на уровне команд поверх этого. Например, постоянные операции записи `/config set` и `/config unset` требуют `operator.admin`.

### Возможности/команды/разрешения (узел)

Узлы объявляют заявки на возможности во время подключения:

-   `caps`: категории возможностей высокого уровня.
-   `commands`: белый список команд для вызова.
-   `permissions`: детальные переключатели (например, `screen.record`, `camera.capture`).

Шлюз рассматривает их как **заявки** и применяет белые списки на стороне сервера.

## Присутствие

-   `system-presence` возвращает записи, ключом которых является идентификатор устройства.
-   Записи о присутствии включают `deviceId`, `roles` и `scopes`, чтобы интерфейсы могли показывать одну строку на устройство, даже если оно подключено и как **оператор**, и как **узел**.

### Вспомогательные методы для узлов

-   Узлы могут вызывать `skills.bins` для получения текущего списка исполняемых файлов навыков для автоматических проверок разрешений.

### Вспомогательные методы для операторов

-   Операторы могут вызывать `tools.catalog` (`operator.read`) для получения каталога инструментов времени выполнения для агента. Ответ включает сгруппированные инструменты и метаданные происхождения:
    -   `source`: `core` или `plugin`
    -   `pluginId`: владелец плагина, когда `source="plugin"`
    -   `optional`: является ли инструмент плагина опциональным

## Одобрение выполнения

-   Когда запрос на выполнение требует одобрения, шлюз рассылает событие `exec.approval.requested`.
-   Клиенты-операторы разрешают его, вызывая `exec.approval.resolve` (требует область видимости `operator.approvals`).
-   Для `host=node` запрос `exec.approval.request` должен включать `systemRunPlan` (канонические `argv`/`cwd`/`rawCommand`/метаданные сессии). Запросы без `systemRunPlan` отклоняются.

## Версионирование

-   `PROTOCOL_VERSION` находится в `src/gateway/protocol/schema.ts`.
-   Клиенты отправляют `minProtocol` + `maxProtocol`; сервер отклоняет несоответствия.
-   Схемы + модели генерируются из определений TypeBox:
    -   `pnpm protocol:gen`
    -   `pnpm protocol:gen:swift`
    -   `pnpm protocol:check`

## Аутентификация

-   Если установлена переменная `OPENCLAW_GATEWAY_TOKEN` (или `--token`), то `connect.params.auth.token` должен совпадать, иначе соединение закрывается.
-   После сопряжения Шлюз выдаёт **токен устройства**, ограниченный ролью подключения + областями видимости. Он возвращается в `hello-ok.auth.deviceToken` и должен сохраняться клиентом для будущих подключений.
-   Токены устройств можно обновить/отозвать с помощью `device.token.rotate` и `device.token.revoke` (требует область видимости `operator.pairing`).

## Идентификация устройства + сопряжение

-   Узлы должны включать стабильный идентификатор устройства (`device.id`), полученный из отпечатка ключевой пары.
-   Шлюзы выдают токены для каждого устройства + роли.
-   Для новых идентификаторов устройств требуется одобрение сопряжения, если не включено локальное автоматическое одобрение.
-   **Локальные** подключения включают петлевой интерфейс и адрес tailnet самого хоста шлюза (чтобы привязки tailnet на том же хосте всё ещё могли автоматически одобряться).
-   Все WS-клиенты должны включать идентификатор `device` во время `connect` (оператор + узел). Панель управления может опускать его **только** когда включено `gateway.controlUi.dangerouslyDisableDeviceAuth` для аварийного использования.
-   Все подключения должны подписывать одноразовый номер `connect.challenge`, предоставленный сервером.

### Диагностика миграции аутентификации устройства

Для устаревших клиентов, которые всё ещё используют поведение подписи до запроса, `connect` теперь возвращает коды деталей `DEVICE_AUTH_*` под `error.details.code` со стабильным `error.details.reason`. Распространённые ошибки миграции:

| Сообщение | details.code | details.reason | Значение |
| --- | --- | --- | --- |
| `device nonce required` | `DEVICE_AUTH_NONCE_REQUIRED` | `device-nonce-missing` | Клиент опустил `device.nonce` (или отправил пустое значение). |
| `device nonce mismatch` | `DEVICE_AUTH_NONCE_MISMATCH` | `device-nonce-mismatch` | Клиент подписал устаревшим/неправильным одноразовым номером. |
| `device signature invalid` | `DEVICE_AUTH_SIGNATURE_INVALID` | `device-signature` | Полезная нагрузка подписи не соответствует полезной нагрузке v2. |
| `device signature expired` | `DEVICE_AUTH_SIGNATURE_EXPIRED` | `device-signature-stale` | Временная метка подписи выходит за допустимые пределы. |
| `device identity mismatch` | `DEVICE_AUTH_DEVICE_ID_MISMATCH` | `device-id-mismatch` | `device.id` не соответствует отпечатку открытого ключа. |
| `device public key invalid` | `DEVICE_AUTH_PUBLIC_KEY_INVALID` | `device-public-key` | Не удалось проверить формат/канонизацию открытого ключа. |

Цель миграции:

-   Всегда дожидайтесь `connect.challenge`.
-   Подпишите полезную нагрузку v2, которая включает серверный одноразовый номер.
-   Отправьте тот же одноразовый номер в `connect.params.device.nonce`.
-   Предпочтительная полезная нагрузка для подписи — `v3`, которая связывает `platform` и `deviceFamily` в дополнение к полям устройства/клиента/роли/областей видимости/токена/одноразового номера.
-   Устаревшие подписи `v2` по-прежнему принимаются для совместимости, но привязка метаданных сопряжённого устройства по-прежнему контролирует политику команд при повторном подключении.

## TLS + привязка сертификата

-   TLS поддерживается для WS-соединений.
-   Клиенты могут опционально привязывать отпечаток сертификата шлюза (см. конфигурацию `gateway.tls` плюс `gateway.remote.tlsFingerprint` или CLI `--tls-fingerprint`).

## Область видимости

Этот протокол предоставляет **полный API шлюза** (статус, каналы, модели, чат, агент, сессии, узлы, одобрения и т.д.). Точная поверхность определена схемами TypeBox в `src/gateway/protocol/schema.ts`.

[Песочница vs Политика инструментов vs Повышенные права](./sandbox-vs-tool-policy-vs-elevated.md)[Протокол Bridge](./bridge-protocol.md)