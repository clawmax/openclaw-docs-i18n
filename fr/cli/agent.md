

  Commandes CLI

  
# agent

Exécutez un tour d'agent via la Gateway (utilisez `--local` pour le mode embarqué). Utilisez `--agent ` pour cibler directement un agent configuré. Liens utiles :

-   Outil d'envoi d'agent : [Envoi d'agent](../tools/agent-send.md)

## Exemples

```bash
openclaw agent --to +15555550123 --message "status update" --deliver
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```

## Notes

-   Lorsque cette commande déclenche la régénération de `models.json`, les identifiants des fournisseurs gérés par SecretRef sont persistés sous forme de marqueurs non secrets (par exemple, des noms de variables d'environnement ou `secretref-managed`), et non sous forme de texte clair de secret résolu.

[acp](./acp.md)[agents](./agents.md)

---