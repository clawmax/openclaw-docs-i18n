

  Платформы для обмена сообщениями

  
# Feishu

Feishu (Lark) — это платформа для командного чата, используемая компаниями для обмена сообщениями и совместной работы. Этот плагин подключает OpenClaw к боту Feishu/Lark с использованием подписки на события WebSocket платформы, что позволяет получать сообщения без раскрытия публичного URL вебхука.

* * *

## Требуемый плагин

Установите плагин Feishu:

```bash
openclaw plugins install @openclaw/feishu
```

Локальная установка (при запуске из git-репозитория):

```bash
openclaw plugins install ./extensions/feishu
```

* * *

## Быстрый старт

Есть два способа добавить канал Feishu:

### Способ 1: мастер настройки (рекомендуется)

Если вы только что установили OpenClaw, запустите мастер:

```bash
openclaw onboard
```

Мастер проведёт вас через:

1.  Создание приложения Feishu и сбор учётных данных
2.  Настройку учётных данных приложения в OpenClaw
3.  Запуск шлюза

✅ **После настройки** проверьте статус шлюза:

-   `openclaw gateway status`
-   `openclaw logs --follow`

### Способ 2: настройка через CLI

Если вы уже завершили первоначальную установку, добавьте канал через CLI:

```bash
openclaw channels add
```

Выберите **Feishu**, затем введите App ID и App Secret. ✅ **После настройки** управляйте шлюзом:

-   `openclaw gateway status`
-   `openclaw gateway restart`
-   `openclaw logs --follow`

* * *

## Шаг 1: Создание приложения Feishu

### 1. Откройте Feishu Open Platform

Перейдите на [Feishu Open Platform](https://open.feishu.cn/app) и войдите в систему. Тенанты Lark (глобальные) должны использовать [https://open.larksuite.com/app](https://open.larksuite.com/app) и установить `domain: "lark"` в конфигурации Feishu.

### 2. Создайте приложение

1.  Нажмите **Create enterprise app** (Создать корпоративное приложение)
2.  Заполните название и описание приложения
3.  Выберите иконку приложения

![Создание корпоративного приложения](../images/channels-feishu-step2-create-app.png.md)

### 3. Скопируйте учётные данные

В разделе **Credentials & Basic Info** (Учётные данные и основная информация) скопируйте:

-   **App ID** (формат: `cli_xxx`)
-   **App Secret**

❗ **Важно:** храните App Secret в тайне. ![Получение учётных данных](../images/channels-feishu-step3-credentials.png.md)

### 4. Настройте разрешения

В разделе **Permissions** (Разрешения) нажмите **Batch import** (Пакетный импорт) и вставьте:

```json
{
  "scopes": {
    "tenant": [
      "aily:file:read",
      "aily:file:write",
      "application:application.app_message_stats.overview:readonly",
      "application:application:self_manage",
      "application:bot.menu:write",
      "cardkit:card:read",
      "cardkit:card:write",
      "contact:user.employee_id:readonly",
      "corehr:file:download",
      "event:ip_list",
      "im:chat.access_event.bot_p2p_chat:read",
      "im:chat.members:bot_access",
      "im:message",
      "im:message.group_at_msg:readonly",
      "im:message.p2p_msg:readonly",
      "im:message:readonly",
      "im:message:send_as_bot",
      "im:resource"
    ],
    "user": ["aily:file:read", "aily:file:write", "im:chat.access_event.bot_p2p_chat:read"]
  }
}
```

![Настройка разрешений](../images/channels-feishu-step4-permissions.png.md)

### 5. Включите возможность "Бот"

В разделе **App Capability** (Возможности приложения) > **Bot** (Бот):

1.  Включите возможность "Бот"
2.  Установите имя бота

![Включение возможности "Бот"](../images/channels-feishu-step5-bot-capability.png.md)

### 6. Настройте подписку на события

⚠️ **Важно:** перед настройкой подписки на события убедитесь, что:

1.  Вы уже выполнили `openclaw channels add` для Feishu
2.  Шлюз запущен (`openclaw gateway status`)

В разделе **Event Subscription** (Подписка на события):

1.  Выберите **Use long connection to receive events** (Использовать длинное соединение для получения событий) (WebSocket)
2.  Добавьте событие: `im.message.receive_v1`

⚠️ Если шлюз не запущен, настройка длинного соединения может не сохраниться. ![Настройка подписки на события](../images/channels-feishu-step6-event-subscription.png.md)

### 7. Опубликуйте приложение

1.  Создайте версию в разделе **Version Management & Release** (Управление версиями и выпуск)
2.  Отправьте на проверку и опубликуйте
3.  Дождитесь одобрения администратора (корпоративные приложения обычно одобряются автоматически)

* * *

## Шаг 2: Настройка OpenClaw

### Настройка с помощью мастера (рекомендуется)

```bash
openclaw channels add
```

Выберите **Feishu** и вставьте ваш App ID и App Secret.

### Настройка через конфигурационный файл

Отредактируйте `~/.openclaw/openclaw.json`:

```json
{
  channels: {
    feishu: {
      enabled: true,
      dmPolicy: "pairing",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
          botName: "My AI assistant",
        },
      },
    },
  },
}
```

Если вы используете `connectionMode: "webhook"`, установите `verificationToken`. Сервер вебхука Feishu по умолчанию привязывается к `127.0.0.1`; устанавливайте `webhookHost` только если вам намеренно нужен другой адрес привязки.

#### Verification Token (режим вебхука)

При использовании режима вебхука установите `channels.feishu.verificationToken` в вашей конфигурации. Чтобы получить значение:

1.  В Feishu Open Platform откройте ваше приложение
2.  Перейдите в **Development** → **Events & Callbacks** (开发配置 → 事件与回调)
3.  Откройте вкладку **Encryption** (加密策略)
4.  Скопируйте **Verification Token**

![Расположение Verification Token](../images/channels-feishu-verification-token.png.md)

### Настройка через переменные окружения

```bash
export FEISHU_APP_ID="cli_xxx"
export FEISHU_APP_SECRET="xxx"
```

### Домен Lark (глобальный)

Если ваш тенант находится на Lark (международная версия), установите домен в `lark` (или полную строку домена). Вы можете установить его в `channels.feishu.domain` или для каждого аккаунта (`channels.feishu.accounts..domain`).

```json
{
  channels: {
    feishu: {
      domain: "lark",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
        },
      },
    },
  },
}
```

### Флаги оптимизации квот

Вы можете сократить использование API Feishu с помощью двух опциональных флагов:

-   `typingIndicator` (по умолчанию `true`): при `false` пропускает вызовы реакции "печатает".
-   `resolveSenderNames` (по умолчанию `true`): при `false` пропускает вызовы поиска профиля отправителя.

Установите их на верхнем уровне или для каждого аккаунта:

```json
{
  channels: {
    feishu: {
      typingIndicator: false,
      resolveSenderNames: false,
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
          typingIndicator: true,
          resolveSenderNames: false,
        },
      },
    },
  },
}
```

* * *

## Шаг 3: Запуск + тестирование

### 1. Запустите шлюз

```bash
openclaw gateway
```

### 2. Отправьте тестовое сообщение

В Feishu найдите своего бота и отправьте сообщение.

### 3. Подтвердите сопряжение

По умолчанию бот отвечает кодом сопряжения. Подтвердите его:

```bash
openclaw pairing approve feishu <CODE>
```

После подтверждения вы можете общаться в обычном режиме.

* * *

## Обзор

-   **Канал бота Feishu**: бот Feishu, управляемый шлюзом
-   **Детерминированная маршрутизация**: ответы всегда возвращаются в Feishu
-   **Изоляция сессий**: личные сообщения используют общую основную сессию; группы изолированы
-   **WebSocket-соединение**: длинное соединение через SDK Feishu, публичный URL не требуется

* * *

## Контроль доступа

### Личные сообщения

-   **По умолчанию**: `dmPolicy: "pairing"` (неизвестные пользователи получают код сопряжения)
-   **Подтвердить сопряжение**:
    
    Копировать
    
    ```bash
    openclaw pairing list feishu
    openclaw pairing approve feishu <CODE>
    ```
    
-   **Режим белого списка**: установите `channels.feishu.allowFrom` с разрешёнными Open ID

### Групповые чаты

**1. Политика групп** (`channels.feishu.groupPolicy`):

-   `"open"` = разрешить всем в группах (по умолчанию)
-   `"allowlist"` = разрешать только `groupAllowFrom`
-   `"disabled"` = отключить групповые сообщения

**2. Требование упоминания** (`channels.feishu.groups.<chat_id>.requireMention`):

-   `true` = требовать упоминание @ (по умолчанию)
-   `false` = отвечать без упоминаний

* * *

## Примеры конфигурации групп

### Разрешить все группы, требовать упоминание @ (по умолчанию)

```json
{
  channels: {
    feishu: {
      groupPolicy: "open",
      // По умолчанию requireMention: true
    },
  },
}
```

### Разрешить все группы, упоминание @ не требуется

```json
{
  channels: {
    feishu: {
      groups: {
        oc_xxx: { requireMention: false },
      },
    },
  },
}
```

### Разрешить только определённые группы

```json
{
  channels: {
    feishu: {
      groupPolicy: "allowlist",
      // ID групп Feishu (chat_id) выглядят так: oc_xxx
      groupAllowFrom: ["oc_xxx", "oc_yyy"],
    },
  },
}
```

### Ограничить, какие отправители могут писать в группе (белый список отправителей)

В дополнение к разрешению самой группы, **все сообщения** в этой группе фильтруются по open_id отправителя: только пользователи, перечисленные в `groups.<chat_id>.allowFrom`, имеют свои сообщения обработанными; сообщения от других участников игнорируются (это полное ограничение на уровне отправителя, а не только для управляющих команд, таких как /reset или /new).

```json
{
  channels: {
    feishu: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["oc_xxx"],
      groups: {
        oc_xxx: {
          // ID пользователей Feishu (open_id) выглядят так: ou_xxx
          allowFrom: ["ou_user1", "ou_user2"],
        },
      },
    },
  },
}
```

* * *

## Получение ID групп/пользователей

### ID групп (chat_id)

ID групп выглядят как `oc_xxx`. **Способ 1 (рекомендуется)**

1.  Запустите шлюз и упомяните бота @ в группе
2.  Выполните `openclaw logs --follow` и найдите `chat_id`

**Способ 2** Используйте отладчик API Feishu для получения списка групповых чатов.

### ID пользователей (open_id)

ID пользователей выглядят как `ou_xxx`. **Способ 1 (рекомендуется)**

1.  Запустите шлюз и напишите боту в личные сообщения
2.  Выполните `openclaw logs --follow` и найдите `open_id`

**Способ 2** Проверьте запросы на сопряжение для Open ID пользователей:

```bash
openclaw pairing list feishu
```

* * *

## Общие команды

| Команда | Описание |
| --- | --- |
| `/status` | Показать статус бота |
| `/reset` | Сбросить сессию |
| `/model` | Показать/сменить модель |

> Примечание: Feishu пока не поддерживает нативные меню команд, поэтому команды должны отправляться как текст.

## Команды управления шлюзом

| Команда | Описание |
| --- | --- |
| `openclaw gateway status` | Показать статус шлюза |
| `openclaw gateway install` | Установить/запустить службу шлюза |
| `openclaw gateway stop` | Остановить службу шлюза |
| `openclaw gateway restart` | Перезапустить службу шлюза |
| `openclaw logs --follow` | Выводить логи шлюза в реальном времени |

* * *

## Устранение неполадок

### Бот не отвечает в групповых чатах

1.  Убедитесь, что бот добавлен в группу
2.  Убедитесь, что вы упомянули бота @ (поведение по умолчанию)
3.  Проверьте, что `groupPolicy` не установлен в `"disabled"`
4.  Проверьте логи: `openclaw logs --follow`

### Бот не получает сообщения

1.  Убедитесь, что приложение опубликовано и одобрено
2.  Убедитесь, что подписка на события включает `im.message.receive_v1`
3.  Убедитесь, что **длинное соединение** включено
4.  Убедитесь, что разрешения приложения полные
5.  Убедитесь, что шлюз запущен: `openclaw gateway status`
6.  Проверьте логи: `openclaw logs --follow`

### Утечка App Secret

1.  Сбросьте App Secret в Feishu Open Platform
2.  Обновите App Secret в вашей конфигурации
3.  Перезапустите шлюз

### Ошибки отправки сообщений

1.  Убедитесь, что у приложения есть разрешение `im:message:send_as_bot`
2.  Убедитесь, что приложение опубликовано
3.  Проверьте логи для подробных ошибок

* * *

## Расширенная конфигурация

### Несколько аккаунтов

```json
{
  channels: {
    feishu: {
      defaultAccount: "main",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
          botName: "Primary bot",
        },
        backup: {
          appId: "cli_yyy",
          appSecret: "yyy",
          botName: "Backup bot",
          enabled: false,
        },
      },
    },
  },
}
```

`defaultAccount` определяет, какой аккаунт Feishu используется, когда исходящие API явно не указывают `accountId`.

### Ограничения сообщений

-   `textChunkLimit`: размер фрагмента исходящего текста (по умолчанию: 2000 символов)
-   `mediaMaxMb`: ограничение на загрузку/скачивание медиа (по умолчанию: 30 МБ)

### Потоковая передача

Feishu поддерживает потоковые ответы через интерактивные карточки. При включении бот обновляет карточку по мере генерации текста.

```json
{
  channels: {
    feishu: {
      streaming: true, // включить потоковый вывод карточек (по умолчанию true)
      blockStreaming: true, // включить потоковую передачу на уровне блоков (по умолчанию true)
    },
  },
}
```

Установите `streaming: false`, чтобы дождаться полного ответа перед отправкой.

### Маршрутизация между несколькими агентами

Используйте `bindings` для маршрутизации личных сообщений или групп Feishu к разным агентам.

```json
{
  agents: {
    list: [
      { id: "main" },
      {
        id: "clawd-fan",
        workspace: "/home/user/clawd-fan",
        agentDir: "/home/user/.openclaw/agents/clawd-fan/agent",
      },
      {
        id: "clawd-xi",
        workspace: "/home/user/clawd-xi",
        agentDir: "/home/user/.openclaw/agents/clawd-xi/agent",
      },
    ],
  },
  bindings: [
    {
      agentId: "main",
      match: {
        channel: "feishu",
        peer: { kind: "direct", id: "ou_xxx" },
      },
    },
    {
      agentId: "clawd-fan",
      match: {
        channel: "feishu",
        peer: { kind: "direct", id: "ou_yyy" },
      },
    },
    {
      agentId: "clawd-xi",
      match: {
        channel: "feishu",
        peer: { kind: "group", id: "oc_zzz" },
      },
    },
  ],
}
```

Поля маршрутизации:

-   `match.channel`: `"feishu"`
-   `match.peer.kind`: `"direct"` или `"group"`
-   `match.peer.id`: Open ID пользователя (`ou_xxx`) или ID группы (`oc_xxx`)

См. советы по поиску в разделе [Получение ID групп/пользователей](#get-groupuser-ids).

* * *

## Справочник по конфигурации

Полная конфигурация: [Конфигурация шлюза](../gateway/configuration.md) Ключевые параметры:

| Параметр | Описание | По умолчанию |
| --- | --- | --- |
| `channels.feishu.enabled` | Включить/отключить канал | `true` |
| `channels.feishu.domain` | Домен API (`feishu` или `lark`) | `feishu` |
| `channels.feishu.connectionMode` | Режим транспорта событий | `websocket` |
| `channels.feishu.defaultAccount` | ID аккаунта по умолчанию для исходящей маршрутизации | `default` |
| `channels.feishu.verificationToken` | Обязателен для режима вебхука | \- |
| `channels.feishu.webhookPath` | Путь маршрута вебхука | `/feishu/events` |
| `channels.feishu.webhookHost` | Хост привязки вебхука | `127.0.0.1` |
| `channels.feishu.webhookPort` | Порт привязки вебхука | `3000` |
| `channels.feishu.accounts..appId` | App ID | \- |
| `channels.feishu.accounts..appSecret` | App Secret | \- |
| `channels.feishu.accounts..domain` | Переопределение домена API для конкретного аккаунта | `feishu` |
| `channels.feishu.dmPolicy` | Политика личных сообщений | `pairing` |
| `channels.feishu.allowFrom` | Белый список для ЛС (список open_id) | \- |
| `channels.feishu.groupPolicy` | Политика групп | `open` |
| `channels.feishu.groupAllowFrom` | Белый список групп | \- |
| `channels.feishu.groups.<chat_id>.requireMention` | Требовать упоминание @ | `true` |
| `channels.feishu.groups.<chat_id>.enabled` | Включить группу | `true` |
| `channels.feishu.textChunkLimit` | Размер фрагмента сообщения | `2000` |
| `channels.feishu.mediaMaxMb` | Ограничение размера медиа | `30` |
| `channels.feishu.streaming` | Включить потоковый вывод карточек | `true` |
| `channels.feishu.blockStreaming` | Включить потоковую передачу блоков | `true` |

* * *

## Справочник по dmPolicy

| Значение | Поведение |
| --- | --- |
| `"pairing"` | **По умолчанию.** Неизвестные пользователи получают код сопряжения; необходимо подтверждение |
| `"allowlist"` | Только пользователи из `allowFrom` могут общаться |
| `"open"` | Разрешить всем пользователям (требуется `"*"` в allowFrom) |
| `"disabled"` | Отключить личные сообщения |

* * *

## Поддерживаемые типы сообщений

### Получение

-   ✅ Текст
-   ✅ Форматированный текст (пост)
-   ✅ Изображения
-   ✅ Файлы
-   ✅ Аудио
-   ✅ Видео
-   ✅ Стикеры

### Отправка

-   ✅ Текст
-   ✅ Изображения
-   ✅ Файлы
-   ✅ Аудио
-   ⚠️ Форматированный текст (частичная поддержка)

[Discord](./discord.md)[Google Chat](./googlechat.md)