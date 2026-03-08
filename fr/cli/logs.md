title: "Utilisation et exemples de la commande logs de l'interface CLI OpenClaw"
description: "Apprenez à utiliser la commande CLI openclaw logs pour suivre les journaux de fichiers de la passerelle via RPC. Consultez des exemples pour le suivi, la sortie JSON et le fuseau horaire local."
keywords: ["openclaw logs", "cli logs", "journaux de la passerelle", "suivre les journaux", "journalisation rpc", "journaux ligne de commande", "surveillance des journaux"]
---

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