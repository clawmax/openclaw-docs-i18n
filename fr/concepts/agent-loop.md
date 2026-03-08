title: "Comprendre la boucle d'agent dans OpenClaw AI"
description: "Apprenez comment fonctionne la boucle d'agent OpenClaw de bout en bout, de la réception et l'assemblage du contexte à l'inférence du modèle, l'exécution d'outils et les réponses en streaming."
keywords: ["boucle d'agent", "agent openclaw", "exécution d'agent IA", "exécution d'outil", "gestion de session", "réponses en streaming", "cycle de vie de l'agent", "hooks d'agent"]
---

  Fondamentaux

  
# Boucle d'agent

Une boucle agentique est l'exécution complète et "réelle" d'un agent : réception → assemblage du contexte → inférence du modèle → exécution d'outils → réponses en streaming → persistance. C'est le chemin autoritaire qui transforme un message en actions et une réponse finale, tout en maintenant l'état de la session cohérent. Dans OpenClaw, une boucle est une exécution unique et sérialisée par session qui émet des événements de cycle de vie et de flux pendant que le modèle raisonne, appelle des outils et diffuse la sortie. Ce document explique comment cette boucle authentique est câblée de bout en bout.

## Points d'entrée

-   RPC Gateway : `agent` et `agent.wait`.
-   CLI : commande `agent`.

## Fonctionnement (haut niveau)

1.  Le RPC `agent` valide les paramètres, résout la session (sessionKey/sessionId), persiste les métadonnées de la session, et renvoie immédiatement `{ runId, acceptedAt }`.
2.  `agentCommand` exécute l'agent :
    -   résout le modèle et les valeurs par défaut de raisonnement/verbose
    -   charge un instantané des compétences
    -   appelle `runEmbeddedPiAgent` (runtime pi-agent-core)
    -   émet un **événement de cycle de vie end/error** si la boucle embarquée n'en émet pas
3.  `runEmbeddedPiAgent` :
    -   sérialise les exécutions via des files d'attente par session et globales
    -   résout le modèle et le profil d'authentification et construit la session pi
    -   s'abonne aux événements pi et diffuse les deltas assistant/outil
    -   applique un timeout -> annule l'exécution si dépassé
    -   renvoie les payloads et les métadonnées d'utilisation
4.  `subscribeEmbeddedPiSession` fait le pont entre les événements pi-agent-core et le flux `agent` d'OpenClaw :
    -   événements outil => `stream: "tool"`
    -   deltas assistant => `stream: "assistant"`
    -   événements de cycle de vie => `stream: "lifecycle"` (`phase: "start" | "end" | "error"`)
5.  `agent.wait` utilise `waitForAgentJob` :
    -   attend un **événement de cycle de vie end/error** pour le `runId`
    -   renvoie `{ status: ok|error|timeout, startedAt, endedAt, error? }`

## File d'attente + concurrence

-   Les exécutions sont sérialisées par clé de session (voie de session) et optionnellement via une voie globale.
-   Cela empêche les conflits d'accès aux outils/sessions et maintient l'historique de la session cohérent.
-   Les canaux de messagerie peuvent choisir des modes de file d'attente (collect/steer/followup) qui alimentent ce système de voies. Voir [File d'attente de commandes](./queue.md).

## Préparation de la session + espace de travail

-   L'espace de travail est résolu et créé ; les exécutions en sandbox peuvent être redirigées vers une racine d'espace de travail sandbox.
-   Les compétences sont chargées (ou réutilisées depuis un instantané) et injectées dans l'environnement et l'invite.
-   Les fichiers de bootstrap/contexte sont résolus et injectés dans le rapport de l'invite système.
-   Un verrou d'écriture de session est acquis ; `SessionManager` est ouvert et préparé avant le streaming.

## Assemblage de l'invite + invite système

-   L'invite système est construite à partir de l'invite de base d'OpenClaw, de l'invite des compétences, du contexte de bootstrap et des surcharges par exécution.
-   Les limites spécifiques au modèle et les jetons de réserve pour la compaction sont appliqués.
-   Voir [Invite système](./system-prompt.md) pour ce que le modèle voit.

## Points d'accroche (où vous pouvez intercepter)

OpenClaw a deux systèmes de hooks :

-   **Hooks internes** (hooks Gateway) : scripts pilotés par des événements pour les commandes et les événements du cycle de vie.
-   **Hooks de plugin** : points d'extension à l'intérieur du cycle de vie de l'agent/outil et du pipeline de la gateway.

### Hooks internes (hooks Gateway)

-   **`agent:bootstrap`** : s'exécute pendant la construction des fichiers de bootstrap avant que l'invite système ne soit finalisée. Utilisez-le pour ajouter/supprimer des fichiers de contexte de bootstrap.
-   **Hooks de commande** : `/new`, `/reset`, `/stop`, et autres événements de commande (voir le doc Hooks).

Voir [Hooks](../automation/hooks.md) pour la configuration et des exemples.

### Hooks de plugin (cycle de vie de l'agent + gateway)

Ceux-ci s'exécutent à l'intérieur de la boucle d'agent ou du pipeline de la gateway :

-   **`before_model_resolve`** : s'exécute avant la session (pas de `messages`) pour surcharger de manière déterministe le fournisseur/le modèle avant la résolution du modèle.
-   **`before_prompt_build`** : s'exécute après le chargement de la session (avec `messages`) pour injecter `prependContext`, `systemPrompt`, `prependSystemContext`, ou `appendSystemContext` avant la soumission de l'invite. Utilisez `prependContext` pour du texte dynamique par tour et les champs de contexte système pour des instructions stables qui doivent se trouver dans l'espace de l'invite système.
-   **`before_agent_start`** : hook de compatibilité hérité qui peut s'exécuter dans l'une ou l'autre phase ; préférez les hooks explicites ci-dessus.
-   **`agent_end`** : inspecte la liste finale des messages et les métadonnées de l'exécution après la fin.
-   **`before_compaction` / `after_compaction`** : observe ou annote les cycles de compaction.
-   **`before_tool_call` / `after_tool_call`** : intercepte les paramètres/résultats d'outil.
-   **`tool_result_persist`** : transforme de manière synchrone les résultats d'outil avant qu'ils ne soient écrits dans la transcription de la session.
-   **`message_received` / `message_sending` / `message_sent`** : hooks pour les messages entrants et sortants.
-   **`session_start` / `session_end`** : limites du cycle de vie de la session.
-   **`gateway_start` / `gateway_stop`** : événements du cycle de vie de la gateway.

Voir [Plugins](../tools/plugin.md#plugin-hooks) pour l'API des hooks et les détails d'enregistrement.

## Streaming + réponses partielles

-   Les deltas de l'assistant sont diffusés depuis pi-agent-core et émis comme événements `assistant`.
-   Le streaming par bloc peut émettre des réponses partielles soit sur `text_end` soit sur `message_end`.
-   Le streaming du raisonnement peut être émis comme un flux séparé ou comme des réponses par bloc.
-   Voir [Streaming](./streaming.md) pour le découpage et le comportement des réponses par bloc.

## Exécution d'outils + outils de messagerie

-   Les événements de début/mise à jour/fin d'outil sont émis sur le flux `tool`.
-   Les résultats d'outil sont assainis pour la taille et les payloads d'image avant la journalisation/l'émission.
-   Les envois d'outils de messagerie sont suivis pour supprimer les confirmations en double de l'assistant.

## Façonnage de la réponse + suppression

-   Les payloads finaux sont assemblés à partir de :
    -   le texte de l'assistant (et le raisonnement optionnel)
    -   les résumés d'outils en ligne (quand verbose + autorisé)
    -   le texte d'erreur de l'assistant quand le modèle échoue
-   `NO_REPLY` est traité comme un jeton silencieux et filtré des payloads sortants.
-   Les doublons d'outils de messagerie sont supprimés de la liste finale des payloads.
-   Si aucun payload affichable ne reste et qu'un outil a échoué, une réponse de secours pour l'erreur d'outil est émise (à moins qu'un outil de messagerie n'ait déjà envoyé une réponse visible à l'utilisateur).

## Compaction + nouvelles tentatives

-   La compaction automatique émet des événements de flux `compaction` et peut déclencher une nouvelle tentative.
-   Lors d'une nouvelle tentative, les tampons en mémoire et les résumés d'outils sont réinitialisés pour éviter une sortie en double.
-   Voir [Compaction](./compaction.md) pour le pipeline de compaction.

## Flux d'événements (aujourd'hui)

-   `lifecycle` : émis par `subscribeEmbeddedPiSession` (et en secours par `agentCommand`)
-   `assistant` : deltas diffusés depuis pi-agent-core
-   `tool` : événements d'outil diffusés depuis pi-agent-core

## Gestion des canaux de chat

-   Les deltas de l'assistant sont mis en tampon dans des messages `delta` du chat.
-   Un message `final` de chat est émis sur **lifecycle end/error**.

## Timeouts

-   `agent.wait` par défaut : 30s (seulement l'attente). Le paramètre `timeoutMs` le remplace.
-   Runtime de l'agent : `agents.defaults.timeoutSeconds` par défaut 600s ; appliqué dans le minuteur d'abandon de `runEmbeddedPiAgent`.

## Où les choses peuvent se terminer prématurément

-   Timeout de l'agent (abandon)
-   AbortSignal (annulation)
-   Déconnexion de la gateway ou timeout RPC
-   Timeout de `agent.wait` (attente uniquement, n'arrête pas l'agent)

[Runtime de l'agent](./agent.md)[Invite système](./system-prompt.md)