

  Команды CLI

  
# voicecall

`voicecall` — это команда, предоставляемая плагином. Она появляется только если плагин голосовых вызовов установлен и включен. Основная документация:

-   Плагин голосовых вызовов: [Голосовой вызов](../plugins/voice-call.md)

## Распространённые команды

```bash
openclaw voicecall status --call-id <id>
openclaw voicecall call --to "+15555550123" --message "Hello" --mode notify
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall end --call-id <id>
```

## Предоставление вебхуков (Tailscale)

```bash
openclaw voicecall expose --mode serve
openclaw voicecall expose --mode funnel
openclaw voicecall expose --mode off
```

Примечание по безопасности: предоставляйте конечную точку вебхука только доверенным сетям. По возможности используйте Tailscale Serve вместо Funnel.

[update](./update.md)[webhooks](./webhooks.md)

---