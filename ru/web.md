title: "Веб-интерфейсы OpenClaw: Панель управления, вебхуки и безопасность"
description: "Узнайте, как настроить веб-интерфейс шлюза OpenClaw, Панель управления, вебхуки и доступ через Tailscale. Настройте безопасность, аутентификацию и режимы развертывания."
keywords: ["веб-интерфейс openclaw", "панель управления шлюза", "интеграция tailscale", "настройка вебхуков", "безопасность шлюза", "привязка loopback", "развертывание funnel", "токен аутентификации"]
---

  Веб-интерфейсы

  
# Веб

Шлюз обслуживает небольшую **браузерную Панель управления** (Vite + Lit) на том же порту, что и WebSocket шлюза:

-   по умолчанию: `http://:18789/`
-   опциональный префикс: установите `gateway.controlUi.basePath` (например, `/openclaw`)

Возможности описаны в разделе [Панель управления](./web/control-ui.md). На этой странице рассматриваются режимы привязки, безопасность и веб-интерфейсы.

## Вебхуки

Когда `hooks.enabled=true`, Шлюз также предоставляет небольшую конечную точку для вебхуков на том же HTTP-сервере. См. [Конфигурация шлюза](./gateway/configuration.md) → `hooks` для информации об аутентификации и форматах данных.

## Конфигурация (включено по умолчанию)

Панель управления **включена по умолчанию**, если присутствуют её файлы (`dist/control-ui`). Вы можете управлять ею через конфигурацию:

```json
{
  gateway: {
    controlUi: { enabled: true, basePath: "/openclaw" }, // basePath опционально
  },
}
```

## Доступ через Tailscale

### Встроенный Serve (рекомендуется)

Оставьте шлюз на loopback-адресе и позвольте Tailscale Serve выступать в роли прокси:

```json
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" },
  },
}
```

Затем запустите шлюз:

```bash
openclaw gateway
```

Откройте:

-   `https:///` (или ваш настроенный `gateway.controlUi.basePath`)

### Привязка к Tailnet + токен

```json
{
  gateway: {
    bind: "tailnet",
    controlUi: { enabled: true },
    auth: { mode: "token", token: "your-token" },
  },
}
```

Затем запустите шлюз (токен требуется для привязки не к loopback-адресу):

```bash
openclaw gateway
```

Откройте:

-   `http://<tailscale-ip>:18789/` (или ваш настроенный `gateway.controlUi.basePath`)

### Публичный интернет (Funnel)

```json
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password" }, // или OPENCLAW_GATEWAY_PASSWORD
  },
}
```

## Примечания по безопасности

-   Аутентификация шлюза требуется по умолчанию (токен/пароль или заголовки идентификации Tailscale).
-   Привязка не к loopback-адресу по-прежнему **требует** общего токена/пароля (`gateway.auth` или переменная окружения).
-   Мастер настройки по умолчанию генерирует токен шлюза (даже для loopback).
-   Панель управления отправляет `connect.params.auth.token` или `connect.params.auth.password`.
-   Для развертывания Панели управления не на loopback-адресе явно задайте `gateway.controlUi.allowedOrigins` (полные origin-адреса). Без этого запуск шлюза по умолчанию будет отклонен.
-   `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback=true` включает режим отката на заголовок Host для определения origin, но это опасное снижение уровня безопасности.
-   При использовании Serve заголовки идентификации Tailscale могут удовлетворить требованиям аутентификации Панели управления/WebSocket, когда `gateway.auth.allowTailscale` имеет значение `true` (токен/пароль не требуются). Конечные точки HTTP API по-прежнему требуют токен/пароль. Установите `gateway.auth.allowTailscale: false`, чтобы требовать явные учетные данные. См. [Tailscale](./gateway/tailscale.md) и [Безопасность](./gateway/security.md). Этот поток без токена предполагает, что хост шлюза является доверенным.
-   `gateway.tailscale.mode: "funnel"` требует `gateway.auth.mode: "password"` (общий пароль).

## Сборка Панели управления

Шлюз обслуживает статические файлы из `dist/control-ui`. Соберите их с помощью:

```bash
pnpm ui:build # автоматически устанавливает зависимости UI при первом запуске
```

[CONTRIBUTING THREAT MODEL](./security/CONTRIBUTING-THREAT-MODEL.md)[Панель управления](./web/control-ui.md)