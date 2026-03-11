

  Обзор платформ

  
# Oracle Cloud

## Цель

Запустить постоянный OpenClaw Gateway на **Always Free** ARM-уровне Oracle Cloud. Бесплатный тариф Oracle может отлично подойти для OpenClaw (особенно если у вас уже есть аккаунт OCI), но у него есть свои компромиссы:

-   Архитектура ARM (большинство вещей работает, но некоторые бинарники могут быть только для x86)
-   Емкость и регистрация могут быть капризными

## Сравнение стоимости (2026)

| Провайдер | Тариф | Характеристики | Цена/мес | Примечания |
| --- | --- | --- | --- | --- |
| Oracle Cloud | Always Free ARM | до 4 OCPU, 24 ГБ ОЗУ | $0 | ARM, ограниченная емкость |
| Hetzner | CX22 | 2 vCPU, 4 ГБ ОЗУ | ~ $4 | Самый дешевый платный вариант |
| DigitalOcean | Basic | 1 vCPU, 1 ГБ ОЗУ | $6 | Простой интерфейс, хорошая документация |
| Vultr | Cloud Compute | 1 vCPU, 1 ГБ ОЗУ | $6 | Много локаций |
| Linode | Nanode | 1 vCPU, 1 ГБ ОЗУ | $5 | Теперь часть Akamai |

* * *

## Предварительные требования

-   Аккаунт Oracle Cloud ([регистрация](https://www.oracle.com/cloud/free/)) — см. [руководство по регистрации от сообщества](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd), если возникнут проблемы
-   Аккаунт Tailscale (бесплатно на [tailscale.com](https://tailscale.com))
-   ~30 минут

## 1) Создание экземпляра OCI

1.  Войдите в [Oracle Cloud Console](https://cloud.oracle.com/)
2.  Перейдите в **Compute → Instances → Create Instance**
3.  Настройте:
    -   **Name:** `openclaw`
    -   **Image:** Ubuntu 24.04 (aarch64)
    -   **Shape:** `VM.Standard.A1.Flex` (Ampere ARM)
    -   **OCPUs:** 2 (или до 4)
    -   **Memory:** 12 ГБ (или до 24 ГБ)
    -   **Boot volume:** 50 ГБ (до 200 ГБ бесплатно)
    -   **SSH key:** Добавьте ваш публичный ключ
4.  Нажмите **Create**
5.  Запишите публичный IP-адрес

**Совет:** Если создание экземпляра завершается ошибкой “Out of capacity”, попробуйте другой домен доступности или повторите попытку позже. Емкость бесплатного тарифа ограничена.

## 2) Подключение и обновление

```bash
# Подключитесь по публичному IP
ssh ubuntu@YOUR_PUBLIC_IP

# Обновите систему
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential
```

**Примечание:** `build-essential` требуется для компиляции некоторых зависимостей под ARM.

## 3) Настройка пользователя и имени хоста

```bash
# Установите имя хоста
sudo hostnamectl set-hostname openclaw

# Установите пароль для пользователя ubuntu
sudo passwd ubuntu

# Включите lingering (поддерживает работу пользовательских сервисов после выхода)
sudo loginctl enable-linger ubuntu
```

## 4) Установите Tailscale

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up --ssh --hostname=openclaw
```

Это включает Tailscale SSH, поэтому вы можете подключаться через `ssh openclaw` с любого устройства в вашей tailnet — публичный IP не нужен. Проверьте:

```bash
tailscale status
```

**Отныне подключайтесь через Tailscale:** `ssh ubuntu@openclaw` (или используйте Tailscale IP).

## 5) Установите OpenClaw

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
source ~/.bashrc
```

Когда появится запрос “How do you want to hatch your bot?”, выберите **“Do this later”**.

> Примечание: Если возникнут проблемы со сборкой под ARM, начните с системных пакетов (например, `sudo apt install -y build-essential`), прежде чем обращаться к Homebrew.

## 6) Настройте шлюз (loopback + аутентификация по токену) и включите Tailscale Serve

Используйте аутентификацию по токену по умолчанию. Это предсказуемо и позволяет избежать необходимости во флагах "insecure auth" в UI управления.

```bash
# Оставьте шлюз приватным на ВМ
openclaw config set gateway.bind loopback

# Требуйте аутентификацию для шлюза и UI управления
openclaw config set gateway.auth.mode token
openclaw doctor --generate-gateway-token

# Откройте доступ через Tailscale Serve (HTTPS + доступ из tailnet)
openclaw config set gateway.tailscale.mode serve
openclaw config set gateway.trustedProxies '["127.0.0.1"]'

systemctl --user restart openclaw-gateway
```

## 7) Проверка

```bash
# Проверьте версию
openclaw --version

# Проверьте статус демона
systemctl --user status openclaw-gateway

# Проверьте Tailscale Serve
tailscale serve status

# Протестируйте локальный ответ
curl http://localhost:18789
```

## 8) Заблокируйте VCN Security

Теперь, когда всё работает, заблокируйте VCN, чтобы запретить весь трафик, кроме Tailscale. Виртуальная облачная сеть (VCN) Oracle действует как межсетевой экран на границе сети — трафик блокируется до того, как достигнет вашего экземпляра.

1.  Перейдите в **Networking → Virtual Cloud Networks** в OCI Console
2.  Нажмите на вашу VCN → **Security Lists** → Default Security List
3.  **Удалите** все правила для входящего трафика, кроме:
    -   `0.0.0.0/0 UDP 41641` (Tailscale)
4.  Оставьте правила для исходящего трафика по умолчанию (разрешить весь исходящий)

Это блокирует SSH на порту 22, HTTP, HTTPS и всё остальное на границе сети. Отныне вы можете подключаться только через Tailscale.

* * *

## Доступ к UI управления

С любого устройства в вашей сети Tailscale:

```
https://openclaw.<tailnet-name>.ts.net/
```

Замените `<tailnet-name>` на имя вашей tailnet (видно в `tailscale status`). SSH-туннель не нужен. Tailscale предоставляет:

-   HTTPS-шифрование (автоматические сертификаты)
-   Аутентификацию через идентификатор Tailscale
-   Доступ с любого устройства в вашей tailnet (ноутбук, телефон и т.д.)

* * *

## Безопасность: VCN + Tailscale (рекомендуемый базовый уровень)

При заблокированном VCN (открыт только UDP 41641) и шлюзе, привязанном к loopback, вы получаете сильную защиту в глубину: публичный трафик блокируется на границе сети, а административный доступ осуществляется через вашу tailnet. Эта настройка часто *устраняет необходимость* в дополнительных правилах межсетевого экрана на уровне хоста для защиты от перебора SSH из интернета — но вы всё равно должны поддерживать ОС в актуальном состоянии, запускать `openclaw security audit` и проверять, что вы случайно не слушаете публичные интерфейсы.

### Что уже защищено

| Традиционный шаг | Нужен? | Почему |
| --- | --- | --- |
| Межсетевой экран UFW | Нет | VCN блокирует трафик до достижения экземпляра |
| fail2ban | Нет | Нет перебора, если порт 22 заблокирован на VCN |
| Усиление sshd | Нет | Tailscale SSH не использует sshd |
| Отключение входа root | Нет | Tailscale использует идентификатор Tailscale, а не системных пользователей |
| Аутентификация только по SSH-ключу | Нет | Tailscale аутентифицирует через вашу tailnet |
| Усиление IPv6 | Обычно нет | Зависит от настроек вашей VCN/подсети; проверьте, что фактически назначено/открыто |

### Всё ещё рекомендуется

-   **Права доступа к учетным данным:** `chmod 700 ~/.openclaw`
-   **Проверка безопасности:** `openclaw security audit`
-   **Обновления системы:** Регулярно `sudo apt update && sudo apt upgrade`
-   **Мониторинг Tailscale:** Просматривайте устройства в [консоли администрирования Tailscale](https://login.tailscale.com/admin)

### Проверка состояния безопасности

```bash
# Убедитесь, что нет публичных портов в состоянии прослушивания
sudo ss -tlnp | grep -v '127.0.0.1\|::1'

# Проверьте, активен ли Tailscale SSH
tailscale status | grep -q 'offers: ssh' && echo "Tailscale SSH активен"

# Опционально: полностью отключите sshd
sudo systemctl disable --now ssh
```

* * *

## Резервный вариант: SSH-туннель

Если Tailscale Serve не работает, используйте SSH-туннель:

```bash
# С вашего локального компьютера (через Tailscale)
ssh -L 18789:127.0.0.1:18789 ubuntu@openclaw
```

Затем откройте `http://localhost:18789`.

* * *

## Устранение неполадок

### Не удается создать экземпляр (“Out of capacity”)

Бесплатные ARM-экземпляры популярны. Попробуйте:

-   Другой домен доступности
-   Повторить попытку в непиковое время (рано утром)
-   Использовать фильтр “Always Free” при выборе формы (shape)

### Tailscale не подключается

```bash
# Проверьте статус
sudo tailscale status

# Повторно пройдите аутентификацию
sudo tailscale up --ssh --hostname=openclaw --reset
```

### Шлюз не запускается

```bash
openclaw gateway status
openclaw doctor --non-interactive
journalctl --user -u openclaw-gateway -n 50
```

### Не удается получить доступ к UI управления

```bash
# Убедитесь, что Tailscale Serve работает
tailscale serve status

# Проверьте, что шлюз слушает порт
curl http://localhost:18789

# Перезапустите при необходимости
systemctl --user restart openclaw-gateway
```

### Проблемы с ARM-бинарниками

Некоторые инструменты могут не иметь сборок для ARM. Проверьте:

```bash
uname -m  # Должно показывать aarch64
```

Большинство npm-пакетов работают нормально. Для бинарников ищите релизы `linux-arm64` или `aarch64`.

* * *

## Сохранение состояния

Все данные состояния хранятся в:

-   `~/.openclaw/` — конфигурация, учетные данные, данные сессии
-   `~/.openclaw/workspace/` — рабочая область (SOUL.md, память, артефакты)

Регулярно создавайте резервные копии:

```bash
tar -czvf openclaw-backup.tar.gz ~/.openclaw ~/.openclaw/workspace
```

* * *

## Смотрите также

-   [Удаленный доступ к шлюзу](../gateway/remote.md) — другие схемы удаленного доступа
-   [Интеграция с Tailscale](../gateway/tailscale.md) — полная документация по Tailscale
-   [Конфигурация шлюза](../gateway/configuration.md) — все параметры конфигурации
-   [Руководство по DigitalOcean](./digitalocean.md) — если нужен платный тариф и простая регистрация
-   [Руководство по Hetzner](../install/hetzner.md) — альтернатива на основе Docker

[DigitalOcean](./digitalocean.md)[Raspberry Pi](./raspberry-pi.md)