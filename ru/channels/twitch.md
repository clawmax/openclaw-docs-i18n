

  Платформы обмена сообщениями

  
# Twitch

Поддержка чата Twitch через IRC-подключение. OpenClaw подключается как пользователь Twitch (аккаунт бота) для получения и отправки сообщений в каналах.

## Требуется плагин

Twitch поставляется в виде плагина и не входит в базовую установку. Установите через CLI (npm registry):

```bash
openclaw plugins install @openclaw/twitch
```

Локальная установка (при запуске из git-репозитория):

```bash
openclaw plugins install ./extensions/twitch
```

Подробнее: [Плагины](../tools/plugin.md)

## Быстрая настройка (для начинающих)

1.  Создайте отдельный аккаунт Twitch для бота (или используйте существующий).
2.  Сгенерируйте учетные данные: [Twitch Token Generator](https://twitchtokengenerator.com/)
    -   Выберите **Bot Token**
    -   Убедитесь, что выбраны области видимости `chat:read` и `chat:write`
    -   Скопируйте **Client ID** и **Access Token**
3.  Найдите свой Twitch user ID: [https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/](https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/)
4.  Настройте токен:
    -   Переменная окружения: `OPENCLAW_TWITCH_ACCESS_TOKEN=...` (только для аккаунта по умолчанию)
    -   Или конфиг: `channels.twitch.accessToken`
    -   Если заданы оба, приоритет имеет конфиг (переменная окружения используется как запасной вариант только для аккаунта по умолчанию).
5.  Запустите шлюз.

**⚠️ Важно:** Добавьте контроль доступа (`allowFrom` или `allowedRoles`), чтобы предотвратить срабатывание бота неавторизованными пользователями. `requireMention` по умолчанию `true`. Минимальная конфигурация:

```json
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw", // Аккаунт бота в Twitch
      accessToken: "oauth:abc123...", // OAuth Access Token (или используйте переменную окружения OPENCLAW_TWITCH_ACCESS_TOKEN)
      clientId: "xyz789...", // Client ID из Token Generator
      channel: "vevisk", // В какой чат Twitch канала присоединиться (обязательно)
      allowFrom: ["123456789"], // (рекомендуется) Только ваш Twitch user ID - получите его на https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/
    },
  },
}
```

## Что это такое

-   Канал Twitch, принадлежащий Шлюзу.
-   Детерминированная маршрутизация: ответы всегда отправляются обратно в Twitch.
-   Каждый аккаунт сопоставляется с изолированным ключом сессии `agent::twitch:`.
-   `username` — это аккаунт бота (кто аутентифицируется), `channel` — это чат какого канала нужно присоединиться.

## Настройка (подробно)

### Генерация учетных данных

Используйте [Twitch Token Generator](https://twitchtokengenerator.com/):

-   Выберите **Bot Token**
-   Убедитесь, что выбраны области видимости `chat:read` и `chat:write`
-   Скопируйте **Client ID** и **Access Token**

Ручная регистрация приложения не требуется. Токены истекают через несколько часов.

### Настройка бота

**Переменная окружения (только для аккаунта по умолчанию):**

```
OPENCLAW_TWITCH_ACCESS_TOKEN=oauth:abc123...
```

**Или конфиг:**

```json
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",
      accessToken: "oauth:abc123...",
      clientId: "xyz789...",
      channel: "vevisk",
    },
  },
}
```

Если заданы и переменная окружения, и конфиг, приоритет имеет конфиг.

### Контроль доступа (рекомендуется)

```json
{
  channels: {
    twitch: {
      allowFrom: ["123456789"], // (рекомендуется) Только ваш Twitch user ID
    },
  },
}
```

Предпочтительнее использовать `allowFrom` для жесткого белого списка. Используйте `allowedRoles`, если нужен контроль доступа на основе ролей. **Доступные роли:** `"moderator"`, `"owner"`, `"vip"`, `"subscriber"`, `"all"`. **Зачем нужны user ID?** Имена пользователей могут меняться, что позволяет выдавать себя за другого. User ID постоянны. Найдите свой Twitch user ID: [https://www.streamweasels.com/tools/convert-twitch-username-%20to-user-id/](https://www.streamweasels.com/tools/convert-twitch-username-%20to-user-id/) (Конвертируйте ваше имя пользователя Twitch в ID)

## Обновление токена (опционально)

Токены из [Twitch Token Generator](https://twitchtokengenerator.com/) не могут быть обновлены автоматически — генерируйте заново по истечении срока действия. Для автоматического обновления токена создайте собственное приложение Twitch в [Twitch Developer Console](https://dev.twitch.tv/console) и добавьте в конфиг:

```json
{
  channels: {
    twitch: {
      clientSecret: "your_client_secret",
      refreshToken: "your_refresh_token",
    },
  },
}
```

Бот автоматически обновляет токены до истечения срока их действия и логирует события обновления.

## Поддержка нескольких аккаунтов

Используйте `channels.twitch.accounts` с токенами для каждого аккаунта. См. общий шаблон в [`gateway/configuration`](../gateway/configuration.md). Пример (один аккаунт бота в двух каналах):

```json
{
  channels: {
    twitch: {
      accounts: {
        channel1: {
          username: "openclaw",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "vevisk",
        },
        channel2: {
          username: "openclaw",
          accessToken: "oauth:def456...",
          clientId: "uvw012...",
          channel: "secondchannel",
        },
      },
    },
  },
}
```

**Примечание:** Каждому аккаунту нужен свой токен (один токен на канал).

## Контроль доступа

### Ограничения на основе ролей

```json
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowedRoles: ["moderator", "vip"],
        },
      },
    },
  },
}
```

### Белый список по User ID (наиболее безопасно)

```json
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowFrom: ["123456789", "987654321"],
        },
      },
    },
  },
}
```

### Доступ на основе ролей (альтернатива)

`allowFrom` — это жесткий белый список. Если он задан, разрешены только указанные user ID. Если вам нужен доступ на основе ролей, оставьте `allowFrom` незаданным и настройте вместо него `allowedRoles`:

```json
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowedRoles: ["moderator"],
        },
      },
    },
  },
}
```

### Отключение требования упоминания (@)

По умолчанию `requireMention` равно `true`. Чтобы отключить и отвечать на все сообщения:

```json
{
  channels: {
    twitch: {
      accounts: {
        default: {
          requireMention: false,
        },
      },
    },
  },
}
```

## Устранение неполадок

Сначала выполните диагностические команды:

```bash
openclaw doctor
openclaw channels status --probe
```

### Бот не отвечает на сообщения

**Проверьте контроль доступа:** Убедитесь, что ваш user ID есть в `allowFrom`, или временно удалите `allowFrom` и установите `allowedRoles: ["all"]` для тестирования. **Проверьте, что бот находится в канале:** Бот должен присоединиться к каналу, указанному в `channel`.

### Проблемы с токеном

**«Failed to connect» или ошибки аутентификации:**

-   Убедитесь, что `accessToken` — это значение OAuth access token (обычно начинается с префикса `oauth:`)
-   Проверьте, что токен имеет области видимости `chat:read` и `chat:write`
-   Если используется обновление токена, убедитесь, что заданы `clientSecret` и `refreshToken`

### Обновление токена не работает

**Проверьте логи на наличие событий обновления:**

```bash
Using env token source for mybot
Access token refreshed for user 123456 (expires in 14400s)
```

Если вы видите «token refresh disabled (no refresh token)»:

-   Убедитесь, что предоставлен `clientSecret`
-   Убедитесь, что предоставлен `refreshToken`

## Конфигурация

**Конфигурация аккаунта:**

-   `username` - Имя пользователя бота
-   `accessToken` - OAuth access token с областями `chat:read` и `chat:write`
-   `clientId` - Twitch Client ID (из Token Generator или вашего приложения)
-   `channel` - Канал для присоединения (обязательно)
-   `enabled` - Включить этот аккаунт (по умолчанию: `true`)
-   `clientSecret` - Опционально: Для автоматического обновления токена
-   `refreshToken` - Опционально: Для автоматического обновления токена
-   `expiresIn` - Срок действия токена в секундах
-   `obtainmentTimestamp` - Временная метка получения токена
-   `allowFrom` - Белый список User ID
-   `allowedRoles` - Контроль доступа на основе ролей (`"moderator" | "owner" | "vip" | "subscriber" | "all"`)
-   `requireMention` - Требовать упоминание @ (по умолчанию: `true`)

**Параметры провайдера:**

-   `channels.twitch.enabled` - Включить/отключить запуск канала
-   `channels.twitch.username` - Имя пользователя бота (упрощенная конфигурация для одного аккаунта)
-   `channels.twitch.accessToken` - OAuth access token (упрощенная конфигурация для одного аккаунта)
-   `channels.twitch.clientId` - Twitch Client ID (упрощенная конфигурация для одного аккаунта)
-   `channels.twitch.channel` - Канал для присоединения (упрощенная конфигурация для одного аккаунта)
-   `channels.twitch.accounts.` - Конфигурация для нескольких аккаунтов (все поля аккаунта выше)

Полный пример:

```json
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",
      accessToken: "oauth:abc123...",
      clientId: "xyz789...",
      channel: "vevisk",
      clientSecret: "secret123...",
      refreshToken: "refresh456...",
      allowFrom: ["123456789"],
      allowedRoles: ["moderator", "vip"],
      accounts: {
        default: {
          username: "mybot",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "your_channel",
          enabled: true,
          clientSecret: "secret123...",
          refreshToken: "refresh456...",
          expiresIn: 14400,
          obtainmentTimestamp: 1706092800000,
          allowFrom: ["123456789", "987654321"],
          allowedRoles: ["moderator"],
        },
      },
    },
  },
}
```

## Действия инструмента

Агент может вызывать `twitch` с действием:

-   `send` - Отправить сообщение в канал

Пример:

```json
{
  action: "twitch",
  params: {
    message: "Hello Twitch!",
    to: "#mychannel",
  },
}
```

## Безопасность и эксплуатация

-   **Относитесь к токенам как к паролям** — Никогда не коммитьте токены в git
-   **Используйте автоматическое обновление токена** для долгоживущих ботов
-   **Используйте белые списки user ID** вместо имен пользователей для контроля доступа
-   **Мониторьте логи** на предмет событий обновления токена и статуса подключения
-   **Ограничивайте области видимости токенов** — Запрашивайте только `chat:read` и `chat:write`
-   **Если зависло**: Перезапустите шлюз после подтверждения, что никакой другой процесс не владеет сессией

## Ограничения

-   **500 символов** на сообщение (автоматически разбивается по границам слов)
-   Markdown удаляется перед разбиением
-   Ограничение скорости отсутствует (используются встроенные ограничения Twitch)

[Tlon](./tlon.md)[WhatsApp](./whatsapp.md)