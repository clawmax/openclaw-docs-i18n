

  Платформы для обмена сообщениями

  
# Google Chat

Статус: готово для личных чатов и пространств через вебхуки Google Chat API (только HTTP).

## Быстрая настройка (для начинающих)

1.  Создайте проект в Google Cloud и включите **Google Chat API**.
    -   Перейдите по адресу: [Google Chat API Credentials](https://console.cloud.google.com/apis/api/chat.googleapis.com/credentials)
    -   Включите API, если он ещё не включен.
2.  Создайте **Сервисный аккаунт**:
    -   Нажмите **Create Credentials** > **Service Account**.
    -   Дайте ему любое имя (например, `openclaw-chat`).
    -   Оставьте разрешения пустыми (нажмите **Continue**).
    -   Оставьте список пользователей с доступом пустым (нажмите **Done**).
3.  Создайте и скачайте **JSON-ключ**:
    -   В списке сервисных аккаунтов нажмите на только что созданный.
    -   Перейдите на вкладку **Keys**.
    -   Нажмите **Add Key** > **Create new key**.
    -   Выберите **JSON** и нажмите **Create**.
4.  Сохраните скачанный JSON-файл на хосте шлюза (например, `~/.openclaw/googlechat-service-account.json`).
5.  Создайте приложение Google Chat в [Google Cloud Console Chat Configuration](https://console.cloud.google.com/apis/api/chat.googleapis.com/hangouts-chat):
    -   Заполните **Application info**:
        -   **App name**: (например, `OpenClaw`)
        -   **Avatar URL**: (например, `https://openclaw.ai/logo.png`)
        -   **Description**: (например, `Personal AI Assistant`)
    -   Включите **Interactive features**.
    -   В разделе **Functionality** отметьте **Join spaces and group conversations**.
    -   В разделе **Connection settings** выберите **HTTP endpoint URL**.
    -   В разделе **Triggers** выберите **Use a common HTTP endpoint URL for all triggers** и укажите публичный URL вашего шлюза с добавлением `/googlechat`.
        -   *Подсказка: выполните `openclaw status`, чтобы найти публичный URL вашего шлюза.*
    -   В разделе **Visibility** отметьте **Make this Chat app available to specific people and groups in `Your Domain`**.
    -   Введите свой адрес электронной почты (например, `user@example.com`) в текстовое поле.
    -   Нажмите **Save** внизу.
6.  **Включите статус приложения**:
    -   После сохнения **обновите страницу**.
    -   Найдите раздел **App status** (обычно вверху или внизу после сохранения).
    -   Измените статус на **Live - available to users**.
    -   Снова нажмите **Save**.
7.  Настройте OpenClaw с указанием пути к сервисному аккаунту и аудитории вебхука:
    -   Переменная окружения: `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE=/path/to/service-account.json`
    -   Или в конфиге: `channels.googlechat.serviceAccountFile: "/path/to/service-account.json"`.
8.  Установите тип аудитории вебхука + значение (должно соответствовать конфигурации вашего приложения Chat).
9.  Запустите шлюз. Google Chat будет отправлять POST-запросы на путь вашего вебхука.

## Добавление в Google Chat

После запуска шлюза и добавления вашего email в список видимости:

1.  Перейдите в [Google Chat](https://chat.google.com/).
2.  Нажмите значок **+** (плюс) рядом с разделом **Direct Messages**.
3.  В строке поиска (где обычно добавляют людей) введите **App name**, который вы указали в Google Cloud Console.
    -   **Примечание**: Бот *не* появится в списке обзора "Marketplace", так как это приватное приложение. Вы должны найти его по имени.
4.  Выберите своего бота из результатов.
5.  Нажмите **Add** или **Chat**, чтобы начать личный диалог.
6.  Отправьте "Hello", чтобы активировать ассистента!

## Публичный URL (только для вебхуков)

Вебхуки Google Chat требуют публичной HTTPS-конечной точки. В целях безопасности **открывайте для интернета только путь `/googlechat`**. Держите панель управления OpenClaw и другие чувствительные конечные точки в вашей частной сети.

### Вариант A: Tailscale Funnel (Рекомендуется)

Используйте Tailscale Serve для приватной панели управления и Funnel для публичного пути вебхука. Это сохраняет `/` приватным, открывая только `/googlechat`.

1.  **Проверьте, к какому адресу привязан ваш шлюз:**
    
    Копировать
    
    ```bash
    ss -tlnp | grep 18789
    ```
    
    Запомните IP-адрес (например, `127.0.0.1`, `0.0.0.0` или ваш Tailscale IP, например `100.x.x.x`).
2.  **Откройте панель управления только для tailnet (порт 8443):**
    
    Копировать
    
    ```bash
    # Если привязан к localhost (127.0.0.1 или 0.0.0.0):
    tailscale serve --bg --https 8443 http://127.0.0.1:18789
    
    # Если привязан только к Tailscale IP (например, 100.106.161.80):
    tailscale serve --bg --https 8443 http://100.106.161.80:18789
    ```
    
3.  **Откройте публично только путь вебхука:**
    
    Копировать
    
    ```bash
    # Если привязан к localhost (127.0.0.1 или 0.0.0.0):
    tailscale funnel --bg --set-path /googlechat http://127.0.0.1:18789/googlechat
    
    # Если привязан только к Tailscale IP (например, 100.106.161.80):
    tailscale funnel --bg --set-path /googlechat http://100.106.161.80:18789/googlechat
    ```
    
4.  **Авторизуйте узел для доступа к Funnel:** Если появится запрос, перейдите по URL авторизации, указанному в выводе, чтобы включить Funnel для этого узла в политике вашего tailnet.
5.  **Проверьте конфигурацию:**
    
    Копировать
    
    ```
    tailscale serve status
    tailscale funnel status
    ```
    

Ваш публичный URL вебхука будет: `https://<node-name>..ts.net/googlechat` Ваша приватная панель управления останется доступной только в tailnet: `https://<node-name>..ts.net:8443/` Используйте публичный URL (без `:8443`) в конфигурации приложения Google Chat.

> Примечание: Эта конфигурация сохраняется после перезагрузок. Чтобы удалить её позже, выполните `tailscale funnel reset` и `tailscale serve reset`.

### Вариант B: Обратный прокси (Caddy)

Если вы используете обратный прокси, например Caddy, проксируйте только конкретный путь:

```
your-domain.com {
    reverse_proxy /googlechat* localhost:18789
}
```

При такой конфигурации любой запрос к `your-domain.com/` будет проигнорирован или вернёт 404, а `your-domain.com/googlechat` будет безопасно перенаправлен в OpenClaw.

### Вариант C: Cloudflare Tunnel

Настройте правила входящего трафика вашего туннеля так, чтобы маршрутизировать только путь вебхука:

-   **Путь**: `/googlechat` -> `http://localhost:18789/googlechat`
-   **Правило по умолчанию**: HTTP 404 (Not Found)

## Как это работает

1.  Google Chat отправляет POST-запросы вебхуков на шлюз. Каждый запрос включает заголовок `Authorization: Bearer `.
    -   OpenClaw проверяет аутентификацию по Bearer токену перед чтением/разбором полных тел вебхуков, когда заголовок присутствует.
    -   Запросы Google Workspace Add-on, содержащие `authorizationEventObject.systemIdToken` в теле, поддерживаются через более строгий лимит на предварительную проверку тела.
2.  OpenClaw проверяет токен на соответствие настроенным `audienceType` + `audience`:
    -   `audienceType: "app-url"` → аудитория — это ваш HTTPS URL вебхука.
    -   `audienceType: "project-number"` → аудитория — это номер проекта Cloud.
3.  Сообщения маршрутизируются по пространствам:
    -   Личные сообщения используют ключ сессии `agent::googlechat:dm:`.
    -   Пространства используют ключ сессии `agent::googlechat:group:`.
4.  Доступ к личным сообщениям по умолчанию требует сопряжения. Неизвестным отправителям отправляется код сопряжения; подтвердите его командой:
    -   `openclaw pairing approve googlechat `
5.  Групповые пространства по умолчанию требуют упоминания через @. Используйте `botUser`, если для обнаружения упоминаний требуется имя пользователя приложения.

## Цели (Targets)

Используйте эти идентификаторы для доставки и белых списков:

-   Личные сообщения: `users/` (рекомендуется).
-   Сырой email `name@example.com` является изменяемым и используется только для прямого сопоставления с белым списком при `channels.googlechat.dangerouslyAllowNameMatching: true`.
-   Устаревший вариант: `users/` трактуется как идентификатор пользователя, а не как белый список email-адресов.
-   Пространства: `spaces/`.

## Основные моменты конфигурации

```json
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      // или serviceAccountRef: { source: "file", provider: "filemain", id: "/channels/googlechat/serviceAccount" }
      audienceType: "app-url",
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890", // опционально; помогает в обнаружении упоминаний
      dm: {
        policy: "pairing",
        allowFrom: ["users/1234567890"],
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": {
          allow: true,
          requireMention: true,
          users: ["users/1234567890"],
          systemPrompt: "Short answers only.",
        },
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```

Примечания:

-   Учётные данные сервисного аккаунта также можно передать встроенно с помощью `serviceAccount` (JSON-строка).
-   Также поддерживается `serviceAccountRef` (SecretRef из env/файла), включая ссылки на аккаунт в `channels.googlechat.accounts..serviceAccountRef`.
-   Путь вебхука по умолчанию — `/googlechat`, если `webhookPath` не задан.
-   `dangerouslyAllowNameMatching` повторно включает сопоставление изменяемых email-принципалов для белых списков (режим совместимости на случай аварии).
-   Реакции доступны через инструмент `reactions` и `channels action`, когда включено `actions.reactions`.
-   `typingIndicator` поддерживает значения `none`, `message` (по умолчанию) и `reaction` (для реакции требуется OAuth пользователя).
-   Вложения загружаются через Chat API и сохраняются в медиа-конвейере (размер ограничен `mediaMaxMb`).

Подробности о ссылках на секреты: [Управление секретами](../gateway/secrets.md).

## Устранение неполадок

### 405 Method Not Allowed

Если Google Cloud Logs Explorer показывает ошибки вида:

```bash
status code: 405, reason phrase: HTTP error response: HTTP/1.1 405 Method Not Allowed
```

Это означает, что обработчик вебхука не зарегистрирован. Распространённые причины:

1.  **Канал не настроен**: Раздел `channels.googlechat` отсутствует в вашей конфигурации. Проверьте:
    
    Копировать
    
    ```bash
    openclaw config get channels.googlechat
    ```
    
    Если возвращается "Config path not found", добавьте конфигурацию (см. [Основные моменты конфигурации](#config-highlights)).
2.  **Плагин не включён**: Проверьте статус плагина:
    
    Копировать
    
    ```bash
    openclaw plugins list | grep googlechat
    ```
    
    Если отображается "disabled", добавьте `plugins.entries.googlechat.enabled: true` в вашу конфигурацию.
3.  **Шлюз не перезапущен**: После добавления конфигурации перезапустите шлюз:
    
    Копировать
    
    ```bash
    openclaw gateway restart
    ```
    

Убедитесь, что канал работает:

```bash
openclaw channels status
# Должно показывать: Google Chat default: enabled, configured, ...
```

### Другие проблемы

-   Проверьте `openclaw channels status --probe` на наличие ошибок аутентификации или отсутствующей конфигурации аудитории.
-   Если сообщения не поступают, проверьте URL вебхука приложения Chat и подписки на события.
-   Если упоминание блокирует ответы, установите `botUser` в значение ресурса пользователя приложения и проверьте `requireMention`.
-   Используйте `openclaw logs --follow` при отправке тестового сообщения, чтобы увидеть, доходят ли запросы до шлюза.

Связанная документация:

-   [Конфигурация шлюза](../gateway/configuration.md)
-   [Безопасность](../gateway/security.md)
-   [Реакции](../tools/reactions.md)

[Feishu](./feishu.md)[iMessage](./imessage.md)