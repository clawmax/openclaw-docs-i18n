title: "Guide et exemples de la commande de configuration d'OpenClaw CLI"
description: "Apprenez à initialiser la configuration de l'interface CLI OpenClaw et l'espace de travail de l'agent à l'aide de la commande setup, avec des exemples et les options de l'assistant."
keywords: ["configuration openclaw", "configuration cli", "espace de travail agent", "openclaw.json", "configuration ligne de commande", "assistant d'intégration", "initialisation cli", "configuration espace de travail"]
---

  Commandes CLI

  
# setup

Initialisez `~/.openclaw/openclaw.json` et l'espace de travail de l'agent. Liens utiles :

-   Premiers pas : [Premiers pas](../start/getting-started.md)
-   Assistant : [Intégration](../start/onboarding.md)

## Exemples

```bash
openclaw setup
openclaw setup --workspace ~/.openclaw/workspace
```

Pour exécuter l'assistant via setup :

```bash
openclaw setup --wizard
```

[sessions](./sessions.md)[compétences](./skills.md)

---