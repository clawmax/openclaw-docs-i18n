title: "Vue d'ensemble de l'intégration OpenClaw : Assistant CLI et configuration de l'application macOS"
description: "Apprenez à intégrer OpenClaw en utilisant l'assistant CLI ou l'application macOS. Choisissez votre chemin de configuration et configurez des fournisseurs d'IA personnalisés pour votre Gateway."
keywords: ["intégration openclaw", "assistant cli", "configuration application macos", "fournisseur personnalisé", "configuration gateway", "compatible openai", "compatible anthropic", "chemins d'intégration"]
---

  Premières étapes

  
# Vue d'ensemble de l'intégration

OpenClaw prend en charge plusieurs chemins d'intégration selon l'endroit où la Gateway s'exécute et votre préférence de configuration des fournisseurs.

## Choisissez votre chemin d'intégration

-   **Assistant CLI** pour macOS, Linux et Windows (via WSL2).
-   **Application macOS** pour une première exécution guidée sur Mac à puce Apple ou Intel.

## Assistant d'intégration CLI

Exécutez l'assistant dans un terminal :

```bash
openclaw onboard
```

Utilisez l'assistant CLI lorsque vous souhaitez un contrôle total de la Gateway, de l'espace de travail, des canaux et des compétences. Documentation :

-   [Assistant d'intégration (CLI)](./wizard.md)
-   [Commande `openclaw onboard`](../cli/onboard.md)

## Intégration via l'application macOS

Utilisez l'application OpenClaw lorsque vous souhaitez une configuration entièrement guidée sur macOS. Documentation :

-   [Intégration (Application macOS)](./onboarding.md)

## Fournisseur personnalisé

Si vous avez besoin d'un point de terminaison qui n'est pas listé, y compris les fournisseurs hébergés qui exposent des API OpenAI ou Anthropic standard, choisissez **Fournisseur personnalisé** dans l'assistant CLI. Il vous sera demandé de :

-   Choisir Compatible OpenAI, Compatible Anthropic ou **Inconnu** (détection automatique).
-   Saisir une URL de base et une clé API (si requise par le fournisseur).
-   Fournir un identifiant de modèle et un alias optionnel.
-   Choisir un identifiant de point de terminaison pour que plusieurs points de terminaison personnalisés puissent coexister.

Pour des étapes détaillées, suivez la documentation d'intégration CLI ci-dessus.

[Démarrage](./getting-started.md)[Intégration : CLI](./wizard.md)