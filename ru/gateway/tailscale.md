title: "Настройка Tailscale Serve или Funnel для удаленного доступа к OpenClaw Gateway"
description: "Узнайте, как настроить Tailscale Serve или Funnel для безопасного удаленного доступа к панели управления OpenClaw Gateway и порту WebSocket с использованием HTTPS и аутентификации."
keywords: ["tailscale", "openclaw gateway", "удаленный доступ", "tailscale serve", "tailscale funnel", "настройка шлюза", "безопасный туннель", "websocket"]
---

  Удаленный доступ

  
# Tailscale

OpenClaw может автоматически настроить Tailscale **Serve** (tailnet) или **Funnel** (публичный) для панели управления Gateway и порта WebSocket. Это позволяет Gateway оставаться привязанным к loopback-адресу, в то время как Tailscale обеспечивает HTTPS, маршрутизацию и (для Serve) заголовки идентификации.

## Режимы

-   `serve`: Serve только для tailnet через `tailscale serve`. Шлюз остается на `127.0.0.1`.
-   `funnel`: Публичный HTTPS через `tailscale funnel`. OpenClaw требует общий пароль.
-   `off`: По умолчанию (без автоматизации Tailscale).

## Аутентификация

Установите `gateway.auth.mode` для управления подтверждением подключения:

-   `token` (по умолчанию, когда установлена переменная `OPENCLAW_GATEWAY_TOKEN`)
-   `password` (общий секрет через `OPENCLAW_GATEWAY_PASSWORD` или конфигурацию)

Когда `tailscale.mode = "serve"` и `gateway.auth.allowTailscale` имеет значение `true`, аутентификация в панели управления/WebSocket может использовать заголовки идентификации Tailscale (`tailscale-user-login`) без предоставления токена/пароля. OpenClaw проверяет личность, разрешая адрес `x-forwarded-for` через локальный демон Tailscale (`tailscale whois`) и сопоставляя его с заголовком перед принятием. OpenClaw рассматривает запрос как Serve только тогда, когда он поступает с loopback-адреса с заголовками Tailscale `x-forwarded-for`, `x-forwarded-proto` и `x-forwarded-host`. Конечные точки HTTP API (например, `/v1/*`, `/tools/invoke` и `/api/channels/*`) по-прежнему требуют аутентификации по токену/паролю. Этот поток без токена предполагает, что хост шлюза является доверенным. Если на том же хосте может выполняться ненадежный локальный код, отключите `gateway.auth.allowTailscale` и вместо этого требуйте аутентификацию по токену/паролю. Чтобы требовать явные учетные данные, установите `gateway.auth.allowTailscale: false` или принудительно укажите `gateway.auth.mode: "password"`.

## Примеры конфигурации

### Только для tailnet (Serve)

```json
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" },
  },
}
```

Открыть: `https:///` (или ваш настроенный `gateway.controlUi.basePath`)

### Только для tailnet (привязка к IP-адресу Tailnet)

Используйте этот вариант, когда хотите, чтобы Gateway прослушивал напрямую IP-адрес Tailnet (без Serve/Funnel).

```json
{
  gateway: {
    bind: "tailnet",
    auth: { mode: "token", token: "your-token" },
  },
}
```

Подключение с другого устройства в Tailnet:

-   Панель управления: `http://<tailscale-ip>:18789/`
-   WebSocket: `ws://<tailscale-ip>:18789`

Примечание: loopback (`http://127.0.0.1:18789`) в этом режиме работать **не будет**.

### Публичный интернет (Funnel + общий пароль)

```json
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password", password: "replace-me" },
  },
}
```

Предпочтительнее использовать переменную окружения `OPENCLAW_GATEWAY_PASSWORD`, а не хранить пароль в файле.

## Примеры CLI

```bash
openclaw gateway --tailscale serve
openclaw gateway --tailscale funnel --auth password
```

## Примечания

-   Tailscale Serve/Funnel требует установки и входа в систему через CLI `tailscale`.
-   `tailscale.mode: "funnel"` отказывается запускаться, если режим аутентификации не `password`, чтобы избежать публичного доступа.
-   Установите `gateway.tailscale.resetOnExit`, если хотите, чтобы OpenClaw отменял конфигурацию `tailscale serve` или `tailscale funnel` при завершении работы.
-   `gateway.bind: "tailnet"` — это прямая привязка к Tailnet (без HTTPS, без Serve/Funnel).
-   `gateway.bind: "auto"` предпочитает loopback; используйте `tailnet`, если нужен доступ только внутри Tailnet.
-   Serve/Funnel предоставляют доступ только к **панели управления Gateway + WebSocket**. Узлы подключаются через ту же конечную точку WebSocket Gateway, поэтому Serve может работать для доступа узлов.

## Управление браузером (удаленный Gateway + локальный браузер)

Если вы запускаете Gateway на одной машине, но хотите управлять браузером на другой машине, запустите **хост узла** на машине с браузером и держите обе машины в одной сети Tailnet. Gateway будет проксировать действия браузера на узел; отдельный сервер управления или URL Serve не нужны. Избегайте использования Funnel для управления браузером; рассматривайте сопряжение узлов как доступ оператора.

## Предварительные требования и ограничения Tailscale

-   Serve требует включения HTTPS для вашей сети Tailnet; CLI запросит подтверждение, если оно отсутствует.
-   Serve добавляет заголовки идентификации Tailscale; Funnel этого не делает.
-   Funnel требует Tailscale версии 1.38.3+, MagicDNS, включенный HTTPS и атрибут узла funnel.
-   Funnel поддерживает только порты `443`, `8443` и `10000` через TLS.
-   Funnel на macOS требует использования варианта приложения Tailscale с открытым исходным кодом.

## Узнать больше

-   Обзор Tailscale Serve: [https://tailscale.com/kb/1312/serve](https://tailscale.com/kb/1312/serve)
-   Команда `tailscale serve`: [https://tailscale.com/kb/1242/tailscale-serve](https://tailscale.com/kb/1242/tailscale-serve)
-   Обзор Tailscale Funnel: [https://tailscale.com/kb/1223/tailscale-funnel](https://tailscale.com/kb/1223/tailscale-funnel)
-   Команда `tailscale funnel`: [https://tailscale.com/kb/1311/tailscale-funnel](https://tailscale.com/kb/1311/tailscale-funnel)

[Настройка удаленного Gateway](./remote-gateway-readme.md)[Формальная верификация (модели безопасности)](../security/formal-verification.md)