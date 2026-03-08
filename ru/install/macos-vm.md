

  Хостинг и развертывание

  
# Виртуальные машины macOS

## Рекомендуемый вариант по умолчанию (для большинства пользователей)

-   **Небольшой Linux VPS** для постоянно работающего Шлюза и низкой стоимости. См. [Хостинг VPS](../vps.md).
-   **Выделенное оборудование** (Mac mini или Linux-машина), если нужен полный контроль и **резидентский IP** для автоматизации браузера. Многие сайты блокируют IP-адреса дата-центров, поэтому локальный просмотр часто работает лучше.
-   **Гибридный вариант:** держите Шлюз на дешевом VPS, а ваш Mac подключайте как **узел**, когда нужна автоматизация браузера/UI. См. [Узлы](../nodes.md) и [Удаленный Шлюз](../gateway/remote.md).

Используйте виртуальную машину macOS, когда вам нужны возможности, доступные только в macOS (iMessage/BlueBubbles), или требуется строгая изоляция от вашего основного Mac.

## Варианты виртуальных машин macOS

### Локальная VM на вашем Mac с Apple Silicon (Lume)

Запустите OpenClaw в изолированной виртуальной машине macOS на вашем существующем Mac с Apple Silicon с помощью [Lume](https://cua.ai/docs/lume). Это дает вам:

-   Полноценную среду macOS в изоляции (ваш хост остается чистым)
-   Поддержку iMessage через BlueBubbles (невозможно на Linux/Windows)
-   Мгновенный сброс путем клонирования ВМ
-   Никаких дополнительных затрат на оборудование или облако

### Провайдеры размещенных Mac (облако)

Если вам нужен macOS в облаке, также подойдут провайдеры размещенных Mac:

-   [MacStadium](https://www.macstadium.com/) (размещенные Mac)
-   Другие поставщики размещенных Mac также работают; следуйте их инструкциям по VM + SSH

Как только у вас будет доступ по SSH к виртуальной машине macOS, переходите к шагу 6 ниже.

* * *

## Быстрый путь (Lume, для опытных пользователей)

1.  Установите Lume
2.  `lume create openclaw --os macos --ipsw latest`
3.  Пройдите Ассистент настройки, включите Удаленный вход (SSH)
4.  `lume run openclaw --no-display`
5.  Подключитесь по SSH, установите OpenClaw, настройте каналы
6.  Готово

* * *

## Что вам понадобится (Lume)

-   Mac с Apple Silicon (M1/M2/M3/M4)
-   macOS Sequoia или новее на хосте
-   ~60 ГБ свободного места на диске на одну ВМ
-   ~20 минут

* * *

## 1) Установите Lume

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/trycua/cua/main/libs/lume/scripts/install.sh)"
```

Если `~/.local/bin` нет в вашем PATH:

```bash
echo 'export PATH="$PATH:$HOME/.local/bin"' >> ~/.zshrc && source ~/.zshrc
```

Проверьте:

```bash
lume --version
```

Документация: [Установка Lume](https://cua.ai/docs/lume/guide/getting-started/installation)

* * *

## 2) Создайте виртуальную машину macOS

```bash
lume create openclaw --os macos --ipsw latest
```

Это скачает macOS и создаст ВМ. Окно VNC откроется автоматически. Примечание: загрузка может занять некоторое время в зависимости от вашего соединения.

* * *

## 3) Пройдите Ассистент настройки

В окне VNC:

1.  Выберите язык и регион
2.  Пропустите Apple ID (или войдите, если позже понадобится iMessage)
3.  Создайте учетную запись пользователя (запомните имя пользователя и пароль)
4.  Пропустите все дополнительные функции

После завершения настройки включите SSH:

1.  Откройте Системные настройки → Основные → Общий доступ
2.  Включите «Удаленный вход»

* * *

## 4) Получите IP-адрес ВМ

```bash
lume get openclaw
```

Найдите IP-адрес (обычно `192.168.64.x`).

* * *

## 5) Подключитесь к ВМ по SSH

```bash
ssh youruser@192.168.64.X
```

Замените `youruser` на созданную учетную запись, а IP — на IP-адрес вашей ВМ.

* * *

## 6) Установите OpenClaw

Внутри ВМ:

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

Следуйте подсказкам мастера настройки, чтобы настроить вашего провайдера моделей (Anthropic, OpenAI и т.д.).

* * *

## 7) Настройте каналы

Отредактируйте файл конфигурации:

```bash
nano ~/.openclaw/openclaw.json
```

Добавьте ваши каналы:

```json
{
  "channels": {
    "whatsapp": {
      "dmPolicy": "allowlist",
      "allowFrom": ["+15551234567"]
    },
    "telegram": {
      "botToken": "YOUR_BOT_TOKEN"
    }
  }
}
```

Затем войдите в WhatsApp (отсканируйте QR-код):

```bash
openclaw channels login
```

* * *

## 8) Запустите ВМ в headless-режиме

Остановите ВМ и перезапустите без дисплея:

```bash
lume stop openclaw
lume run openclaw --no-display
```

ВМ работает в фоновом режиме. Демон OpenClaw поддерживает работу шлюза. Чтобы проверить статус:

```bash
ssh youruser@192.168.64.X "openclaw status"
```

* * *

## Бонус: Интеграция с iMessage

Это ключевая особенность работы на macOS. Используйте [BlueBubbles](https://bluebubbles.app), чтобы добавить iMessage в OpenClaw. Внутри ВМ:

1.  Скачайте BlueBubbles с bluebubbles.app
2.  Войдите с вашим Apple ID
3.  Включите Web API и установите пароль
4.  Направьте вебхуки BlueBubbles на ваш шлюз (пример: `https://your-gateway-host:3000/bluebubbles-webhook?password=`)

Добавьте в конфигурацию OpenClaw:

```json
{
  "channels": {
    "bluebubbles": {
      "serverUrl": "http://localhost:1234",
      "password": "your-api-password",
      "webhookPath": "/bluebubbles-webhook"
    }
  }
}
```

Перезапустите шлюз. Теперь ваш агент может отправлять и получать iMessages. Полная инструкция по настройке: [Канал BlueBubbles](../channels/bluebubbles.md)

* * *

## Сохраните золотой образ

Прежде чем продолжить настройку, создайте снапшот чистого состояния:

```bash
lume stop openclaw
lume clone openclaw openclaw-golden
```

Сбросьте в любое время:

```bash
lume stop openclaw && lume delete openclaw
lume clone openclaw-golden openclaw
lume run openclaw --no-display
```

* * *

## Работа 24/7

Чтобы ВМ работала постоянно:

-   Держите ваш Mac подключенным к питанию
-   Отключите спящий режим в Системных настройках → Экономия энергии
-   Используйте `caffeinate` при необходимости

Для действительно непрерывной работы рассмотрите выделенный Mac mini или небольшой VPS. См. [Хостинг VPS](../vps.md).

* * *

## Устранение неполадок

| Проблема | Решение |
| --- | --- |
| Не удается подключиться по SSH | Проверьте, включен ли «Удаленный вход» в Системных настройках ВМ |
| IP-адрес ВМ не отображается | Подождите полной загрузки ВМ, снова выполните `lume get openclaw` |
| Команда lume не найдена | Добавьте `~/.local/bin` в ваш PATH |
| QR-код WhatsApp не сканируется | Убедитесь, что вы вошли в ВМ (а не на хост), когда запускаете `openclaw channels login` |

* * *

## Связанная документация

-   [Хостинг VPS](../vps.md)
-   [Узлы](../nodes.md)
-   [Удаленный Шлюз](../gateway/remote.md)
-   [Канал BlueBubbles](../channels/bluebubbles.md)
-   [Быстрый старт Lume](https://cua.ai/docs/lume/guide/getting-started/quickstart)
-   [Справочник по CLI Lume](https://cua.ai/docs/lume/reference/cli-reference)
-   [Автоматическая настройка ВМ](https://cua.ai/docs/lume/guide/fundamentals/unattended-setup) (продвинутый уровень)
-   [Песочница Docker](./docker.md) (альтернативный подход к изоляции)

[GCP](./gcp.md)[exe.dev](./exe-dev.md)