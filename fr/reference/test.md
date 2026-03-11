

  Notes de version

  
# Tests

-   Kit de test complet (suites, live, Docker) : [Testing](../help/testing.md)
-   `pnpm test:force` : Tue tout processus gateway résiduel occupant le port de contrôle par défaut, puis exécute la suite Vitest complète avec un port gateway isolé pour que les tests serveur n'entrent pas en conflit avec une instance en cours d'exécution. Utilisez ceci lorsqu'une exécution gateway précédente a laissé le port 18789 occupé.
-   `pnpm test:coverage` : Exécute la suite unitaire avec la couverture V8 (via `vitest.unit.config.ts`). Les seuils globaux sont de 70% pour les lignes/branches/fonctions/instructions. La couverture exclut les points d'entrée à forte intégration (câblage CLI, ponts gateway/telegram, serveur statique webchat) pour que la cible reste concentrée sur la logique testable unitairement.
-   `pnpm test` sur Node 24+ : OpenClaw désactive automatiquement les `vmForks` de Vitest et utilise `forks` pour éviter `ERR_VM_MODULE_LINK_FAILURE` / `module is already linked`. Vous pouvez forcer le comportement avec `OPENCLAW_TEST_VM_FORKS=0|1`.
-   `pnpm test` : exécute par défaut la voie unitaire principale rapide pour un retour local rapide.
-   `pnpm test:channels` : exécute les suites axées sur les canaux.
-   `pnpm test:extensions` : exécute les suites d'extensions/plugins.
-   Intégration Gateway : opt-in via `OPENCLAW_TEST_INCLUDE_GATEWAY=1 pnpm test` ou `pnpm test:gateway`.
-   `pnpm test:e2e` : Exécute les tests de fumée end-to-end du gateway (appariement multi-instance WS/HTTP/node). Par défaut, utilise `vmForks` + workers adaptatifs dans `vitest.e2e.config.ts` ; ajustez avec `OPENCLAW_E2E_WORKERS=` et définissez `OPENCLAW_E2E_VERBOSE=1` pour des logs verbeux.
-   `pnpm test:live` : Exécute les tests en direct des fournisseurs (minimax/zai). Nécessite des clés API et `LIVE=1` (ou `*_LIVE_TEST=1` spécifique au fournisseur) pour être activés.

## Porte de validation locale pour PR

Pour les vérifications locales de validation/déploiement de PR, exécutez :

-   `pnpm check`
-   `pnpm build`
-   `pnpm test`
-   `pnpm check:docs`

Si `pnpm test` échoue de manière intermittente sur un hôte chargé, relancez une fois avant de le considérer comme une régression, puis isolez avec `pnpm vitest run <chemin/vers/test>`. Pour les hôtes à mémoire limitée, utilisez :

-   `OPENCLAW_TEST_PROFILE=low OPENCLAW_TEST_SERIAL_GATEWAY=1 pnpm test`

## Benchmark de latence modèle (clés locales)

Script : [`scripts/bench-model.ts`](https://github.com/openclaw/openclaw/blob/main/scripts/bench-model.ts) Utilisation :

-   `source ~/.profile && pnpm tsx scripts/bench-model.ts --runs 10`
-   Variables d'env optionnelles : `MINIMAX_API_KEY`, `MINIMAX_BASE_URL`, `MINIMAX_MODEL`, `ANTHROPIC_API_KEY`
-   Prompt par défaut : “Réponds par un seul mot : ok. Pas de ponctuation ni de texte supplémentaire.”

Dernière exécution (2025-12-31, 20 runs) :

-   minimax médiane 1279ms (min 1114, max 2431)
-   opus médiane 2454ms (min 1224, max 3170)

## Benchmark de démarrage CLI

Script : [`scripts/bench-cli-startup.ts`](https://github.com/openclaw/openclaw/blob/main/scripts/bench-cli-startup.ts) Utilisation :

-   `pnpm tsx scripts/bench-cli-startup.ts`
-   `pnpm tsx scripts/bench-cli-startup.ts --runs 12`
-   `pnpm tsx scripts/bench-cli-startup.ts --entry dist/entry.js --timeout-ms 45000`

Ceci benchmarke ces commandes :

-   `--version`
-   `--help`
-   `health --json`
-   `status --json`
-   `status`

La sortie inclut la moyenne, p50, p95, min/max, et la distribution des codes de sortie/signaux pour chaque commande.

## E2E d'intégration (Docker)

Docker est optionnel ; ceci n'est nécessaire que pour les tests de fumée d'intégration conteneurisés. Flux de démarrage à froid complet dans un conteneur Linux propre :

```
scripts/e2e/onboard-docker.sh
```

Ce script pilote l'assistant interactif via un pseudo-tty, vérifie les fichiers de config/workspace/session, puis démarre le gateway et exécute `openclaw health`.

## Test de fumée d'import QR (Docker)

Garantit que `qrcode-terminal` se charge sous Node 22+ dans Docker :

```bash
pnpm test:docker:qr
```

[Checklist de version](./RELEASING.md)[Intégration gateway Kilo](../design/kilo-gateway-integration.md)

---