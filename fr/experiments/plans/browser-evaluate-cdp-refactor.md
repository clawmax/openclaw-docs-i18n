

  Expériences

  
# Refactorisation CDP pour Browser Evaluate

## Contexte

`act:evaluate` exécute le JavaScript fourni par l'utilisateur dans la page. Actuellement, il s'exécute via Playwright (`page.evaluate` ou `locator.evaluate`). Playwright sérialise les commandes CDP par page, donc une évaluation bloquée ou de longue durée peut blouer la file d'attente des commandes de la page et donner l'impression que chaque action ultérieure sur cet onglet est "bloquée". La PR #13498 ajoute un filet de sécurité pragmatique (évaluation bornée, propagation de l'abandon et récupération au mieux). Ce document décrit une refactorisation plus importante qui rend `act:evaluate` intrinsèquement isolé de Playwright, de sorte qu'une évaluation bloquée ne puisse pas entraver les opérations normales de Playwright.

## Objectifs

-   `act:evaluate` ne peut pas bloquer définitivement les actions ultérieures du navigateur sur le même onglet.
-   Les délais d'attente sont une source unique de vérité de bout en bout, permettant à l'appelant de s'appuyer sur un budget.
-   L'abandon et le délai d'attente sont traités de la même manière pour les dispatchs HTTP et en processus.
-   Le ciblage d'élément pour l'évaluation est pris en charge sans tout basculer hors de Playwright.
-   Maintenir la compatibilité ascendante pour les appelants et les charges utiles existants.

## Non-objectifs

-   Remplacer toutes les actions du navigateur (clic, saisie, attente, etc.) par des implémentations CDP.
-   Supprimer le filet de sécurité existant introduit dans la PR #13498 (il reste une solution de secours utile).
-   Introduire de nouvelles capacités non sécurisées au-delà de la porte existante `browser.evaluateEnabled`.
-   Ajouter une isolation de processus (processus/tâche de travail) pour l'évaluation. Si nous observons toujours des états de blocage difficiles à récupérer après cette refactorisation, c'est une idée pour la suite.

## Architecture actuelle (Pourquoi ça se bloque)

À un haut niveau :

-   Les appelants envoient `act:evaluate` au service de contrôle du navigateur.
-   Le gestionnaire de route appelle Playwright pour exécuter le JavaScript.
-   Playwright sérialise les commandes de page, donc une évaluation qui ne se termine jamais bloque la file d'attente.
-   Une file d'attente bloquée signifie que les opérations ultérieures de clic/saisie/attente sur l'onglet peuvent sembler planter.

## Architecture proposée

### 1. Propagation de l'échéance

Introduire un concept unique de budget et en dériver tout le reste :

-   L'appelant définit `timeoutMs` (ou une échéance dans le futur).
-   Le délai d'attente de la requête externe, la logique du gestionnaire de route et le budget d'exécution à l'intérieur de la page utilisent tous le même budget, avec une petite marge là où nécessaire pour la surcharge de sérialisation.
-   L'abandon est propagé sous forme de `AbortSignal` partout pour que l'annulation soit cohérente.

Direction d'implémentation :

-   Ajouter un petit assistant (par exemple `createBudget({ timeoutMs, signal })`) qui retourne :
    -   `signal` : le AbortSignal lié
    -   `deadlineAtMs` : échéance absolue
    -   `remainingMs()` : budget restant pour les opérations enfants
-   Utiliser cet assistant dans :
    -   `src/browser/client-fetch.ts` (dispatch HTTP et en processus)
    -   `src/node-host/runner.ts` (chemin proxy)
    -   les implémentations d'actions du navigateur (Playwright et CDP)

### 2. Moteur d'évaluation séparé (Chemin CDP)

Ajouter une implémentation d'évaluation basée sur CDP qui ne partage pas la file d'attente de commandes par page de Playwright. La propriété clé est que le transport d'évaluation est une connexion WebSocket séparée et une session CDP séparée attachée à la cible. Direction d'implémentation :

-   Nouveau module, par exemple `src/browser/cdp-evaluate.ts`, qui :
    -   Se connecte au point de terminaison CDP configuré (socket au niveau du navigateur).
    -   Utilise `Target.attachToTarget({ targetId, flatten: true })` pour obtenir un `sessionId`.
    -   Exécute soit :
        -   `Runtime.evaluate` pour l'évaluation au niveau de la page, soit
        -   `DOM.resolveNode` plus `Runtime.callFunctionOn` pour l'évaluation d'élément.
    -   En cas de délai d'attente ou d'abandon :
        -   Envoie `Runtime.terminateExecution` au mieux pour la session.
        -   Ferme la WebSocket et retourne une erreur claire.

Notes :

-   Cela exécute toujours du JavaScript dans la page, donc la terminaison peut avoir des effets secondaires. L'avantage est que cela ne bloque pas la file d'attente Playwright, et c'est annulable au niveau de la couche transport en tuant la session CDP.

### 3. Réf Story (Ciblage d'élément sans réécriture complète)

La partie difficile est le ciblage d'élément. CDP a besoin d'un handle DOM ou d'un `backendDOMNodeId`, alors qu'aujourd'hui la plupart des actions du navigateur utilisent des localisateurs Playwright basés sur des refs provenant d'instantanés. Approche recommandée : conserver les refs existants, mais y attacher un identifiant CDP résoluble optionnel.

#### 3.1 Étendre les informations de ref stockées

Étendre les métadonnées de ref de rôle stockées pour inclure optionnellement un identifiant CDP :

-   Aujourd'hui : `{ role, name, nth }`
-   Proposé : `{ role, name, nth, backendDOMNodeId?: number }`

Cela maintient toutes les actions existantes basées sur Playwright fonctionnelles et permet à l'évaluation CDP d'accepter la même valeur `ref` lorsque le `backendDOMNodeId` est disponible.

#### 3.2 Remplir backendDOMNodeId au moment de l'instantané

Lors de la production d'un instantané de rôle :

1.  Générer la carte de ref de rôle existante comme aujourd'hui (role, name, nth).
2.  Récupérer l'arbre AX via CDP (`Accessibility.getFullAXTree`) et calculer une carte parallèle de `(role, name, nth) -> backendDOMNodeId` en utilisant les mêmes règles de gestion des doublons.
3.  Fusionner l'identifiant dans les informations de ref stockées pour l'onglet courant.

Si la correspondance échoue pour une ref, laisser `backendDOMNodeId` non défini. Cela rend la fonctionnalité au mieux et sûre à déployer.

#### 3.3 Comportement de l'évaluation avec Ref

Dans `act:evaluate` :

-   Si `ref` est présent et a un `backendDOMNodeId`, exécuter l'évaluation d'élément via CDP.
-   Si `ref` est présent mais n'a pas de `backendDOMNodeId`, revenir au chemin Playwright (avec le filet de sécurité).

Échappatoire optionnelle :

-   Étendre la forme de la requête pour accepter `backendDOMNodeId` directement pour les appelants avancés (et pour le débogage), tout en gardant `ref` comme interface principale.

### 4. Conserver un chemin de récupération de dernier recours

Même avec l'évaluation CDP, il existe d'autres moyens de bloquer un onglet ou une connexion. Conserver les mécanismes de récupération existants (terminer l'exécution + déconnecter Playwright) comme dernier recours pour :

-   les appelants hérités
-   les environnements où l'attache CDP est bloquée
-   les cas limites inattendus de Playwright

## Plan d'implémentation (Itération unique)

### Livrables

-   Un moteur d'évaluation basé sur CDP qui s'exécute en dehors de la file d'attente de commandes par page de Playwright.
-   Un budget unique de délai d'attente/abandon de bout en bout utilisé de manière cohérente par les appelants et les gestionnaires.
-   Des métadonnées de ref qui peuvent optionnellement porter un `backendDOMNodeId` pour l'évaluation d'élément.
-   `act:evaluate` préfère le moteur CDP lorsque possible et revient à Playwright sinon.
-   Des tests qui prouvent qu'une évaluation bloquée ne bloque pas les actions ultérieures.
-   Des journaux/métriques qui rendent visibles les échecs et les retours en arrière.

### Liste de contrôle d'implémentation

1.  Ajouter un assistant "budget" partagé pour lier `timeoutMs` + `AbortSignal` amont en :
    -   un unique `AbortSignal`
    -   une échéance absolue
    -   un assistant `remainingMs()` pour les opérations en aval
2.  Mettre à jour tous les chemins d'appelant pour utiliser cet assistant afin que `timeoutMs` signifie la même chose partout :
    -   `src/browser/client-fetch.ts` (dispatch HTTP et en processus)
    -   `src/node-host/runner.ts` (chemin proxy node)
    -   Les wrappers CLI qui appellent `/act` (ajouter `--timeout-ms` à `browser evaluate`)
3.  Implémenter `src/browser/cdp-evaluate.ts` :
    -   se connecter au socket CDP au niveau du navigateur
    -   `Target.attachToTarget` pour obtenir un `sessionId`
    -   exécuter `Runtime.evaluate` pour l'évaluation de page
    -   exécuter `DOM.resolveNode` + `Runtime.callFunctionOn` pour l'évaluation d'élément
    -   en cas de délai d'attente/abandon : `Runtime.terminateExecution` au mieux puis fermer le socket
4.  Étendre les métadonnées de ref de rôle stockées pour inclure optionnellement `backendDOMNodeId` :
    -   conserver le comportement existant `{ role, name, nth }` pour les actions Playwright
    -   ajouter `backendDOMNodeId?: number` pour le ciblage d'élément CDP
5.  Remplir `backendDOMNodeId` lors de la création d'instantané (au mieux) :
    -   récupérer l'arbre AX via CDP (`Accessibility.getFullAXTree`)
    -   calculer `(role, name, nth) -> backendDOMNodeId` et fusionner dans la carte de ref stockée
    -   si la correspondance est ambiguë ou manquante, laisser l'identifiant non défini
6.  Mettre à jour le routage `act:evaluate` :
    -   si pas de `ref` : toujours utiliser l'évaluation CDP
    -   si `ref` se résout en un `backendDOMNodeId` : utiliser l'évaluation d'élément CDP
    -   sinon : revenir à l'évaluation Playwright (toujours bornée et annulable)
7.  Conserver le chemin de récupération "dernier recours" existant comme solution de secours, pas comme chemin par défaut.
8.  Ajouter des tests :
    -   une évaluation bloquée expire dans le budget et le clic/saisie suivant réussit
    -   l'abandon annule l'évaluation (déconnexion client ou délai d'attente) et débloque les actions ultérieures
    -   les échecs de correspondance reviennent proprement à Playwright
9.  Ajouter de l'observabilité :
    -   compteurs de durée d'évaluation et de délai d'attente
    -   utilisation de terminateExecution
    -   taux de retour en arrière (CDP -> Playwright) et raisons

### Critères d'acceptation

-   Un `act:evaluate` délibérément bloqué retourne dans le budget de l'appelant et ne bloque pas l'onglet pour les actions ultérieures.
-   `timeoutMs` se comporte de manière cohérente à travers la CLI, l'outil agent, le proxy node et les appels en processus.
-   Si `ref` peut être mappé à `backendDOMNodeId`, l'évaluation d'élément utilise CDP ; sinon, le chemin de secours reste borné et récupérable.

## Plan de test

-   Tests unitaires :
    -   logique de correspondance `(role, name, nth)` entre les refs de rôle et les nœuds de l'arbre AX.
    -   comportement de l'assistant budget (marge, calcul du temps restant).
-   Tests d'intégration :
    -   le délai d'attente de l'évaluation CDP retourne dans le budget et ne bloque pas l'action suivante.
    -   l'abandon annule l'évaluation et déclenche la terminaison au mieux.
-   Tests de contrat :
    -   S'assurer que `BrowserActRequest` et `BrowserActResponse` restent compatibles.

## Risques et atténuations

-   Le mappage est imparfait :
    -   Atténuation : mappage au mieux, retour à l'évaluation Playwright, et ajout d'outils de débogage.
-   `Runtime.terminateExecution` a des effets secondaires :
    -   Atténuation : l'utiliser uniquement en cas de délai d'attente/abandon et documenter le comportement dans les erreurs.
-   Surcharge supplémentaire :
    -   Atténuation : ne récupérer l'arbre AX que lorsque les instantanés sont demandés, mettre en cache par cible, et garder la session CDP de courte durée.
-   Limitations du relais d'extension :
    -   Atténuation : utiliser les API d'attache au niveau du navigateur lorsque les sockets par page ne sont pas disponibles, et conserver le chemin Playwright actuel comme solution de secours.

## Questions ouvertes

-   Le nouveau moteur doit-il être configurable comme `playwright`, `cdp` ou `auto` ?
-   Voulons-nous exposer un nouveau format "nodeRef" pour les utilisateurs avancés, ou garder uniquement `ref` ?
-   Comment les instantanés de cadre et les instantanés délimités par sélecteur doivent-ils participer au mappage AX ?

[Plan de refactorisation de streaming unifié pour Runtime](./acp-unified-streaming-refactor.md)[Plan de passerelle OpenResponses](./openresponses-gateway.md)