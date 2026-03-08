

  Sessions et mémoire

  
# Compaction

Chaque modèle a une **fenêtre de contexte** (nombre maximum de tokens qu'il peut voir). Les chats de longue durée accumulent des messages et des résultats d'outils ; une fois la fenêtre saturée, OpenClaw **compacte** l'historique plus ancien pour rester dans les limites.

## Qu'est-ce que la compaction

La compaction **résume la conversation plus ancienne** en une entrée de résumé compact et conserve les messages récents intacts. Le résumé est stocké dans l'historique de la session, donc les requêtes futures utilisent :

-   Le résumé de compaction
-   Les messages récents après le point de compaction

La compaction **persiste** dans l'historique JSONL de la session.

## Configuration

Utilisez le paramètre `agents.defaults.compaction` dans votre `openclaw.json` pour configurer le comportement de compaction (mode, tokens cible, etc.). La condensation de la compaction préserve les identifiants opaques par défaut (`identifierPolicy: "strict"`). Vous pouvez remplacer ce comportement avec `identifierPolicy: "off"` ou fournir un texte personnalisé avec `identifierPolicy: "custom"` et `identifierInstructions`.

## Auto-compaction (activée par défaut)

Lorsqu'une session approche ou dépasse la fenêtre de contexte du modèle, OpenClaw déclenche l'auto-compaction et peut réessayer la requête originale en utilisant le contexte compacté. Vous verrez :

-   `🧹 Auto-compaction terminée` en mode verbeux
-   `/status` affichant `🧹 Compactions : `

Avant la compaction, OpenClaw peut exécuter un **tour de vidage de mémoire silencieux** pour stocker des notes durables sur le disque. Voir [Mémoire](./memory.md) pour les détails et la configuration.

## Compaction manuelle

Utilisez `/compact` (éventuellement avec des instructions) pour forcer un passage de compaction :

```bash
/compact Focus on decisions and open questions
```

## Source de la fenêtre de contexte

La fenêtre de contexte est spécifique au modèle. OpenClaw utilise la définition du modèle provenant du catalogue du fournisseur configuré pour déterminer les limites.

## Compaction vs élagage

-   **Compaction** : résume et **persiste** dans le JSONL.
-   **Élagage de session** : supprime uniquement les anciens **résultats d'outils**, **en mémoire**, par requête.

Voir [/concepts/session-pruning](./session-pruning.md) pour les détails sur l'élagage.

## Compaction côté serveur OpenAI

OpenClaw prend également en charge les indications de compaction côté serveur OpenAI Responses pour les modèles OpenAI directs compatibles. Ceci est distinct de la compaction locale OpenClaw et peut fonctionner conjointement avec elle.

-   Compaction locale : OpenClaw résume et persiste dans le JSONL de la session.
-   Compaction côté serveur : OpenAI compacte le contexte côté fournisseur lorsque `store` + `context_management` sont activés.

Voir [Fournisseur OpenAI](../providers/openai.md) pour les paramètres du modèle et les remplacements.

## Conseils

-   Utilisez `/compact` lorsque les sessions semblent obsolètes ou que le contexte est gonflé.
-   Les grandes sorties d'outils sont déjà tronquées ; l'élagage peut réduire davantage l'accumulation des résultats d'outils.
-   Si vous avez besoin d'une ardoise vierge, `/new` ou `/reset` démarre un nouvel identifiant de session.

[Mémoire](./memory.md)[Routage Multi-Agent](./multi-agent.md)