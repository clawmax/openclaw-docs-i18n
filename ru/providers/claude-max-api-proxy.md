

  Провайдеры

  
# Claude Max API Proxy

**claude-max-api-proxy** — это инструмент сообщества, который предоставляет вашу подписку Claude Max/Pro в виде API-эндпоинта, совместимого с OpenAI. Это позволяет использовать вашу подписку с любым инструментом, поддерживающим формат API OpenAI.

> **⚠️** Этот путь обеспечивает только техническую совместимость. Anthropic в прошлом блокировал использование некоторых подписок вне Claude Code. Вы должны самостоятельно решить, использовать ли его, и проверить текущие условия Anthropic, прежде чем полагаться на него.

## Зачем это использовать?

| Подход | Стоимость | Лучше всего подходит для |
| --- | --- | --- |
| API Anthropic | Оплата за токен (~15/Minput,15/M input, 15/Minput,75/M output для Opus) | Продакшен-приложения, высокий объем |
| Подписка Claude Max | $200/месяц фиксированно | Личное использование, разработка, неограниченное использование |

Если у вас есть подписка Claude Max и вы хотите использовать её с инструментами, совместимыми с OpenAI, этот прокси может снизить стоимость для некоторых рабочих процессов. API-ключи остаются более ясным путем с точки зрения политики для продакшен-использования.

## Как это работает

```
Ваше приложение → claude-max-api-proxy → Claude Code CLI → Anthropic (через подписку)
     (Формат OpenAI)              (конвертирует формат)      (использует ваш логин)
```

Прокси:

1.  Принимает запросы в формате OpenAI по адресу `http://localhost:3456/v1/chat/completions`
2.  Конвертирует их в команды Claude Code CLI
3.  Возвращает ответы в формате OpenAI (поддерживается потоковая передача)

## Установка

```bash
# Требуется Node.js 20+ и Claude Code CLI
npm install -g claude-max-api-proxy

# Убедитесь, что CLI Claude аутентифицирован
claude --version
```

## Использование

### Запуск сервера

```
claude-max-api
# Сервер запускается по адресу http://localhost:3456
```

### Тестирование

```bash
# Проверка работоспособности
curl http://localhost:3456/health

# Список моделей
curl http://localhost:3456/v1/models

# Завершение чата
curl http://localhost:3456/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-opus-4",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

### С OpenClaw

Вы можете направить OpenClaw на прокси как на пользовательский эндпоинт, совместимый с OpenAI:

```json
{
  env: {
    OPENAI_API_KEY: "not-needed",
    OPENAI_BASE_URL: "http://localhost:3456/v1",
  },
  agents: {
    defaults: {
      model: { primary: "openai/claude-opus-4" },
    },
  },
}
```

## Доступные модели

| ID модели | Соответствует |
| --- | --- |
| `claude-opus-4` | Claude Opus 4 |
| `claude-sonnet-4` | Claude Sonnet 4 |
| `claude-haiku-4` | Claude Haiku 4 |

## Автозапуск на macOS

Создайте LaunchAgent для автоматического запуска прокси:

```bash
cat > ~/Library/LaunchAgents/com.claude-max-api.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>com.claude-max-api</string>
  <key>RunAtLoad</key>
  <true/>
  <key>KeepAlive</key>
  <true/>
  <key>ProgramArguments</key>
  <array>
    <string>/usr/local/bin/node</string>
    <string>/usr/local/lib/node_modules/claude-max-api-proxy/dist/server/standalone.js</string>
  </array>
  <key>EnvironmentVariables</key>
  <dict>
    <key>PATH</key>
    <string>/usr/local/bin:/opt/homebrew/bin:~/.local/bin:/usr/bin:/bin</string>
  </dict>
</dict>
</plist>
EOF

launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.claude-max-api.plist
```

## Ссылки

-   **npm:** [https://www.npmjs.com/package/claude-max-api-proxy](https://www.npmjs.com/package/claude-max-api-proxy)
-   **GitHub:** [https://github.com/atalovesyou/claude-max-api-proxy](https://github.com/atalovesyou/claude-max-api-proxy)
-   **Issues:** [https://github.com/atalovesyou/claude-max-api-proxy/issues](https://github.com/atalovesyou/claude-max-api-proxy/issues)

## Примечания

-   Это **инструмент сообщества**, официально не поддерживаемый Anthropic или OpenClaw
-   Требуется активная подписка Claude Max/Pro с аутентифицированным Claude Code CLI
-   Прокси работает локально и не отправляет данные на сторонние серверы
-   Потоковые ответы полностью поддерживаются

## Смотрите также

-   [Провайдер Anthropic](./anthropic.md) — Нативная интеграция OpenClaw с Claude setup-token или API-ключами
-   [Провайдер OpenAI](./openai.md) — Для подписок OpenAI/Codex

[Cloudflare AI Gateway](./cloudflare-ai-gateway.md)[Deepgram](./deepgram.md)

---