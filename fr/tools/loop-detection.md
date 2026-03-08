

  Outils intégrés

  
# Détection de boucle d'outils

OpenClaw peut empêcher les agents de rester bloqués dans des schémas répétitifs d'appels d'outils. Cette protection est **désactivée par défaut**. Activez-la uniquement là où c'est nécessaire, car elle peut bloquer des appels répétés légitimes avec des paramètres stricts.

## Pourquoi cela existe

-   Détecter les séquences répétitives qui ne font pas progresser l'agent.
-   Détecter les boucles à haute fréquence sans résultat (même outil, mêmes entrées, erreurs répétées).
-   Détecter des schémas spécifiques d'appels répétés pour des outils de scrutation connus.

## Bloc de configuration

Valeurs par défaut globales :

```json
{
  tools: {
    loopDetection: {
      enabled: false,
      historySize: 30,
      warningThreshold: 10,
      criticalThreshold: 20,
      globalCircuitBreakerThreshold: 30,
      detectors: {
        genericRepeat: true,
        knownPollNoProgress: true,
        pingPong: true,
      },
    },
  },
}
```

Surcharge par agent (optionnelle) :

```json
{
  agents: {
    list: [
      {
        id: "safe-runner",
        tools: {
          loopDetection: {
            enabled: true,
            warningThreshold: 8,
            criticalThreshold: 16,
          },
        },
      },
    ],
  },
}
```

### Comportement des champs

-   `enabled` : Interrupteur principal. `false` signifie qu'aucune détection de boucle n'est effectuée.
-   `historySize` : nombre d'appels d'outils récents conservés pour l'analyse.
-   `warningThreshold` : seuil avant de classer un schéma comme "avertissement uniquement".
-   `criticalThreshold` : seuil pour bloquer les schémas de boucle répétitifs.
-   `globalCircuitBreakerThreshold` : seuil global du disjoncteur "pas de progression".
-   `detectors.genericRepeat` : détecte les schémas répétés de même outil + mêmes paramètres.
-   `detectors.knownPollNoProgress` : détecte les schémas de type scrutation connus sans changement d'état.
-   `detectors.pingPong` : détecte les schémas alternés de type ping-pong.

## Configuration recommandée

-   Commencez avec `enabled: true`, les valeurs par défaut inchangées.
-   Gardez les seuils ordonnés comme suit : `warningThreshold < criticalThreshold < globalCircuitBreakerThreshold`.
-   Si des faux positifs se produisent :
    -   augmentez `warningThreshold` et/ou `criticalThreshold`
    -   (optionnellement) augmentez `globalCircuitBreakerThreshold`
    -   désactivez uniquement le détecteur causant les problèmes
    -   réduisez `historySize` pour un contexte historique moins strict

## Journaux et comportement attendu

Lorsqu'une boucle est détectée, OpenClaw signale un événement de boucle et bloque ou atténue le prochain cycle d'outil en fonction de la gravité. Cela protège les utilisateurs d'une consommation incontrôlée de tokens et de blocages tout en préservant l'accès normal aux outils.

-   Privilégiez d'abord l'avertissement et la suppression temporaire.
-   Passez à l'étape supérieure uniquement lorsque des preuves répétées s'accumulent.

## Notes

-   `tools.loopDetection` est fusionné avec les surcharges au niveau de l'agent.
-   La configuration par agent remplace ou étend entièrement les valeurs globales.
-   Si aucune configuration n'existe, les garde-fous restent désactivés.

[Lobster](./lobster.md)[Reactions](./reactions.md)

---