

  Платформы обмена сообщениями

  
# LINE

LINE подключается к OpenClaw через LINE Messaging API. Плагин работает как получатель вебхуков на шлюзе и использует ваш токен доступа к каналу и секрет канала для аутентификации. Статус: поддерживается через плагин. Поддерживаются личные сообщения, групповые чаты, медиа, местоположения, Flex-сообщения, шаблонные сообщения и быстрые ответы. Реакции и ветки не поддерживаются.

## Требуется плагин

Установите плагин LINE:

```bash
openclaw plugins install @openclaw/line
```

Локальная установка (при запуске из git-репозитория):

```bash
openclaw plugins install ./extensions/line
```

## Настройка

1.  Создайте аккаунт LINE Developers и откройте Консоль: [https://developers.line.biz/console/](https://developers.line.biz/console/)
2.  Создайте (или выберите) Провайдера и добавьте канал **Messaging API**.
3.  Скопируйте **Channel access token** и **Channel secret** из настроек канала.
4.  Включите **Use webhook** в настройках Messaging API.
5.  Установите URL вебхука на конечную точку вашего шлюза (требуется HTTPS):

```
https://gateway-host/line/webhook
```

Шлюз отвечает на проверку вебхука LINE (GET) и входящие события (POST). Если вам нужен пользовательский путь, установите `channels.line.webhookPath` или `channels.line.accounts..webhookPath` и обновите URL соответственно. Примечание по безопасности:

-   Проверка подписи LINE зависит от тела запроса (HMAC над исходным телом), поэтому OpenClaw применяет строгие предварительные ограничения на размер тела и таймаут перед проверкой.

## Конфигурация

Минимальная конфигурация:

```json
{
  channels: {
    line: {
      enabled: true,
      channelAccessToken: "LINE_CHANNEL_ACCESS_TOKEN",
      channelSecret: "LINE_CHANNEL_SECRET",
      dmPolicy: "pairing",
    },
  },
}
```

Переменные окружения (только для аккаунта по умолчанию):

-   `LINE_CHANNEL_ACCESS_TOKEN`
-   `LINE_CHANNEL_SECRET`

Файлы с токеном/секретом:

```json
{
  channels: {
    line: {
      tokenFile: "/path/to/line-token.txt",
      secretFile: "/path/to/line-secret.txt",
    },
  },
}
```

Несколько аккаунтов:

```json
{
  channels: {
    line: {
      accounts: {
        marketing: {
          channelAccessToken: "...",
          channelSecret: "...",
          webhookPath: "/line/marketing",
        },
      },
    },
  },
}
```

## Контроль доступа

Личные сообщения по умолчанию используют режим сопряжения. Неизвестные отправители получают код сопряжения, и их сообщения игнорируются до подтверждения.

```bash
openclaw pairing list line
openclaw pairing approve line <CODE>
```

Белые списки и политики:

-   `channels.line.dmPolicy`: `pairing | allowlist | open | disabled`
-   `channels.line.allowFrom`: разрешённые ID пользователей LINE для личных сообщений
-   `channels.line.groupPolicy`: `allowlist | open | disabled`
-   `channels.line.groupAllowFrom`: разрешённые ID пользователей LINE для групп
-   Переопределения для конкретных групп: `channels.line.groups..allowFrom`
-   Примечание для времени выполнения: если `channels.line` полностью отсутствует, среда выполнения возвращается к `groupPolicy="allowlist"` для проверок групп (даже если установлен `channels.defaults.groupPolicy`).

ID LINE чувствительны к регистру. Валидные ID выглядят так:

-   Пользователь: `U` + 32 шестнадцатеричных символа
-   Группа: `C` + 32 шестнадцатеричных символа
-   Комната: `R` + 32 шестнадцатеричных символа

## Поведение сообщений

-   Текст разбивается на части по 5000 символов.
-   Форматирование Markdown удаляется; блоки кода и таблицы, по возможности, преобразуются в Flex-карточки.
-   Потоковые ответы буферизуются; LINE получает полные фрагменты с анимацией загрузки, пока агент работает.
-   Загрузка медиа ограничена параметром `channels.line.mediaMaxMb` (по умолчанию 10).

## Данные канала (расширенные сообщения)

Используйте `channelData.line` для отправки быстрых ответов, местоположений, Flex-карточек или шаблонных сообщений.

```json
{
  text: "Вот, пожалуйста",
  channelData: {
    line: {
      quickReplies: ["Статус", "Помощь"],
      location: {
        title: "Офис",
        address: "ул. Главная, 123",
        latitude: 35.681236,
        longitude: 139.767125,
      },
      flexMessage: {
        altText: "Карточка статуса",
        contents: {
          /* Полезная нагрузка Flex */
        },
      },
      templateMessage: {
        type: "confirm",
        text: "Продолжить?",
        confirmLabel: "Да",
        confirmData: "yes",
        cancelLabel: "Нет",
        cancelData: "no",
      },
    },
  },
}
```

Плагин LINE также включает команду `/card` для предустановленных Flex-сообщений:

```bash
/card info "Добро пожаловать" "Спасибо, что присоединились!"
```

## Устранение неполадок

-   **Не удаётся проверить вебхук:** убедитесь, что URL вебхука использует HTTPS и `channelSecret` совпадает с указанным в консоли LINE.
-   **Нет входящих событий:** проверьте, что путь вебхука совпадает с `channels.line.webhookPath` и что шлюз доступен из LINE.
-   **Ошибки загрузки медиа:** увеличьте значение `channels.line.mediaMaxMb`, если медиа превышает лимит по умолчанию.

[IRC](./irc.md)[Matrix](./matrix.md)