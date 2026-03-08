title: "Руководство по установке и использованию плагина Zalo Personal для OpenClaw"
description: "Узнайте, как установить, настроить и использовать плагин Zalo Personal для OpenClaw, чтобы автоматизировать работу личного аккаунта Zalo через CLI и инструменты агента."
keywords: ["плагин zalo personal", "openclaw zalouser", "автоматизация zalo", "установка плагина openclaw", "пользовательский аккаунт zalo", "настройка канала", "openclaw cli", "чат-бот zalo"]
---

  Расширения

  
# Плагин Zalo Personal

Поддержка личного аккаунта Zalo в OpenClaw через плагин, использующий нативный `zca-js` для автоматизации обычного пользовательского аккаунта Zalo.

> **Предупреждение:** Неофициальная автоматизация может привести к блокировке аккаунта. Используйте на свой страх и риск.

## Наименование

Идентификатор канала — `zalouser`, чтобы явно указать, что это автоматизация **личного пользовательского аккаунта Zalo** (неофициальная). Мы оставляем `zalo` зарезервированным для потенциальной будущей официальной интеграции с API Zalo.

## Где работает

Этот плагин работает **внутри процесса Gateway**. Если вы используете удалённый Gateway, установите/настройте его на **машине, где запущен Gateway**, затем перезапустите Gateway. Внешний бинарный файл CLI `zca`/`openzca` не требуется.

## Установка

### Вариант A: установка из npm

```bash
openclaw plugins install @openclaw/zalouser
```

После этого перезапустите Gateway.

### Вариант B: установка из локальной папки (разработка)

```bash
openclaw plugins install ./extensions/zalouser
cd ./extensions/zalouser && pnpm install
```

После этого перезапустите Gateway.

## Конфигурация

Конфигурация канала находится в `channels.zalouser` (не в `plugins.entries.*`):

```json
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing",
    },
  },
}
```

## CLI

```bash
openclaw channels login --channel zalouser
openclaw channels logout --channel zalouser
openclaw channels status --probe
openclaw message send --channel zalouser --target <threadId> --message "Hello from OpenClaw"
openclaw directory peers list --channel zalouser --query "name"
```

## Инструмент агента

Название инструмента: `zalouser` Действия: `send`, `image`, `link`, `friends`, `groups`, `me`, `status` Действия с сообщениями канала также поддерживают `react` для реакций на сообщения.

[Плагин голосовых вызовов](./voice-call.md)[Манифест плагина](./manifest.md)

---