

  Application compagnon macOS

  
# Barre de menus

## Ce qui est affiché

-   Nous affichons l'état de travail actuel de l'agent dans l'icône de la barre de menus et dans la première ligne de statut du menu.
-   Le statut de santé est masqué pendant qu'un travail est actif ; il réapparaît lorsque toutes les sessions sont inactives.
-   Le bloc "Nœuds" dans le menu liste uniquement les **appareils** (nœuds appairés via `node.list`), pas les entrées client/présence.
-   Une section "Utilisation" apparaît sous Contexte lorsque des instantanés d'utilisation des fournisseurs sont disponibles.

## Modèle d'état

-   Sessions : les événements arrivent avec un `runId` (par exécution) ainsi qu'une `sessionKey` dans la charge utile. La session "principale" est la clé `main` ; si elle est absente, nous revenons à la session mise à jour le plus récemment.
-   Priorité : la session principale l'emporte toujours. Si la session principale est active, son état est affiché immédiatement. Si la session principale est inactive, la session non principale la plus récemment active est affichée. Nous ne basculons pas en plein milieu d'une activité ; nous ne changeons que lorsque la session actuelle devient inactive ou que la session principale devient active.
-   Types d'activité :
    -   `job` : exécution de commande de haut niveau (`state: started|streaming|done|error`).
    -   `tool` : `phase: start|result` avec `toolName` et `meta/args`.

## Enum IconState (Swift)

-   `idle`
-   `workingMain(ActivityKind)`
-   `workingOther(ActivityKind)`
-   `overridden(ActivityKind)` (surcharge de débogage)

### ActivityKind → glyphe

-   `exec` → 💻
-   `read` → 📄
-   `write` → ✍️
-   `edit` → 📝
-   `attach` → 📎
-   par défaut → 🛠️

### Correspondance visuelle

-   `idle` : mascotte normale.
-   `workingMain` : badge avec glyphe, teinte pleine, animation de "travail" de la patte.
-   `workingOther` : badge avec glyphe, teinte atténuée, pas de déplacement rapide.
-   `overridden` : utilise le glyphe/la teinte choisi(e) indépendamment de l'activité.

## Texte de la ligne de statut (menu)

-   Pendant qu'un travail est actif : `<Rôle de la session> · <libellé de l'activité>`
    -   Exemples : `Principal · exec: pnpm test`, `Autre · read: apps/macos/Sources/OpenClaw/AppState.swift`.
-   Lorsqu'inactif : revient au résumé de santé.

## Ingestion d'événements

-   Source : événements `agent` du canal de contrôle (`ControlChannel.handleAgentEvent`).
-   Champs analysés :
    -   `stream: "job"` avec `data.state` pour le début/la fin.
    -   `stream: "tool"` avec `data.phase`, `name`, `meta`/`args` optionnels.
-   Libellés :
    -   `exec` : première ligne de `args.command`.
    -   `read`/`write` : chemin raccourci.
    -   `edit` : chemin plus le type de changement inféré à partir de `meta`/des comptages de diff.
    -   valeur par défaut : nom de l'outil.

## Surcharge de débogage

-   Paramètres ▸ Débogage ▸ Sélecteur "Surcharge d'icône" :
    -   `Système (auto)` (par défaut)
    -   `En travail : principal` (par type d'outil)
    -   `En travail : autre` (par type d'outil)
    -   `Inactif`
-   Stocké via `@AppStorage("iconOverride")` ; mappé sur `IconState.overridden`.

## Liste de vérification pour les tests

-   Déclencher un travail de session principale : vérifier que l'icône bascule immédiatement et que la ligne de statut affiche le libellé "principal".
-   Déclencher un travail de session non principale pendant que la session principale est inactive : l'icône/le statut affiche "non principal" ; reste stable jusqu'à sa fin.
-   Démarrer la session principale pendant qu'une autre est active : l'icône bascule instantanément vers "principal".
-   Rafales rapides d'outils : s'assurer que le badge ne clignote pas (délai de grâce TTL sur les résultats d'outils).
-   La ligne de santé réapparaît une fois toutes les sessions inactives.

[Configuration Dev macOS](./dev-setup.md)[Voice Wake](./voicewake.md)

---