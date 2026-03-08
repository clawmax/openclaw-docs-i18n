

  Команды CLI

  
# completion

Сгенерируйте скрипты автодополнения для оболочки и, при желании, установите их в профиль вашей оболочки.

## Использование

```bash
openclaw completion
openclaw completion --shell zsh
openclaw completion --install
openclaw completion --shell fish --install
openclaw completion --write-state
openclaw completion --shell bash --write-state
```

## Опции

-   `-s, --shell `: целевая оболочка (`zsh`, `bash`, `powershell`, `fish`; по умолчанию: `zsh`)
-   `-i, --install`: установить автодополнение, добавив строку source в профиль вашей оболочки
-   `--write-state`: записать скрипт(ы) автодополнения в `$OPENCLAW_STATE_DIR/completions` без вывода в stdout
-   `-y, --yes`: пропустить подтверждающие запросы при установке

## Примечания

-   `--install` записывает небольшой блок «OpenClaw Completion» в ваш профиль оболочки и указывает на кэшированный скрипт.
-   Без `--install` или `--write-state` команда выводит скрипт в stdout.
-   Генерация автодополнения активно загружает деревья команд, поэтому вложенные подкоманды включаются.

[clawbot](./clawbot.md)[config](./config.md)