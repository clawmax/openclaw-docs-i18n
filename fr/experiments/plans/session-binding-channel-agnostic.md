

  Expériences

  
# Plan de Liaison de Session Indépendant du Canal

## Vue d'ensemble

Ce document définit le modèle de liaison de session indépendant du canal à long terme et le périmètre concret pour la prochaine itération de mise en œuvre. Objectif :

-   faire du routage de session liée aux sous-agents une capacité centrale
-   conserver le comportement spécifique au canal dans les adaptateurs
-   éviter les régressions dans le comportement normal de Discord

## Pourquoi ce document existe

Le comportement actuel mélange :

-   la politique de contenu des complétions
-   la politique de routage des destinations
-   les détails spécifiques à Discord

Cela a causé des cas limites tels que :

-   une livraison en double dans le canal principal et le fil de discussion lors d'exécutions concurrentes
-   l'utilisation de jetons obsolètes sur des gestionnaires de liaison réutilisés
-   l'absence de comptabilisation de l'activité pour les envois par webhook

## Périmètre de l'itération 1

Cette itération est volontairement limitée.

### 1\. Ajouter des interfaces centrales indépendantes du canal

Ajouter des types centraux et des interfaces de service pour les liaisons et le routage. Types centraux proposés :

```bash
export type BindingTargetKind = "subagent" | "session";
export type BindingStatus = "active" | "ending" | "ended";

export type ConversationRef = {
  channel: string;
  accountId: string;
  conversationId: string;
  parentConversationId?: string;
};

export type SessionBindingRecord = {
  bindingId: string;
  targetSessionKey: string;
  targetKind: BindingTargetKind;
  conversation: ConversationRef;
  status: BindingStatus;
  boundAt: number;
  expiresAt?: number;
  metadata?: Record<string, unknown>;
};
```

Contrat de service central :

```bash
export interface SessionBindingService {
  bind(input: {
    targetSessionKey: string;
    targetKind: BindingTargetKind;
    conversation: ConversationRef;
    metadata?: Record<string, unknown>;
    ttlMs?: number;
  }): Promise<SessionBindingRecord>;

  listBySession(targetSessionKey: string): SessionBindingRecord[];
  resolveByConversation(ref: ConversationRef): SessionBindingRecord | null;
  touch(bindingId: string, at?: number): void;
  unbind(input: {
    bindingId?: string;
    targetSessionKey?: string;
    reason: string;
  }): Promise<SessionBindingRecord[]>;
}
```

### 2\. Ajouter un routeur de livraison central pour les complétions de sous-agents

Ajouter un chemin unique de résolution de destination pour les événements de complétion. Contrat du routeur :

```bash
export interface BoundDeliveryRouter {
  resolveDestination(input: {
    eventKind: "task_completion";
    targetSessionKey: string;
    requester?: ConversationRef;
    failClosed: boolean;
  }): {
    binding: SessionBindingRecord | null;
    mode: "bound" | "fallback";
    reason: string;
  };
}
```

Pour cette itération :

-   seul `task_completion` est routé via ce nouveau chemin
-   les chemins existants pour les autres types d'événements restent inchangés

### 3\. Conserver Discord comme adaptateur

Discord reste la première implémentation d'adaptateur. Responsabilités de l'adaptateur :

-   créer/réutiliser les conversations de fil de discussion
-   envoyer les messages liés via webhook ou envoi de canal
-   valider l'état du fil de discussion (archivé/supprimé)
-   mapper les métadonnées de l'adaptateur (identité du webhook, identifiants de fil)

### 4\. Corriger les problèmes de justesse actuellement connus

Nécessaire dans cette itération :

-   actualiser l'utilisation du jeton lors de la réutilisation d'un gestionnaire de liaison de fil existant
-   enregistrer l'activité sortante pour les envois Discord basés sur des webhooks
-   arrêter le repli implicite vers le canal principal lorsqu'une destination de fil liée est sélectionnée pour une complétion en mode session

### 5\. Préserver les paramètres de sécurité d'exécution actuels par défaut

Aucun changement de comportement pour les utilisateurs ayant désactivé la création liée aux fils. Les valeurs par défaut restent :

-   `channels.discord.threadBindings.spawnSubagentSessions = false`

Résultat :

-   les utilisateurs normaux de Discord conservent le comportement actuel
-   le nouveau chemin central n'affecte que le routage des complétions de session liée là où il est activé

## Non inclus dans l'itération 1

Explicitement reporté :

-   les cibles de liaison ACP (`targetKind: "acp"`)
-   les nouveaux adaptateurs de canal au-delà de Discord
-   le remplacement global de tous les chemins de livraison (`spawn_ack`, futur `subagent_message`)
-   les changements au niveau du protocole
-   la refonte de la migration/versionnage du stockage pour toute la persistance des liaisons

Notes sur l'ACP :

-   la conception de l'interface laisse de la place pour l'ACP
-   l'implémentation de l'ACP n'est pas commencée dans cette itération

## Invariants de routage

Ces invariants sont obligatoires pour l'itération 1.

-   la sélection de la destination et la génération du contenu sont des étapes distinctes
-   si une complétion en mode session résout vers une destination liée active, la livraison doit cibler cette destination
-   pas de reroutage caché de la destination liée vers le canal principal
-   le comportement de repli doit être explicite et observable

## Compatibilité et déploiement

Objectif de compatibilité :

-   aucune régression pour les utilisateurs ayant désactivé la création liée aux fils
-   aucun changement pour les canaux non-Discord dans cette itération

Déploiement :

1.  Intégrer les interfaces et le routeur derrière les portes de fonctionnalités actuelles.
2.  Router les livraisons liées en mode complétion Discord via le routeur.
3.  Conserver l'ancien chemin pour les flux non liés.
4.  Vérifier avec des tests ciblés et des journaux d'exécution en canary.

## Tests requis dans l'itération 1

Couverture unitaire et d'intégration requise :

-   la rotation des jetons du gestionnaire utilise le dernier jeton après réutilisation du gestionnaire
-   les envois par webhook mettent à jour les horodatages d'activité du canal
-   deux sessions liées actives dans le même canal demandeur ne dupliquent pas vers le canal principal
-   la complétion pour une exécution en mode session liée se résout uniquement vers la destination du fil
-   le drapeau de création désactivé maintient le comportement hérité inchangé

## Fichiers d'implémentation proposés

Central :

-   `src/infra/outbound/session-binding-service.ts` (nouveau)
-   `src/infra/outbound/bound-delivery-router.ts` (nouveau)
-   `src/agents/subagent-announce.ts` (intégration de la résolution de destination des complétions)

Adaptateur Discord et exécution :

-   `src/discord/monitor/thread-bindings.manager.ts`
-   `src/discord/monitor/reply-delivery.ts`
-   `src/discord/send.outbound.ts`

Tests :

-   `src/discord/monitor/provider*.test.ts`
-   `src/discord/monitor/reply-delivery.test.ts`
-   `src/agents/subagent-announce.format.test.ts`

## Critères de fin pour l'itération 1

-   les interfaces centrales existent et sont connectées pour le routage des complétions
-   les corrections de justesse ci-dessus sont fusionnées avec des tests
-   aucune livraison de complétion en double dans le canal principal et le fil de discussion lors d'exécutions liées en mode session
-   aucun changement de comportement pour les déploiements avec création liée désactivée
-   l'ACP reste explicitement reportée

[Plan de Supervision PTY et Processus](./pty-process-supervision.md)[Recherche sur la Mémoire de l'Espace de Travail](../research/memory.md)