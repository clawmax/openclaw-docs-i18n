

  Обзор платформ

  
# Приложение для iOS

Доступность: внутренняя предварительная версия. Приложение для iOS пока не распространяется публично.

## Что оно делает

-   Подключается к шлюзу через WebSocket (локальная сеть или tailnet).
-   Предоставляет возможности узла: Canvas, снимок экрана, захват камеры, геолокация, режим разговора, голосовое пробуждение.
-   Получает команды `node.invoke` и отправляет события статуса узла.

## Требования

-   Шлюз, запущенный на другом устройстве (macOS, Linux или Windows через WSL2).
-   Сетевой доступ:
    -   Та же локальная сеть через Bonjour, **или**
    -   Tailnet через unicast DNS-SD (пример домена: `openclaw.internal.`), **или**
    -   Ручной ввод хоста/порта (резервный вариант).

## Быстрый старт (сопряжение + подключение)

1.  Запустите шлюз:

```bash
openclaw gateway --port 18789
```

2.  В приложении iOS откройте Настройки и выберите обнаруженный шлюз (или включите "Ручной хост" и введите хост/порт).
3.  Подтвердите запрос на сопряжение на хосте шлюза:

```bash
openclaw devices list
openclaw devices approve <requestId>
```

4.  Проверьте подключение:

```bash
openclaw nodes status
openclaw gateway call node.list --params "{}"
```

## Методы обнаружения

### Bonjour (локальная сеть)

Шлюз анонсирует `_openclaw-gw._tcp` в домене `local.`. Приложение iOS автоматически отображает эти записи.

### Tailnet (межсетевое подключение)

Если mDNS заблокирован, используйте зону unicast DNS-SD (выберите домен; пример: `openclaw.internal.`) и Split DNS в Tailscale. Подробности см. в [Bonjour](../gateway/bonjour.md) (пример для CoreDNS).

### Ручной хост/порт

В Настройках включите **Ручной хост** и введите хост шлюза и порт (по умолчанию `18789`).

## Canvas + A2UI

Узел iOS отображает холст (WKWebView). Используйте `node.invoke` для управления им:

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.navigate --params '{"url":"http://<gateway-host>:18789/__openclaw__/canvas/"}'
```

Примечания:

-   Хост холста шлюза обслуживает `/__openclaw__/canvas/` и `/__openclaw__/a2ui/`.
-   Он обслуживается HTTP-сервером шлюза (тот же порт, что и `gateway.port`, по умолчанию `18789`).
-   Узел iOS автоматически переходит на A2UI при подключении, если анонсирован URL хоста холста.
-   Вернуться к встроенному шаблону можно с помощью `canvas.navigate` и `{"url":""}`.

### Выполнение кода в Canvas / снимок

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.eval --params '{"javaScript":"(() => { const {ctx} = window.__openclaw; ctx.clearRect(0,0,innerWidth,innerHeight); ctx.lineWidth=6; ctx.strokeStyle=\"#ff2d55\"; ctx.beginPath(); ctx.moveTo(40,40); ctx.lineTo(innerWidth-40, innerHeight-40); ctx.stroke(); return \"ok\"; })()"}'
```

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.snapshot --params '{"maxWidth":900,"format":"jpeg"}'
```

## Голосовое пробуждение + режим разговора

-   Голосовое пробуждение и режим разговора доступны в Настройках.
-   iOS может приостанавливать аудио в фоне; рассматривайте голосовые функции как работающие с максимальным усилием, когда приложение неактивно.

## Распространённые ошибки

-   `NODE_BACKGROUND_UNAVAILABLE`: переведите приложение iOS на передний план (командам canvas/камеры/экрана это требуется).
-   `A2UI_HOST_NOT_CONFIGURED`: шлюз не анонсировал URL хоста холста; проверьте параметр `canvasHost` в [конфигурации шлюза](../gateway/configuration.md).
-   Запрос на сопряжение не появляется: выполните `openclaw devices list` и подтвердите вручную.
-   Не удаётся переподключиться после переустановки: токен сопряжения в Keychain был удалён; выполните повторное сопряжение узла.

## Связанная документация

-   [Сопряжение](../channels/pairing.md)
-   [Обнаружение](../gateway/discovery.md)
-   [Bonjour](../gateway/bonjour.md)

[Приложение для Android](./android.md)[DigitalOcean](./digitalocean.md)