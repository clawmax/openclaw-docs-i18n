

  Expériences

  
# Plan de Supervision de Processus PTY

## 1\. Problème et objectif

Nous avons besoin d'un cycle de vie fiable pour l'exécution de commandes longue durée à travers :

-   les exécutions `exec` au premier plan
-   les exécutions `exec` en arrière-plan
-   les actions de suivi `process` (`poll`, `log`, `send-keys`, `paste`, `submit`, `kill`, `remove`)
-   les sous-processus de l'exécuteur d'agent CLI

L'objectif n'est pas seulement de supporter PTY. L'objectif est d'avoir une propriété, une annulation, un délai d'attente et un nettoyage prévisibles sans heuristiques de correspondance de processus non sécurisées.

## 2\. Périmètre et limites

-   Garder l'implémentation interne dans `src/process/supervisor`.
-   Ne pas créer de nouveau package pour cela.
-   Maintenir la compatibilité avec le comportement actuel lorsque c'est pratique.
-   Ne pas élargir le périmètre à la relecture de terminal ou à la persistance de session de type tmux.

## 3\. Implémenté dans cette branche

### Base du superviseur déjà présente

-   Le module superviseur est en place sous `src/process/supervisor/*`.
-   L'exécution Exec et l'exécuteur CLI sont déjà acheminés via le lancement et l'attente du superviseur.
-   La finalisation du registre est idempotente.

### Cette étape terminée

1.  Contrat de commande PTY explicite

-   `SpawnInput` est maintenant une union discriminée dans `src/process/supervisor/types.ts`.
-   Les exécutions PTY nécessitent `ptyCommand` au lieu de réutiliser le `argv` générique.
-   Le superviseur ne reconstruit plus les chaînes de commande PTY à partir de jointures d'argv dans `src/process/supervisor/supervisor.ts`.
-   L'exécution Exec passe maintenant `ptyCommand` directement dans `src/agents/bash-tools.exec-runtime.ts`.

2.  Découplage des types de la couche processus

-   Les types du superviseur n'importent plus `SessionStdin` des agents.
-   Le contrat stdin local au processus réside dans `src/process/supervisor/types.ts` (`ManagedRunStdin`).
-   Les adaptateurs dépendent maintenant uniquement des types de la couche processus :
    -   `src/process/supervisor/adapters/child.ts`
    -   `src/process/supervisor/adapters/pty.ts`

3.  Amélioration de la propriété du cycle de vie des outils processus

-   `src/agents/bash-tools.process.ts` demande maintenant l'annulation via le superviseur en premier.
-   `process kill/remove` utilise maintenant la terminaison de secours par arborescence de processus lorsque la recherche dans le superviseur échoue.
-   `remove` conserve un comportement de suppression déterministe en supprimant immédiatement les entrées de session en cours d'exécution après que la terminaison a été demandée.

4.  Source unique des valeurs par défaut du watchdog

-   Ajout de valeurs par défaut partagées dans `src/agents/cli-watchdog-defaults.ts`.
-   `src/agents/cli-backends.ts` consomme les valeurs par défaut partagées.
-   `src/agents/cli-runner/reliability.ts` consomme les mêmes valeurs par défaut partagées.

5.  Nettoyage des aides inutilisées

-   Suppression du chemin d'aide `killSession` inutilisé de `src/agents/bash-tools.shared.ts`.

6.  Tests du chemin direct du superviseur ajoutés

-   Ajout de `src/agents/bash-tools.process.supervisor.test.ts` pour couvrir le routage de kill et remove via l'annulation du superviseur.

7.  Corrections des lacunes de fiabilité terminées

-   `src/agents/bash-tools.process.ts` utilise maintenant une terminaison réelle au niveau du système d'exploitation en secours lorsque la recherche dans le superviseur échoue.
-   `src/process/supervisor/adapters/child.ts` utilise maintenant la sémantique de terminaison par arborescence de processus pour les chemins de kill par défaut d'annulation/délai d'attente.
-   Ajout d'un utilitaire d'arborescence de processus partagé dans `src/process/kill-tree.ts`.

8.  Couverture des cas limites du contrat PTY ajoutée

-   Ajout de `src/process/supervisor/supervisor.pty-command.test.ts` pour le transfert littéral de commande PTY et le rejet des commandes vides.
-   Ajout de `src/process/supervisor/adapters/child.test.ts` pour le comportement de kill par arborescence de processus dans l'annulation de l'adaptateur enfant.

## 4\. Lacunes et décisions restantes

### État de la fiabilité

Les deux lacunes de fiabilité requises pour cette étape sont maintenant comblées :

-   `process kill/remove` dispose maintenant d'une terminaison réelle au niveau du système d'exploitation en secours lorsque la recherche dans le superviseur échoue.
-   l'annulation/délai d'attente enfant utilise maintenant la sémantique de kill par arborescence de processus pour le chemin de kill par défaut.
-   Des tests de régression ont été ajoutés pour les deux comportements.

### Durabilité et réconciliation au démarrage

Le comportement au redémarrage est maintenant explicitement défini comme un cycle de vie uniquement en mémoire.

-   `reconcileOrphans()` reste une opération sans effet dans `src/process/supervisor/supervisor.ts` par conception.
-   Les exécutions actives ne sont pas récupérées après un redémarrage du processus.
-   Cette limite est intentionnelle pour cette étape d'implémentation afin d'éviter les risques de persistance partielle.

### Suivis de maintenabilité

1.  `runExecProcess` dans `src/agents/bash-tools.exec-runtime.ts` gère encore plusieurs responsabilités et peut être divisé en aides ciblées lors d'un suivi.

## 5\. Plan d'implémentation

L'étape d'implémentation pour les éléments de fiabilité et de contrat requis est terminée. Terminé :

-   la terminaison réelle de secours pour `process kill/remove`
-   l'annulation par arborescence de processus pour le chemin de kill par défaut de l'adaptateur enfant
-   les tests de régression pour le kill de secours et le chemin de kill de l'adaptateur enfant
-   les tests des cas limites de commande PTY sous `ptyCommand` explicite
-   la limite explicite de redémarrage en mémoire avec `reconcileOrphans()` sans effet par conception

Suivi optionnel :

-   diviser `runExecProcess` en aides ciblées sans dérive de comportement

## 6\. Carte des fichiers

### Superviseur de processus

-   `src/process/supervisor/types.ts` mis à jour avec l'entrée de lancement discriminée et le contrat stdin local au processus.
-   `src/process/supervisor/supervisor.ts` mis à jour pour utiliser `ptyCommand` explicitement.
-   `src/process/supervisor/adapters/child.ts` et `src/process/supervisor/adapters/pty.ts` découplés des types d'agents.
-   `src/process/supervisor/registry.ts` finalisation idempotente inchangée et conservée.

### Intégration Exec et processus

-   `src/agents/bash-tools.exec-runtime.ts` mis à jour pour passer la commande PTY explicitement et conserver le chemin de secours.
-   `src/agents/bash-tools.process.ts` mis à jour pour annuler via le superviseur avec une terminaison de secours réelle par arborescence de processus.
-   `src/agents/bash-tools.shared.ts` chemin d'aide de kill direct supprimé.

### Fiabilité CLI

-   `src/agents/cli-watchdog-defaults.ts` ajouté comme base partagée.
-   `src/agents/cli-backends.ts` et `src/agents/cli-runner/reliability.ts` consomment maintenant les mêmes valeurs par défaut.

## 7\. Exécution de validation dans cette étape

Tests unitaires :

-   `pnpm vitest src/process/supervisor/registry.test.ts`
-   `pnpm vitest src/process/supervisor/supervisor.test.ts`
-   `pnpm vitest src/process/supervisor/supervisor.pty-command.test.ts`
-   `pnpm vitest src/process/supervisor/adapters/child.test.ts`
-   `pnpm vitest src/agents/cli-backends.test.ts`
-   `pnpm vitest src/agents/bash-tools.exec.pty-cleanup.test.ts`
-   `pnpm vitest src/agents/bash-tools.process.poll-timeout.test.ts`
-   `pnpm vitest src/agents/bash-tools.process.supervisor.test.ts`
-   `pnpm vitest src/process/exec.test.ts`

Cibles E2E :

-   `pnpm vitest src/agents/cli-runner.test.ts`
-   `pnpm vitest run src/agents/bash-tools.exec.pty-fallback.test.ts src/agents/bash-tools.exec.background-abort.test.ts src/agents/bash-tools.process.send-keys.test.ts`

Note de vérification de type :

-   Utiliser `pnpm build` (et `pnpm check` pour la porte complète de lint/docs) dans ce dépôt. Les notes plus anciennes mentionnant `pnpm tsgo` sont obsolètes.

## 8\. Garanties opérationnelles préservées

-   Le comportement de durcissement de l'environnement Exec est inchangé.
-   Le flux d'approbation et de liste autorisée est inchangé.
-   L'assainissement des sorties et les plafonds de sortie sont inchangés.
-   L'adaptateur PTY garantit toujours le règlement de l'attente sur un kill forcé et la suppression des écouteurs.

## 9\. Définition de terminé

1.  Le superviseur est le propriétaire du cycle de vie pour les exécutions gérées.
2.  Le lancement PTY utilise un contrat de commande explicite sans reconstruction d'argv.
3.  La couche processus n'a pas de dépendance de type sur la couche agent pour les contrats stdin du superviseur.
4.  Les valeurs par défaut du watchdog proviennent d'une source unique.
5.  Les tests unitaires et e2e ciblés restent verts.
6.  La limite de durabilité au redémarrage est explicitement documentée ou entièrement implémentée.

## 10\. Résumé

La branche a maintenant une forme de supervision cohérente et plus sûre :

-   contrat PTY explicite
-   couche processus plus propre
-   chemin d'annulation piloté par le superviseur pour les opérations sur les processus
-   terminaison réelle de secours lorsque la recherche dans le superviseur échoue
-   annulation par arborescence de processus pour les chemins de kill par défaut des exécutions enfant
-   valeurs par défaut du watchdog unifiées
-   limite explicite de redémarrage en mémoire (pas de réconciliation d'orphelins au redémarrage dans cette étape)

[Plan de Passerelle OpenResponses](./openresponses-gateway.md)[Plan de Liaison de Session Indépendant du Canal](./session-binding-channel-agnostic.md)