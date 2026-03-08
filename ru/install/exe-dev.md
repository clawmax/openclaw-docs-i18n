title: "Развертывание OpenClaw Gateway на виртуальной машине exe.dev"
description: "Пошаговое руководство по установке и запуску шлюза OpenClaw AI на виртуальной машине exe.dev. Изучите автоматическую и ручную настройку для удаленного доступа через exe.xyz."
keywords: ["openclaw", "exe.dev", "развертывание шлюза", "виртуальная машина", "nginx proxy", "удаленный доступ", "автоматическая установка", "ручная установка"]
---

  Хостинг и развертывание

  
# exe.dev

Цель: запустить OpenClaw Gateway на виртуальной машине exe.dev, доступной с вашего ноутбука по адресу: `https://<vm-name>.exe.xyz` Эта страница предполагает использование стандартного образа **exeuntu** от exe.dev. Если вы выбрали другой дистрибутив, сопоставьте пакеты соответствующим образом.

## Быстрый путь для начинающих

1.  [https://exe.new/openclaw](https://exe.new/openclaw)
2.  Заполните ваш ключ аутентификации/токен при необходимости
3.  Нажмите на "Agent" рядом с вашей виртуальной машиной и подождите…
4.  ???
5.  Profit

## Что вам понадобится

-   Аккаунт на exe.dev
-   Доступ `ssh exe.dev` к виртуальным машинам [exe.dev](https://exe.dev) (опционально)

## Автоматическая установка с помощью Shelley

Shelley, агент [exe.dev](https://exe.dev), может мгновенно установить OpenClaw с помощью нашего промпта. Используемый промпт приведен ниже:

```bash
Set up OpenClaw (https://docs.openclaw.ai/install) on this VM. Use the non-interactive and accept-risk flags for openclaw onboarding. Add the supplied auth or token as needed. Configure nginx to forward from the default port 18789 to the root location on the default enabled site config, making sure to enable Websocket support. Pairing is done by "openclaw devices list" and "openclaw devices approve <request id>". Make sure the dashboard shows that OpenClaw's health is OK. exe.dev handles forwarding from port 8000 to port 80/443 and HTTPS for us, so the final "reachable" should be <vm-name>.exe.xyz, without port specification.
```

## Ручная установка

## 1) Создайте виртуальную машину

С вашего устройства:

```bash
ssh exe.dev new
```

Затем подключитесь:

```bash
ssh <vm-name>.exe.xyz
```

Совет: сохраняйте состояние этой виртуальной машины (**stateful**). OpenClaw хранит состояние в `~/.openclaw/` и `~/.openclaw/workspace/`.

## 2) Установите необходимые компоненты (на виртуальной машине)

```bash
sudo apt-get update
sudo apt-get install -y git curl jq ca-certificates openssl
```

## 3) Установите OpenClaw

Запустите скрипт установки OpenClaw:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

## 4) Настройте nginx для проксирования OpenClaw на порт 8000

Отредактируйте файл `/etc/nginx/sites-enabled/default`:

```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    listen 8000;
    listen [::]:8000;

    server_name _;

    location / {
        proxy_pass http://127.0.0.1:18789;
        proxy_http_version 1.1;

        # WebSocket support
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # Standard proxy headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Timeout settings for long-lived connections
        proxy_read_timeout 86400s;
        proxy_send_timeout 86400s;
    }
}
```

## 5) Получите доступ к OpenClaw и предоставьте привилегии

Перейдите по адресу `https://<vm-name>.exe.xyz/` (см. вывод Control UI после onboarding). Если запрашивается аутентификация, вставьте токен из `gateway.auth.token` на виртуальной машине (получите его командой `openclaw config get gateway.auth.token` или сгенерируйте с помощью `openclaw doctor --generate-gateway-token`). Одобрите устройства командами `openclaw devices list` и `openclaw devices approve `. Если сомневаетесь, используйте Shelley из вашего браузера!

## Удаленный доступ

Удаленный доступ обрабатывается аутентификацией [exe.dev](https://exe.dev). По умолчанию HTTP-трафик с порта 8000 перенаправляется на `https://<vm-name>.exe.xyz` с аутентификацией по email.

## Обновление

```bash
npm i -g openclaw@latest
openclaw doctor
openclaw gateway restart
openclaw health
```

Руководство: [Обновление](./updating.md)

[Виртуальные машины macOS](./macos-vm.md)[Развертывание на Railway](./railway.md)