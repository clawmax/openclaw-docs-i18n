

  Commandes CLI

  
# voicecall

`voicecall` est une commande fournie par un plugin. Elle n'apparaît que si le plugin voice-call est installé et activé. Documentation principale :

-   Plugin voice-call : [Voice Call](../plugins/voice-call.md)

## Commandes courantes

```bash
openclaw voicecall status --call-id <id>
openclaw voicecall call --to "+15555550123" --message "Hello" --mode notify
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall end --call-id <id>
```

## Exposer des webhooks (Tailscale)

```bash
openclaw voicecall expose --mode serve
openclaw voicecall expose --mode funnel
openclaw voicecall expose --mode off
```

Note de sécurité : n'exposez le point de terminaison webhook qu'à des réseaux de confiance. Préférez Tailscale Serve à Funnel lorsque c'est possible.

[mettre à jour](./update.md)[webhooks](./webhooks.md)

---