title: "Guide de référence sur l'utilisation et les coûts des tokens OpenClaw"
description: "Apprenez comment OpenClaw suit les tokens, estime les coûts et gère le contexte. Découvrez les commandes pour surveiller l'utilisation et les stratégies pour réduire la consommation de tokens."
keywords: ["tokens openclaw", "utilisation des tokens", "estimation des coûts", "fenêtre de contexte", "mise en cache des prompts", "cache ttl", "réduire l'utilisation des tokens", "prompt système"]
---

  Référence technique

  
# Utilisation et coûts des tokens

OpenClaw suit les **tokens**, pas les caractères. Les tokens sont spécifiques au modèle, mais la plupart des modèles de type OpenAI comptent en moyenne ~4 caractères par token pour le texte anglais.

## Comment le prompt système est construit

OpenClaw assemble son propre prompt système à chaque exécution. Il inclut :

-   La liste des outils + leurs descriptions courtes
-   La liste des compétences (uniquement les métadonnées ; les instructions sont chargées à la demande avec `read`)
-   Les instructions de mise à jour automatique
-   L'espace de travail + les fichiers de démarrage (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md` lorsqu'ils sont nouveaux, plus `MEMORY.md` et/ou `memory.md` lorsqu'ils sont présents). Les fichiers volumineux sont tronqués par `agents.defaults.bootstrapMaxChars` (par défaut : 20000), et l'injection totale de démarrage est limitée par `agents.defaults.bootstrapTotalMaxChars` (par défaut : 150000). Les fichiers `memory/*.md` sont chargés à la demande via les outils de mémoire et ne sont pas injectés automatiquement.
-   L'heure (UTC + fuseau horaire de l'utilisateur)
-   Les balises de réponse + le comportement du heartbeat
-   Les métadonnées d'exécution (hôte/OS/modèle/réflexion)

Voir le détail complet dans [Prompt système](../concepts/system-prompt.md).

## Ce qui compte dans la fenêtre de contexte

Tout ce que le modèle reçoit compte dans la limite de contexte :

-   Le prompt système (toutes les sections listées ci-dessus)
-   L'historique de la conversation (messages utilisateur + assistant)
-   Les appels d'outils et leurs résultats
-   Les pièces jointes/transcriptions (images, audio, fichiers)
-   Les résumés de compactage et les artefacts d'élagage
-   Les en-têtes de sécurité ou wrappers du fournisseur (non visibles, mais comptabilisés)

Pour les images, OpenClaw réduit la taille des charges utiles d'image de transcription/outil avant les appels au fournisseur. Utilisez `agents.defaults.imageMaxDimensionPx` (par défaut : `1200`) pour ajuster ceci :

-   Des valeurs plus basses réduisent généralement l'utilisation de tokens de vision et la taille de la charge utile.
-   Des valeurs plus élevées préservent plus de détails visuels pour les captures d'écran riches en OCR/interface utilisateur.

Pour une analyse pratique (par fichier injecté, outils, compétences et taille du prompt système), utilisez `/context list` ou `/context detail`. Voir [Contexte](../concepts/context.md).

## Comment voir l'utilisation actuelle des tokens

Utilisez ces commandes dans le chat :

-   `/status` → affiche une **carte de statut riche en emojis** avec le modèle de session, l'utilisation du contexte, les tokens d'entrée/sortie de la dernière réponse, et le **coût estimé** (clé API uniquement).
-   `/usage off|tokens|full` → ajoute un **pied de page d'utilisation par réponse** à chaque réponse.
    -   Persiste par session (stocké sous `responseUsage`).
    -   L'authentification OAuth **masque le coût** (tokens uniquement).
-   `/usage cost` → affiche un résumé local des coûts à partir des journaux de session OpenClaw.

Autres surfaces :

-   **TUI/Web TUI :** `/status` + `/usage` sont pris en charge.
-   **CLI :** `openclaw status --usage` et `openclaw channels list` affichent les fenêtres de quota du fournisseur (pas les coûts par réponse).

## Estimation des coûts (quand affichée)

Les coûts sont estimés à partir de votre configuration tarifaire des modèles :

```
models.providers.<provider>.models[].cost
```

Ce sont des **USD par 1M de tokens** pour `input`, `output`, `cacheRead`, et `cacheWrite`. Si la tarification est manquante, OpenClaw n'affiche que les tokens. Les tokens OAuth n'affichent jamais de coût en dollars.

## Cache TTL et impact de l'élagage

La mise en cache des prompts par le fournisseur ne s'applique que dans la fenêtre TTL du cache. OpenClaw peut optionnellement exécuter un **élagage basé sur le TTL du cache** : il élague la session une fois que le TTL du cache a expiré, puis réinitialise la fenêtre de cache afin que les requêtes suivantes puissent réutiliser le contexte fraîchement mis en cache au lieu de remettre en cache l'historique complet. Cela maintient les coûts d'écriture en cache plus bas lorsqu'une session reste inactive au-delà du TTL. Configurez-le dans [Configuration de la passerelle](../gateway/configuration.md) et consultez les détails du comportement dans [Élagage de session](../concepts/session-pruning.md). Le heartbeat peut maintenir le cache **actif** pendant les périodes d'inactivité. Si le TTL du cache de votre modèle est de `1h`, définir l'intervalle du heartbeat juste en dessous (par exemple, `55m`) peut éviter de remettre en cache le prompt complet, réduisant ainsi les coûts d'écriture en cache. Dans les configurations multi-agents, vous pouvez conserver une configuration de modèle partagée et ajuster le comportement du cache par agent avec `agents.list[].params.cacheRetention`. Pour un guide complet paramètre par paramètre, voir [Mise en cache des prompts](./prompt-caching.md). Pour la tarification de l'API Anthropic, les lectures du cache sont nettement moins chères que les tokens d'entrée, tandis que les écritures en cache sont facturées avec un multiplicateur plus élevé. Consultez la tarification de la mise en cache des prompts d'Anthropic pour les derniers taux et multiplicateurs TTL : [https://docs.anthropic.com/docs/build-with-claude/prompt-caching](https://docs.anthropic.com/docs/build-with-claude/prompt-caching)

### Exemple : maintenir un cache de 1h actif avec le heartbeat

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-6"
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "long"
    heartbeat:
      every: "55m"
```

### Exemple : trafic mixte avec une stratégie de cache par agent

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-6"
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "long" # ligne de base par défaut pour la plupart des agents
  list:
    - id: "research"
      default: true
      heartbeat:
        every: "55m" # garder le cache long actif pour les sessions profondes
    - id: "alerts"
      params:
        cacheRetention: "none" # éviter les écritures en cache pour les notifications par rafales
```

`agents.list[].params` fusionne par-dessus les `params` du modèle sélectionné, vous pouvez donc remplacer uniquement `cacheRetention` et hériter des autres valeurs par défaut du modèle inchangées.

### Exemple : activer l'en-tête bêta pour le contexte 1M d'Anthropic

La fenêtre de contexte 1M d'Anthropic est actuellement en accès bêta. OpenClaw peut injecter la valeur `anthropic-beta` requise lorsque vous activez `context1m` sur les modèles Opus ou Sonnet pris en charge.

```yaml
agents:
  defaults:
    models:
      "anthropic/claude-opus-4-6":
        params:
          context1m: true
```

Ceci correspond à l'en-tête bêta `context-1m-2025-08-07` d'Anthropic. Cela ne s'applique que lorsque `context1m: true` est défini sur cette entrée de modèle. Prérequis : l'identifiant doit être éligible à l'utilisation du contexte long (facturation par clé API, ou abonnement avec Extra Usage activé). Sinon, Anthropic répond avec `HTTP 429: rate_limit_error: Extra usage is required for long context requests`. Si vous vous authentifiez auprès d'Anthropic avec des tokens OAuth/abonnement (`sk-ant-oat-*`), OpenClaw ignore l'en-tête bêta `context-1m-*` car Anthropic rejette actuellement cette combinaison avec une erreur HTTP 401.

## Conseils pour réduire la pression sur les tokens

-   Utilisez `/compact` pour résumer les longues sessions.
-   Réduisez les sorties volumineuses des outils dans vos flux de travail.
-   Baissez `agents.defaults.imageMaxDimensionPx` pour les sessions riches en captures d'écran.
-   Gardez les descriptions de compétences courtes (la liste des compétences est injectée dans le prompt).
-   Préférez des modèles plus petits pour les travaux exploratoires et verbeux.

Voir [Compétences](../tools/skills.md) pour la formule exacte de surcharge de la liste des compétences.

[Référence de l'Assistant](./wizard.md)[Surface d'identification SecretRef](./secretref-credential-surface.md)