

  Команды CLI

  
# pairing

Одобряйте или проверяйте запросы на сопряжение в личных сообщениях (для каналов, поддерживающих сопряжение). Связанные темы:

-   Процесс сопряжения: [Сопряжение](../channels/pairing.md)

## Команды

```bash
openclaw pairing list telegram
openclaw pairing list --channel telegram --account work
openclaw pairing list telegram --json

openclaw pairing approve telegram <code>
openclaw pairing approve --channel telegram --account work <code> --notify
```

## Примечания

-   Ввод канала: передайте его позиционно (`pairing list telegram`) или с помощью `--channel `.
-   `pairing list` поддерживает `--account ` для каналов с несколькими аккаунтами.
-   `pairing approve` поддерживает `--account ` и `--notify`.
-   Если настроен только один канал с поддержкой сопряжения, допустима команда `pairing approve `.

[onboard](./onboard.md)[plugins](./plugins.md)