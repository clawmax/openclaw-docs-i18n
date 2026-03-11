

  Настройка разработчика

  
# Рабочий процесс разработки Pi

В этом руководстве представлен разумный рабочий процесс для работы с интеграцией Pi в OpenClaw.

## Проверка типов и линтинг

-   Проверка типов и сборка: `pnpm build`
-   Линтинг: `pnpm lint`
-   Проверка форматирования: `pnpm format`
-   Полная проверка перед отправкой: `pnpm lint && pnpm build && pnpm test`

## Запуск тестов Pi

Запустите набор тестов, ориентированных на Pi, напрямую с помощью Vitest:

```bash
pnpm test -- \
  "src/agents/pi-*.test.ts" \
  "src/agents/pi-embedded-*.test.ts" \
  "src/agents/pi-tools*.test.ts" \
  "src/agents/pi-settings.test.ts" \
  "src/agents/pi-tool-definition-adapter*.test.ts" \
  "src/agents/pi-extensions/**/*.test.ts"
```

Чтобы включить упражнение с реальным провайдером:

```
OPENCLAW_LIVE_TEST=1 pnpm test -- src/agents/pi-embedded-runner-extraparams.live.test.ts
```

Это охватывает основные модульные наборы Pi:

-   `src/agents/pi-*.test.ts`
-   `src/agents/pi-embedded-*.test.ts`
-   `src/agents/pi-tools*.test.ts`
-   `src/agents/pi-settings.test.ts`
-   `src/agents/pi-tool-definition-adapter.test.ts`
-   `src/agents/pi-extensions/*.test.ts`

## Ручное тестирование

Рекомендуемый процесс:

-   Запустите шлюз в режиме разработки:
    -   `pnpm gateway:dev`
-   Запустите агента напрямую:
    -   `pnpm openclaw agent --message "Hello" --thinking low`
-   Используйте TUI для интерактивной отладки:
    -   `pnpm tui`

Для проверки поведения вызова инструментов запросите действие `read` или `exec`, чтобы увидеть потоковую передачу инструментов и обработку полезной нагрузки.

## Полный сброс состояния

Состояние хранится в каталоге состояния OpenClaw. По умолчанию это `~/.openclaw`. Если установлена переменная `OPENCLAW_STATE_DIR`, используйте этот каталог. Чтобы сбросить всё:

-   `openclaw.json` для конфигурации
-   `credentials/` для профилей аутентификации и токенов
-   `agents//sessions/` для истории сессий агента
-   `agents//sessions.json` для индекса сессий
-   `sessions/` если существуют устаревшие пути
-   `workspace/` если нужна чистая рабочая область

Если нужно сбросить только сессии, удалите `agents//sessions/` и `agents//sessions.json` для этого агента. Сохраните `credentials/`, если не хотите повторно проходить аутентификацию.

## Ссылки

-   [https://docs.openclaw.ai/testing](https://docs.openclaw.ai/testing)
-   [https://docs.openclaw.ai/start/getting-started](https://docs.openclaw.ai/start/getting-started)

[Настройка](./start/setup.md)[CI Пайплайн](./ci.md)

---