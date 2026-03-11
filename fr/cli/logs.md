

  Commandes CLI

  
# logs

Suivre les journaux de fichiers de la passerelle via RPC (fonctionne en mode distant). Liens connexes :

-   Vue d'ensemble de la journalisation : [Journalisation](../logging.md)

## Exemples

```bash
openclaw logs
openclaw logs --follow
openclaw logs --json
openclaw logs --limit 500
openclaw logs --local-time
openclaw logs --follow --local-time
```

Utilisez `--local-time` pour afficher les horodatages dans votre fuseau horaire local.

[hooks](./hooks.md)[memory](./memory.md)

---