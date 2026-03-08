

  Configuration développeur

  
# Flux de travail de développement Pi

Ce guide résume un flux de travail raisonnable pour travailler sur l'intégration Pi dans OpenClaw.

## Vérification des types et Linting

-   Vérification des types et build : `pnpm build`
-   Lint : `pnpm lint`
-   Vérification du formatage : `pnpm format`
-   Contrôle complet avant un push : `pnpm lint && pnpm build && pnpm test`

## Exécution des tests Pi

Exécutez directement l'ensemble de tests axé sur Pi avec Vitest :

```bash
pnpm test -- \
  "src/agents/pi-*.test.ts" \
  "src/agents/pi-embedded-*.test.ts" \
  "src/agents/pi-tools*.test.ts" \
  "src/agents/pi-settings.test.ts" \
  "src/agents/pi-tool-definition-adapter*.test.ts" \
  "src/agents/pi-extensions/**/*.test.ts"
```

Pour inclure l'exercice du fournisseur en direct :

```
OPENCLAW_LIVE_TEST=1 pnpm test -- src/agents/pi-embedded-runner-extraparams.live.test.ts
```

Cela couvre les principales suites de tests unitaires Pi :

-   `src/agents/pi-*.test.ts`
-   `src/agents/pi-embedded-*.test.ts`
-   `src/agents/pi-tools*.test.ts`
-   `src/agents/pi-settings.test.ts`
-   `src/agents/pi-tool-definition-adapter.test.ts`
-   `src/agents/pi-extensions/*.test.ts`

## Tests manuels

Flux recommandé :

-   Exécutez la passerelle en mode dev :
    -   `pnpm gateway:dev`
-   Déclenchez l'agent directement :
    -   `pnpm openclaw agent --message "Bonjour" --thinking low`
-   Utilisez l'interface TUI pour le débogage interactif :
    -   `pnpm tui`

Pour le comportement des appels d'outils, demandez une action `read` ou `exec` pour pouvoir voir le streaming des outils et la gestion des charges utiles.

## Réinitialisation complète

L'état se trouve dans le répertoire d'état d'OpenClaw. Par défaut, c'est `~/.openclaw`. Si `OPENCLAW_STATE_DIR` est défini, utilisez ce répertoire à la place. Pour tout réinitialiser :

-   `openclaw.json` pour la configuration
-   `credentials/` pour les profils d'authentification et les jetons
-   `agents//sessions/` pour l'historique des sessions de l'agent
-   `agents//sessions.json` pour l'index des sessions
-   `sessions/` si des chemins hérités existent
-   `workspace/` si vous voulez un espace de travail vierge

Si vous souhaitez uniquement réinitialiser les sessions, supprimez `agents//sessions/` et `agents//sessions.json` pour cet agent. Conservez `credentials/` si vous ne voulez pas vous réauthentifier.

## Références

-   [https://docs.openclaw.ai/testing](https://docs.openclaw.ai/testing)
-   [https://docs.openclaw.ai/start/getting-started](https://docs.openclaw.ai/start/getting-started)

[Configuration](./start/setup.md)[Pipeline CI](./ci.md)

---