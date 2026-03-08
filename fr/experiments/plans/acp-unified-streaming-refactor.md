

  Expériences

  
# Plan de refactorisation du streaming pour un runtime unifié

## Objectif

Fournir un pipeline de streaming partagé pour `main`, `subagent` et `acp` afin que tous les runtimes obtiennent un comportement identique de coalescence, de découpage, d'ordre de livraison et de récupération après incident.

## Pourquoi cela existe

-   Le comportement actuel est réparti entre plusieurs chemins de mise en forme spécifiques aux runtimes.
-   Les bogues de formatage/coalescence peuvent être corrigés dans un chemin mais subsister dans d'autres.
-   La cohérence de la livraison, la suppression des doublons et la sémantique de récupération sont plus difficiles à raisonner.

## Architecture cible

Pipeline unique, adaptateurs spécifiques au runtime :

1.  Les adaptateurs de runtime émettent uniquement des événements canoniques.
2.  L'assembleur de flux partagé coalesce et finalise les événements de texte/outil/statut.
3.  Le projecteur de canal partagé applique le découpage/formatage spécifique au canal une seule fois.
4.  Le registre de livraison partagé applique une sémantique d'envoi/relecture idempotente.
5.  L'adaptateur de canal sortant exécute les envois et enregistre les points de contrôle de livraison.

Contrat d'événement canonique :

-   `turn_started`
-   `text_delta`
-   `block_final`
-   `tool_started`
-   `tool_finished`
-   `status`
-   `turn_completed`
-   `turn_failed`
-   `turn_cancelled`

## Axes de travail

### 1) Contrat de streaming canonique

-   Définir un schéma d'événement strict + validation dans le noyau.
-   Ajouter des tests de contrat d'adaptateur pour garantir que chaque runtime émet des événements compatibles.
-   Rejeter les événements de runtime malformés tôt et fournir des diagnostics structurés.

### 2) Processeur de flux partagé

-   Remplacer la logique de coalesceur/projecteur spécifique au runtime par un seul processeur.
-   Le processeur gère la mise en mémoire tampon des deltas de texte, le vidage inactif, la division en morceaux maximum et le vidage de fin.
-   Déplacer la résolution de configuration ACP/main/subagent dans un seul assistant pour éviter la divergence.

### 3) Projection de canal partagée

-   Garder les adaptateurs de canal simples : accepter les blocs finalisés et envoyer.
-   Déplacer les particularités de découpage spécifiques à Discord uniquement dans le projecteur de canal.
-   Garder le pipeline indépendant du canal avant la projection.

### 4) Registre de livraison + relecture

-   Ajouter des ID de livraison par tour/par morceau.
-   Enregistrer les points de contrôle avant et après l'envoi physique.
-   Au redémarrage, relire les morceaux en attente de manière idempotente et éviter les doublons.

### 5) Migration et basculement

-   Phase 1 : mode ombre (le nouveau pipeline calcule la sortie mais l'ancien chemin envoie ; comparer).
-   Phase 2 : basculement runtime par runtime (`acp`, puis `subagent`, puis `main` ou l'inverse selon le risque).
-   Phase 3 : supprimer l'ancien code de streaming spécifique au runtime.

## Non-objectifs

-   Aucun changement au modèle de politique/autorisations ACP dans cette refactorisation.
-   Aucune expansion de fonctionnalité spécifique au canal en dehors des corrections de compatibilité de projection.
-   Pas de refonte du transport/backend (le contrat de plugin acpx reste tel quel, sauf si nécessaire pour la parité des événements).

## Risques et atténuations

-   Risque : régressions comportementales dans les chemins main/subagent existants. Atténuation : comparaison en mode ombre + tests de contrat d'adaptateur + tests de bout en bout des canaux.
-   Risque : envois en double pendant la récupération après incident. Atténuation : ID de livraison durables + relecture idempotente dans l'adaptateur de livraison.
-   Risque : divergence à nouveau des adaptateurs de runtime. Atténuation : suite de tests de contrat partagé obligatoire pour tous les adaptateurs.

## Critères d'acceptation

-   Tous les runtimes réussissent les tests de contrat de streaming partagé.
-   Discord ACP/main/subagent produisent un comportement équivalent d'espacement/découpage pour les petits deltas.
-   La relecture après incident/redémarrage n'envoie aucun morceau en double pour le même ID de livraison.
-   L'ancien chemin de projecteur/coalesceur ACP est supprimé.
-   La résolution de configuration du streaming est partagée et indépendante du runtime.

[Agents liés au fil ACP](./acp-thread-bound-agents.md)[Refactorisation CDP d'évaluation du navigateur](./browser-evaluate-cdp-refactor.md)