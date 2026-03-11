

  Примечания к выпуску

  
# Тесты

-   Полный набор тестов (наборы, live, Docker): [Тестирование](../help/testing.md)
-   `pnpm test:force`: Убивает любой зависший процесс шлюза, занимающий порт управления по умолчанию, затем запускает полный набор Vitest с изолированным портом шлюза, чтобы тесты сервера не конфликтовали с запущенным экземпляром. Используйте эту команду, если предыдущий запуск шлюза оставил порт 18789 занятым.
-   `pnpm test:coverage`: Запускает набор модульных тестов с покрытием V8 (через `vitest.unit.config.ts`). Глобальные пороговые значения: 70% строк/ветвей/функций/утверждений. Покрытие исключает точки входа с высокой интеграционной нагрузкой (CLI-обвязка, мосты шлюза/telegram, статический сервер веб-чата), чтобы цель оставалась сосредоточенной на логике, поддающейся модульному тестированию.
-   `pnpm test` на Node 24+: OpenClaw автоматически отключает `vmForks` в Vitest и использует `forks`, чтобы избежать ошибок `ERR_VM_MODULE_LINK_FAILURE` / `module is already linked`. Вы можете принудительно задать поведение с помощью `OPENCLAW_TEST_VM_FORKS=0|1`.
-   `pnpm test`: по умолчанию запускает быстрый основной набор модульных тестов для быстрой локальной обратной связи.
-   `pnpm test:channels`: запускает наборы тестов, ориентированные на каналы.
-   `pnpm test:extensions`: запускает наборы тестов для расширений/плагинов.
-   Интеграция шлюза: опционально через `OPENCLAW_TEST_INCLUDE_GATEWAY=1 pnpm test` или `pnpm test:gateway`.
-   `pnpm test:e2e`: Запускает сквозные smoke-тесты шлюза (мульти-инстансные WS/HTTP/связки узлов). По умолчанию использует `vmForks` + адаптивных воркеров в `vitest.e2e.config.ts`; настройте с помощью `OPENCLAW_E2E_WORKERS=` и установите `OPENCLAW_E2E_VERBOSE=1` для подробных логов.
-   `pnpm test:live`: Запускает live-тесты провайдеров (minimax/zai). Требует API-ключи и `LIVE=1` (или специфичный для провайдера `*_LIVE_TEST=1`), чтобы снять пропуск.

## Локальные проверки для PR

Для локальных проверок перед принятием PR выполните:

-   `pnpm check`
-   `pnpm build`
-   `pnpm test`
-   `pnpm check:docs`

Если `pnpm test` ведёт себя нестабильно на загруженной машине, перезапустите один раз, прежде чем считать это регрессией, затем изолируйте с помощью `pnpm vitest run <path/to/test>`. Для машин с ограниченной памятью используйте:

-   `OPENCLAW_TEST_PROFILE=low OPENCLAW_TEST_SERIAL_GATEWAY=1 pnpm test`

## Бенчмарк задержки модели (локальные ключи)

Скрипт: [`scripts/bench-model.ts`](https://github.com/openclaw/openclaw/blob/main/scripts/bench-model.ts) Использование:

-   `source ~/.profile && pnpm tsx scripts/bench-model.ts --runs 10`
-   Опциональные переменные окружения: `MINIMAX_API_KEY`, `MINIMAX_BASE_URL`, `MINIMAX_MODEL`, `ANTHROPIC_API_KEY`
-   Промпт по умолчанию: “Ответьте одним словом: ok. Без знаков препинания и лишнего текста.”

Последний запуск (2025-12-31, 20 прогонов):

-   minimax медиана 1279мс (мин 1114, макс 2431)
-   opus медиана 2454мс (мин 1224, макс 3170)

## Бенчмарк запуска CLI

Скрипт: [`scripts/bench-cli-startup.ts`](https://github.com/openclaw/openclaw/blob/main/scripts/bench-cli-startup.ts) Использование:

-   `pnpm tsx scripts/bench-cli-startup.ts`
-   `pnpm tsx scripts/bench-cli-startup.ts --runs 12`
-   `pnpm tsx scripts/bench-cli-startup.ts --entry dist/entry.js --timeout-ms 45000`

Этот бенчмарк тестирует следующие команды:

-   `--version`
-   `--help`
-   `health --json`
-   `status --json`
-   `status`

Вывод включает среднее значение, p50, p95, мин/макс и распределение кодов завершения/сигналов для каждой команды.

## Сквозное тестирование онбординга (Docker)

Docker опционален; он нужен только для контейнеризованных smoke-тестов онбординга. Полный процесс холодного старта в чистом контейнере Linux:

```
scripts/e2e/onboard-docker.sh
```

Этот скрипт управляет интерактивным мастером через псевдо-tty, проверяет конфигурационные/рабочие/сессионные файлы, затем запускает шлюз и выполняет `openclaw health`.

## Smoke-тест импорта QR (Docker)

Гарантирует, что `qrcode-terminal` загружается под Node 22+ в Docker:

```bash
pnpm test:docker:qr
```

[Чек-лист выпуска](./RELEASING.md)[Интеграция шлюза Kilo](../design/kilo-gateway-integration.md)

---