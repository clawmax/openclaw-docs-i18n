

  Expériences

  
# Agents liés aux threads ACP

## Vue d'ensemble

Ce plan définit comment OpenClaw doit prendre en charge les agents de codage ACP dans les canaux compatibles avec les threads (Discord en premier) avec un cycle de vie et une récupération de niveau production. Document connexe :

-   [Plan de refactorisation du streaming du runtime unifié](./acp-unified-streaming-refactor.md)

Expérience utilisateur cible :

-   un utilisateur lance ou focalise une session ACP dans un thread
-   les messages de l'utilisateur dans ce thread sont acheminés vers la session ACP liée
-   la sortie de l'agent est diffusée en continu vers la même persona du thread
-   la session peut être persistante ou unique avec des contrôles de nettoyage explicites

## Résumé des décisions

La recommandation à long terme est une architecture hybride :

-   Le cœur d'OpenClaw possède les préoccupations du plan de contrôle ACP
    -   identité et métadonnées de session
    -   liaison aux threads et décisions de routage
    -   invariants de livraison et suppression des doublons
    -   sémantique de nettoyage du cycle de vie et de récupération
-   Le backend du runtime ACP est interchangeable
    -   le premier backend est un service de plugin basé sur acpx
    -   le runtime gère le transport ACP, la mise en file d'attente, l'annulation, la reconnexion

OpenClaw ne doit pas réimplémenter les détails internes du transport ACP dans le cœur. OpenClaw ne doit pas dépendre d'un chemin d'interception purement basé sur des plugins pour le routage.

## Architecture idéale (Saint Graal)

Traiter ACP comme un plan de contrôle de première classe dans OpenClaw, avec des adaptateurs de runtime interchangeables. Invariants non négociables :

-   chaque liaison de thread ACP référence un enregistrement de session ACP valide
-   chaque session ACP a un état de cycle de vie explicite (`creating`, `idle`, `running`, `cancelling`, `closed`, `error`)
-   chaque exécution ACP a un état d'exécution explicite (`queued`, `running`, `completed`, `failed`, `cancelled`)
-   le lancement, la liaison et la mise en file d'attente initiale sont atomiques
-   les nouvelles tentatives de commande sont idempotentes (pas d'exécutions en double ni de sorties Discord en double)
-   la sortie du canal lié au thread est une projection des événements d'exécution ACP, jamais des effets secondaires ad hoc

Modèle de propriété à long terme :

-   `AcpSessionManager` est le seul écrivain et orchestrateur ACP
-   le gestionnaire vit d'abord dans le processus de passerelle ; peut être déplacé plus tard vers un sidecar dédié derrière la même interface
-   par clé de session ACP, le gestionnaire possède un acteur en mémoire (exécution de commande sérialisée)
-   les adaptateurs (`acpx`, futurs backends) sont uniquement des implémentations de transport/runtime

Modèle de persistance à long terme :

-   déplacer l'état du plan de contrôle ACP vers un stockage SQLite dédié (mode WAL) sous le répertoire d'état d'OpenClaw
-   conserver `SessionEntry.acp` comme projection de compatibilité pendant la migration, pas comme source de vérité
-   stocker les événements ACP en mode ajout uniquement pour prendre en charge la relecture, la récupération après incident et la livraison déterministe

### Stratégie de livraison (pont vers le Saint Graal)

-   pont à court terme
    -   conserver les mécanismes actuels de liaison aux threads et la surface de configuration ACP existante
    -   corriger les bogues de manque de métadonnées et acheminer les tours ACP via une seule branche ACP centrale
    -   ajouter immédiatement des clés d'idempotence et des vérifications de routage en échec fermé
-   basculement à long terme
    -   déplacer la source de vérité ACP vers la base de données du plan de contrôle + les acteurs
    -   rendre la livraison liée aux threads purement basée sur la projection d'événements
    -   supprimer le comportement de secours hérité qui dépend des métadonnées opportunistes des entrées de session

## Pourquoi pas uniquement des plugins purs

Les crochets de plugin actuels ne sont pas suffisants pour le routage de session ACP de bout en bout sans modifications du cœur.

-   le routage entrant depuis la liaison aux threads résout d'abord une clé de session dans la distribution centrale
-   les crochets de message sont de type "fire-and-forget" et ne peuvent pas court-circuiter le chemin de réponse principal
-   les commandes de plugin sont bonnes pour les opérations de contrôle mais pas pour remplacer le flux de distribution par tour central

Résultat :

-   le runtime ACP peut être pluginisé
-   la branche de routage ACP doit exister dans le cœur

## Fondation existante à réutiliser

Déjà implémenté et doit rester canonique :

-   la cible de liaison aux threads prend en charge `subagent` et `acp`
-   la surcharge de routage entrant des threads se résout par liaison avant la distribution normale
-   identité de thread sortant via webhook dans la livraison des réponses
-   flux `/focus` et `/unfocus` avec compatibilité de la cible ACP
-   stockage de liaison persistant avec restauration au démarrage
-   cycle de vie de déliaison à l'archivage, la suppression, le défocus, la réinitialisation et la suppression

Ce plan étend cette fondation plutôt que de la remplacer.

## Architecture

### Modèle de frontière

Cœur (doit être dans le cœur d'OpenClaw) :

-   branche de distribution en mode session ACP dans le pipeline de réponse
-   arbitrage de livraison pour éviter la duplication parent plus thread
-   persistance du plan de contrôle ACP (avec projection de compatibilité `SessionEntry.acp` pendant la migration)
-   sémantique de déliaison du cycle de vie et de détachement du runtime liée à la réinitialisation/suppression de session

Backend de plugin (implémentation acpx) :

-   supervision du worker du runtime ACP
-   invocation du processus acpx et analyse des événements
-   gestionnaires de commandes ACP (`/acp ...`) et UX opérateur
-   valeurs par défaut et diagnostics spécifiques au backend

### Modèle de propriété du runtime

-   un processus de passerelle possède l'état d'orchestration ACP
-   l'exécution ACP s'exécute dans des processus enfants supervisés via le backend acpx
-   la stratégie de processus est de longue durée par clé de session ACP active, pas par message

Cela évite le coût de démarrage à chaque prompt et garde les sémantiques d'annulation et de reconnexion fiables.

### Contrat de runtime central

Ajouter un contrat de runtime ACP central pour que le code de routage ne dépende pas des détails CLI et puisse changer de backend sans modifier la logique de distribution :

```bash
export type AcpRuntimePromptMode = "prompt" | "steer";

export type AcpRuntimeHandle = {
  sessionKey: string;
  backend: string;
  runtimeSessionName: string;
};

export type AcpRuntimeEvent =
  | { type: "text_delta"; stream: "output" | "thought"; text: string }
  | { type: "tool_call"; name: string; argumentsText: string }
  | { type: "done"; usage?: Record<string, number> }
  | { type: "error"; code: string; message: string; retryable?: boolean };

export interface AcpRuntime {
  ensureSession(input: {
    sessionKey: string;
    agent: string;
    mode: "persistent" | "oneshot";
    cwd?: string;
    env?: Record<string, string>;
    idempotencyKey: string;
  }): Promise<AcpRuntimeHandle>;

  submit(input: {
    handle: AcpRuntimeHandle;
    text: string;
    mode: AcpRuntimePromptMode;
    idempotencyKey: string;
  }): Promise<{ runtimeRunId: string }>;

  stream(input: {
    handle: AcpRuntimeHandle;
    runtimeRunId: string;
    onEvent: (event: AcpRuntimeEvent) => Promise<void> | void;
    signal?: AbortSignal;
  }): Promise<void>;

  cancel(input: {
    handle: AcpRuntimeHandle;
    runtimeRunId?: string;
    reason?: string;
    idempotencyKey: string;
  }): Promise<void>;

  close(input: { handle: AcpRuntimeHandle; reason: string; idempotencyKey: string }): Promise<void>;

  health?(): Promise<{ ok: boolean; details?: string }>;
}
```

Détail d'implémentation :

-   premier backend : `AcpxRuntime` livré en tant que service de plugin
-   le cœur résout le runtime via un registre et échoue avec une erreur d'opérateur explicite lorsqu'aucun backend de runtime ACP n'est disponible

### Modèle de données du plan de contrôle et persistance

La source de vérité à long terme est une base de données SQLite ACP dédiée (mode WAL), pour les mises à jour transactionnelles et la récupération sécurisée après incident :

-   `acp_sessions`
    -   `session_key` (pk), `backend`, `agent`, `mode`, `cwd`, `state`, `created_at`, `updated_at`, `last_error`
-   `acp_runs`
    -   `run_id` (pk), `session_key` (fk), `state`, `requester_message_id`, `idempotency_key`, `started_at`, `ended_at`, `error_code`, `error_message`
-   `acp_bindings`
    -   `binding_key` (pk), `thread_id`, `channel_id`, `account_id`, `session_key` (fk), `expires_at`, `bound_at`
-   `acp_events`
    -   `event_id` (pk), `run_id` (fk), `seq`, `kind`, `payload_json`, `created_at`
-   `acp_delivery_checkpoint`
    -   `run_id` (pk/fk), `last_event_seq`, `last_discord_message_id`, `updated_at`
-   `acp_idempotency`
    -   `scope`, `idempotency_key`, `result_json`, `created_at`, unique `(scope, idempotency_key)`

```bash
export type AcpSessionMeta = {
  backend: string;
  agent: string;
  runtimeSessionName: string;
  mode: "persistent" | "oneshot";
  cwd?: string;
  state: "idle" | "running" | "error";
  lastActivityAt: number;
  lastError?: string;
};
```

Règles de stockage :

-   conserver `SessionEntry.acp` comme projection de compatibilité pendant la migration
-   les identifiants de processus et les sockets restent uniquement en mémoire
-   le cycle de vie durable et l'état d'exécution vivent dans la base de données ACP, pas dans le JSON de session générique
-   si le propriétaire du runtime meurt, la passerelle se réhydrate à partir de la base de données ACP et reprend à partir des points de contrôle

### Routage et livraison

Entrant :

-   conserver la recherche de liaison aux threads actuelle comme première étape de routage
-   si la cible liée est une session ACP, acheminer vers la branche du runtime ACP au lieu de `getReplyFromConfig`
-   la commande explicite `/acp steer` utilise `mode: "steer"`

Sortant :

-   le flux d'événements ACP est normalisé en fragments de réponse OpenClaw
-   la cible de livraison est résolue via le chemin de destination lié existant
-   lorsqu'un thread lié est actif pour ce tour de session, l'achèvement du canal parent est supprimé

Politique de streaming :

-   diffuser la sortie partielle avec une fenêtre de coalescence
-   intervalle minimum et octets maximum par fragment configurables pour rester sous les limites de débit de Discord
-   le message final est toujours émis à l'achèvement ou à l'échec

### Machines à états et limites transactionnelles

Machine à états de session :

-   `creating -> idle -> running -> idle`
-   `running -> cancelling -> idle | error`
-   `idle -> closed`
-   `error -> idle | closed`

Machine à états d'exécution :

-   `queued -> running -> completed`
-   `running -> failed | cancelled`
-   `queued -> cancelled`

Limites transactionnelles requises :

-   transaction de lancement
    -   créer une ligne de session ACP
    -   créer/mettre à jour une ligne de liaison de thread ACP
    -   mettre en file d'attente une ligne d'exécution initiale
-   transaction de fermeture
    -   marquer la session comme fermée
    -   supprimer/expirer les lignes de liaison
    -   écrire l'événement de fermeture final
-   transaction d'annulation
    -   marquer l'exécution cible comme annulation/annulée avec une clé d'idempotence

Aucun succès partiel n'est autorisé à travers ces limites.

### Modèle d'acteur par session

`AcpSessionManager` exécute un acteur par clé de session ACP :

-   la boîte aux lettres de l'acteur sérialise les effets secondaires `submit`, `cancel`, `close` et `stream`
-   l'acteur possède l'hydratation du handle de runtime et le cycle de vie du processus d'adaptateur de runtime pour cette session
-   l'acteur écrit les événements d'exécution dans l'ordre (`seq`) avant toute livraison Discord
-   l'acteur met à jour les points de contrôle de livraison après un envoi sortant réussi

Cela supprime les courses entre tours et empêche la sortie de thread en double ou dans le désordre.

### Idempotence et projection de livraison

Toutes les actions ACP externes doivent porter des clés d'idempotence :

-   clé d'idempotence de lancement
-   clé d'idempotence de prompt/steer
-   clé d'idempotence d'annulation
-   clé d'idempotence de fermeture

Règles de livraison :

-   les messages Discord sont dérivés de `acp_events` plus `acp_delivery_checkpoint`
-   les nouvelles tentatives reprennent à partir du point de contrôle sans renvoyer les fragments déjà livrés
-   l'émission de réponse finale est exactement une fois par exécution à partir de la logique de projection

### Récupération et auto-guérison

Au démarrage de la passerelle :

-   charger les sessions ACP non terminales (`creating`, `idle`, `running`, `cancelling`, `error`)
-   recréer les acteurs de manière paresseuse au premier événement entrant ou de manière avide sous une limite configurée
-   réconcilier les exécutions `running` manquant des battements de cœur et les marquer `failed` ou les récupérer via l'adaptateur

Sur message de thread Discord entrant :

-   si une liaison existe mais que la session ACP est manquante, échouer en mode fermé avec un message explicite de liaison obsolète
-   optionnellement, délier automatiquement la liaison obsolète après validation sécurisée pour l'opérateur
-   ne jamais acheminer silencieusement les liaisons ACP obsolètes vers le chemin LLM normal

### Cycle de vie et sécurité

Opérations prises en charge :

-   annuler l'exécution en cours : `/acp cancel`
-   délier le thread : `/unfocus`
-   fermer la session ACP : `/acp close`
-   fermeture automatique des sessions inactives par TTL effectif

Politique TTL :

-   TTL effectif est le minimum de
    -   TTL global/de session
    -   TTL de liaison de thread Discord
    -   TTL du propriétaire du runtime ACP

Contrôles de sécurité :

-   liste d'autorisation des agents ACP par nom
-   restreindre les racines d'espace de travail pour les sessions ACP
-   liste d'autorisation d'environnement pour la transmission
-   nombre maximum de sessions ACP simultanées par compte et globalement
-   backoff de redémarrage limité pour les plantages du runtime

## Surface de configuration

Clés centrales :

-   `acp.enabled`
-   `acp.dispatch.enabled` (interrupteur d'arrêt de routage ACP indépendant)
-   `acp.backend` (par défaut `acpx`)
-   `acp.defaultAgent`
-   `acp.allowedAgents[]`
-   `acp.maxConcurrentSessions`
-   `acp.stream.coalesceIdleMs`
-   `acp.stream.maxChunkChars`
-   `acp.runtime.ttlMinutes`
-   `acp.controlPlane.store` (`sqlite` par défaut)
-   `acp.controlPlane.storePath`
-   `acp.controlPlane.recovery.eagerActors`
-   `acp.controlPlane.recovery.reconcileRunningAfterMs`
-   `acp.controlPlane.checkpoint.flushEveryEvents`
-   `acp.controlPlane.checkpoint.flushEveryMs`
-   `acp.idempotency.ttlHours`
-   `channels.discord.threadBindings.spawnAcpSessions`

Clés de plugin/backend (section plugin acpx) :

-   remplacements de commande/chemin du backend
-   liste d'autorisation d'environnement du backend
-   préréglages par agent du backend
-   délais d'attente de démarrage/arrêt du backend
-   nombre maximum d'exécutions en cours par session du backend

## Spécification d'implémentation

### Modules du plan de contrôle (nouveaux)

Ajouter des modules dédiés au plan de contrôle ACP dans le cœur :

-   `src/acp/control-plane/manager.ts`
    -   possède les acteurs ACP, les transitions de cycle de vie, la sérialisation des commandes
-   `src/acp/control-plane/store.ts`
    -   gestion du schéma SQLite, transactions, aides de requête
-   `src/acp/control-plane/events.ts`
    -   définitions et sérialisation d'événements ACP typés
-   `src/acp/control-plane/checkpoint.ts`
    -   points de contrôle de livraison durables et curseurs de relecture
-   `src/acp/control-plane/idempotency.ts`
    -   réservation de clé d'idempotence et relecture de réponse
-   `src/acp/control-plane/recovery.ts`
    -   plan de réconciliation au démarrage et de réhydratation d'acteur

Modules de pont de compatibilité :

-   `src/acp/runtime/session-meta.ts`
    -   reste temporairement pour la projection dans `SessionEntry.acp`
    -   doit cesser d'être la source de vérité après le basculement de migration

### Invariants requis (doivent être appliqués dans le code)

-   la création de session ACP et la liaison aux threads sont atomiques (transaction unique)
-   il y a au plus une exécution active par acteur de session ACP à la fois
-   le `seq` d'événement est strictement croissant par exécution
-   le point de contrôle de livraison n'avance jamais au-delà du dernier événement validé
-   la relecture d'idempotence renvoie la charge utile de succès précédente pour les clés de commande en double
-   les métadonnées ACP obsolètes/manquantes ne peuvent pas être acheminées vers le chemin de réponse non-ACP normal

### Points de contact centraux

Fichiers centraux à modifier :

-   `src/auto-reply/reply/dispatch-from-config.ts`
    -   la branche ACP appelle `AcpSessionManager.submit` et la livraison par projection d'événements
    -   supprimer le secours ACP direct qui contourne les invariants du plan de contrôle
-   `src/auto-reply/reply/inbound-context.ts` (ou la limite de contexte normalisée la plus proche)
    -   exposer les clés de routage normalisées et les graines d'idempotence pour le plan de contrôle ACP
-   `src/config/sessions/types.ts`
    -   conserver `SessionEntry.acp` comme champ de compatibilité uniquement en projection
-   `src/gateway/server-methods/sessions.ts`
    -   la réinitialisation/la suppression/l'archivage doivent appeler le chemin de transaction de fermeture/déliaison du gestionnaire ACP
-   `src/infra/outbound/bound-delivery-router.ts`
    -   appliquer le comportement de destination en échec fermé pour les tours de session liés ACP
-   `src/discord/monitor/thread-bindings.ts`
    -   ajouter des aides de validation de liaison obsolète ACP connectées aux recherches du plan de contrôle
-   `src/auto-reply/reply/commands-acp.ts`
    -   acheminer le lancement/l'annulation/la fermeture/le steer via les API du gestionnaire ACP
-   `src/agents/acp-spawn.ts`
    -   arrêter les écritures de métadonnées ad hoc ; appeler la transaction de lancement du gestionnaire ACP
-   `src/plugin-sdk/**` et pont de runtime de plugin
    -   exposer l'enregistrement du backend ACP et la sémantique de santé de manière propre

Fichiers centraux explicitement non remplacés :

-   `src/discord/monitor/message-handler.preflight.ts`
    -   conserver le comportement de surcharge de liaison aux threads comme résolveur de clé de session canonique

### API de registre du runtime ACP

Ajouter un module de registre central :

-   `src/acp/runtime/registry.ts`

API requise :

```bash
export type AcpRuntimeBackend = {
  id: string;
  runtime: AcpRuntime;
  healthy?: () => boolean;
};

export function registerAcpRuntimeBackend(backend: AcpRuntimeBackend): void;
export function unregisterAcpRuntimeBackend(id: string): void;
export function getAcpRuntimeBackend(id?: string): AcpRuntimeBackend | null;
export function requireAcpRuntimeBackend(id?: string): AcpRuntimeBackend;
```

Comportement :

-   `requireAcpRuntimeBackend` lance une erreur typée de backend ACP manquant lorsqu'il est indisponible
-   le service de plugin enregistre le backend au `start` et le désenregistre au `stop`
-   les recherches de runtime sont en lecture seule et locales au processus

### Contrat de plugin de runtime acpx (détail d'implémentation)

Pour le premier backend de production (`extensions/acpx`), OpenClaw et acpx sont connectés avec un contrat de commande strict :

-   id du backend : `acpx`
-   id du service de plugin : `acpx-runtime`
-   encodage du handle de runtime : `runtimeSessionName = acpx:v1:<base64url(json)>`
-   champs de charge utile encodés :
    -   `name` (session nommée acpx ; utilise la `sessionKey` d'OpenClaw)
    -   `agent` (commande d'agent acpx)
    -   `cwd` (racine de l'espace de travail de session)
    -   `mode` (`persistent | oneshot`)

Correspondance des commandes :

-   assurer la session :
    -   `acpx --format json --json-strict --cwd   sessions ensure --name `
-   tour de prompt :
    -   `acpx --format json --json-strict --cwd   prompt --session  --file -`
-   annuler :
    -   `acpx --format json --json-strict --cwd   cancel --session `
-   fermer :
    -   `acpx --format json --json-strict --cwd   sessions close `

Streaming :

-   OpenClaw consomme les événements ndjson depuis `acpx --format json --json-strict`
-   `text` => `text_delta/output`
-   `thought` => `text_delta/thought`
-   `tool_call` => `tool_call`
-   `done` => `done`
-   `error` => `error`

### Correctif de schéma de session

Corriger `SessionEntry` dans `src/config/sessions/types.ts` :

```typescript
type SessionAcpMeta = {
  backend: string;
  agent: string;
  runtimeSessionName: string;
  mode: "persistent" | "oneshot";
  cwd?: string;
  state: "idle" | "running" | "error";
  lastActivityAt: number;
  lastError?: string;
};
```

Champ persistant :

-   `SessionEntry.acp?: SessionAcpMeta`

Règles de migration :

-   phase A : écriture double (projection `acp` + source de vérité SQLite ACP)
-   phase B : lecture principale depuis SQLite ACP, lecture de secours depuis l'entrée héritée `SessionEntry.acp`
-   phase C : commande de migration remplit les lignes ACP manquantes à partir des entrées héritées valides
-   phase D : supprimer la lecture de secours et garder la projection optionnelle uniquement pour l'UX
-   les champs hérités (`cliSessionIds`, `claudeCliSessionId`) restent inchangés

### Contrat d'erreur

Ajouter des codes d'erreur ACP stables et des messages orientés utilisateur :

-   `ACP_BACKEND_MISSING`
    -   message : `Le backend du runtime ACP n'est pas configuré. Installez et activez le plugin de runtime acpx.`
-   `ACP_BACKEND_UNAVAILABLE`
    -   message : `Le backend du runtime ACP est actuellement indisponible. Réessayez dans un instant.`
-   `ACP_SESSION_INIT_FAILED`
    -   message : `Impossible d'initialiser le runtime de session ACP.`
-   `ACP_TURN_FAILED`
    -   message : `Le tour ACP a échoué avant l'achèvement.`

Règles :

-   renvoyer un message sécurisé et actionnable pour l'utilisateur dans le thread
-   journaliser l'erreur détaillée du backend/système uniquement dans les journaux du runtime
-   ne jamais basculer silencieusement vers le chemin LLM normal lorsque le routage ACP a été explicitement sélectionné

### Arbitrage de livraison en double

Règle de routage unique pour les tours liés ACP :

-   si une liaison de thread active existe pour la session ACP cible et le contexte du demandeur, livrer uniquement à ce thread lié
-   ne pas envoyer également au canal parent pour le même tour
-   si la sélection de destination liée est ambiguë, échouer en mode fermé avec une erreur explicite (pas de secours implicite vers le parent)
-   si aucune liaison active n'existe, utiliser le comportement de destination de session normal

### Observabilité et préparation opérationnelle

Métriques requises :

-   nombre de lancements ACP réussis/échoués par backend et code d'erreur
-   percentiles de latence d'exécution ACP (attente en file, temps de tour du runtime, temps de projection de livraison)
-   nombre de redémarrages d'acteur ACP et raison de redémarrage
-   nombre de détections de liaison obsolète
-   taux de succès de relecture d'idempotence
-   compteurs de nouvelles tentatives de livraison Discord et de limites de débit

Journaux requis :

-   journaux structurés indexés par `sessionKey`, `runId`, `backend`, `threadId`, `idempotencyKey`
-   journaux de transition d'état explicites pour les machines à états de session et d'exécution
-   journaux de commande d'adaptateur avec arguments sécurisés pour la rédaction et résumé de sortie

Diagnostics requis :

-   `/acp sessions` inclut l'état, l'exécution active, la dernière erreur et l'état de liaison
-   `/acp doctor` (ou équivalent) valide l'enregistrement du backend, la santé du stockage et les liaisons obsolètes

### Priorité de configuration et valeurs effectives

Priorité d'activation ACP :

-   remplacement de compte : `channels.discord.accounts..threadBindings.spawnAcpSessions`
-   remplacement de canal : `channels.discord.threadBindings.spawnAcpSessions`
-   porte globale ACP : `acp.enabled`
-   porte de distribution : `acp.dispatch.enabled`
-   disponibilité du backend : backend enregistré pour `acp.backend`

Comportement d'auto-activation :

-   lorsque ACP est configuré (`acp.enabled=true`, `acp.dispatch.enabled=true`, ou `acp.backend=acpx`), l'auto-activation du plugin marque `plugins.entries.acpx.enabled=true` sauf si interdite ou explicitement désactivée

Valeur TTL effective :

-   `min(ttl de session, ttl de liaison de thread discord, ttl du runtime acp)`

### Carte de test

Tests unitaires :

-   `src/acp/runtime/registry.test.ts` (nouveau)
-   `src/auto-reply/reply/dispatch-from-config.acp.test.ts` (nouveau)
-   `src/infra/outbound/bound-delivery-router.test.ts` (étendre les cas d'échec fermé ACP)
-   `src/config/sessions/types.test.ts` ou les tests de stockage de session les plus proches (persistance des métadonnées ACP)

Tests d'intégration :

-   `src/discord/monitor/reply-delivery.test.ts` (comportement de destination de livraison liée ACP)
-   `src/discord/monitor/message-handler.preflight*.test.ts` (continuité de routage de clé de session liée ACP)
-   tests de runtime du plugin acpx dans le package backend (enregistrement/démarrage/arrêt du service + normalisation d'événements)

Tests e2e de passerelle :

-   `src/gateway/server.sessions.gateway-server-sessions-a.e2e.test.ts` (étendre la couverture du cycle de vie de réinitialisation/suppression ACP)
-   e2e de tour de thread ACP pour le lancement, le message, le streaming, l'annulation, le défocus, la récupération après redémarrage

### Garde de déploiement

Ajouter un interrupteur d'arrêt de distribution ACP indépendant :

-   `acp.dispatch.enabled` par défaut `false` pour la première version
-   lorsqu'il est désactivé :
    -   les commandes de contrôle de lancement/focus ACP peuvent toujours lier des sessions
    -   le chemin de distribution ACP ne s'active pas
    -   l'utilisateur reçoit un message explicite que la distribution ACP est désactivée par la politique
-   après validation canari, la valeur par défaut peut être basculée sur `true` dans une version ultérieure

## Plan de commande et d'UX

### Nouvelles commandes

-   `/acp spawn <agent-id> [--mode persistent|oneshot] [--thread auto|here|off]`
-   `/acp cancel [session]`
-   `/acp steer `
-   `/acp close [session]`
-   `/acp sessions`

### Compatibilité des commandes existantes

-   `/focus ` continue de prendre en charge les cibles ACP
-   `/unfocus` conserve la sémantique actuelle
-   `/session idle` et `/session max-age` remplacent l'ancienne surcharge TTL

## Déploiement par phases

### Phase 0 ADR et gel du schéma

-   livrer l'ADR pour la propriété du plan de contrôle ACP et les frontières d'adaptateur
-   geler le schéma de base de données (`acp_sessions`, `acp_runs`, `acp_bindings`, `acp_events`, `acp_delivery_checkpoint`, `acp_idempotency`)
-   définir les codes d'erreur ACP stables, le contrat d'événement et les gardes de transition d'état

### Phase 1 Fondation du plan de contrôle dans le cœur

-   implémenter `AcpSessionManager` et le runtime d'acteur par session
-   implémenter le stockage SQLite ACP et les aides transactionnelles
-   implémenter le stockage d'idempotence et les aides de relecture
-   implémenter les modules d'