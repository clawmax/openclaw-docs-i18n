title: "Documentation du pipeline CI OpenClaw et aperçu des jobs"
description: "Découvrez le pipeline CI OpenClaw, son smart scoping, les objectifs des jobs et les commandes locales équivalentes pour les vérifications, tests et releases."
keywords: ["pipeline ci", "intégration continue", "smart scoping", "jobs ci", "commandes de développement local", "pnpm check", "runner blacksmith", "workflow de contribution"]
---

  Contribuer

  
# Pipeline CI

La CI s'exécute à chaque push sur `main` et à chaque pull request. Elle utilise le smart scoping pour ignorer les jobs coûteux lorsque seuls la documentation ou le code natif ont changé.

## Aperçu des jobs

| Job | Objectif | Quand il s'exécute |
| --- | --- | --- |
| `docs-scope` | Détecter les changements uniquement sur la documentation | Toujours |
| `changed-scope` | Détecter quelles zones ont changé (node/macos/android/windows) | PRs non-documentation |
| `check` | Types TypeScript, lint, format | Push sur `main`, ou PRs avec changements pertinents pour Node |
| `check-docs` | Lint Markdown + vérification des liens cassés | Documentation modifiée |
| `code-analysis` | Vérification du seuil de lignes de code (1000 lignes) | PRs uniquement |
| `secrets` | Détecter les secrets divulgués | Toujours |
| `build-artifacts` | Construire dist une fois, partager avec les autres jobs | Non-documentation, changements node |
| `release-check` | Valider le contenu du pack npm | Après la construction |
| `checks` | Tests Node/Bun + vérification du protocole | Non-documentation, changements node |
| `checks-windows` | Tests spécifiques à Windows | Non-documentation, changements pertinents pour Windows |
| `macos` | Lint/build/test Swift + tests TS | PRs avec changements macos |
| `android` | Construction Gradle + tests | Non-documentation, changements android |

## Ordre Fail-Fast

Les jobs sont ordonnés pour que les vérifications peu coûteuses échouent avant que les plus coûteuses ne s'exécutent :

1.  `docs-scope` + `code-analysis` + `check` (parallèles, ~1-2 min)
2.  `build-artifacts` (bloqué par les précédents)
3.  `checks`, `checks-windows`, `macos`, `android` (bloqués par la construction)

La logique de scope se trouve dans `scripts/ci-changed-scope.mjs` et est couverte par les tests unitaires dans `src/scripts/ci-changed-scope.test.ts`.

## Runners

| Runner | Jobs |
| --- | --- |
| `blacksmith-16vcpu-ubuntu-2404` | La plupart des jobs Linux, y compris la détection de scope |
| `blacksmith-32vcpu-windows-2025` | `checks-windows` |
| `macos-latest` | `macos`, `ios` |

## Équivalents Locaux

```bash
pnpm check          # types + lint + format
pnpm test           # tests vitest
pnpm check:docs     # format + lint docs + liens cassés
pnpm release:check  # valider le pack npm
```

[Workflow de développement Pi](./pi-dev.md)[Hubs de documentation](./start/hubs.md)

---