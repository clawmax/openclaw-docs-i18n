

  Messages et livraison

  
# Messages

Cette page explique comment OpenClaw gère les messages entrants, les sessions, la mise en file d'attente, le streaming et la visibilité du raisonnement.

## Flux des messages (vue d'ensemble)

```
Message entrant
  -> routage/liens -> clé de session
  -> file d'attente (si une exécution est active)
  -> exécution de l'agent (streaming + outils)
  -> réponses sortantes (limites du canal + fragmentation)
```

Les paramètres clés se trouvent dans la configuration :

-   `messages.*` pour les préfixes, la mise en file d'attente et le comportement de groupe.
-   `agents.defaults.*` pour les valeurs par défaut du streaming par blocs et de la fragmentation.
-   Les surcharges par canal (`channels.whatsapp.*`, `channels.telegram.*`, etc.) pour les plafonds et les options de streaming.

Voir [Configuration](../gateway/configuration.md) pour le schéma complet.

## Déduplication des messages entrants

Les canaux peuvent redélivrer le même message après des reconnexions. OpenClaw maintient un cache de courte durée indexé par canal/compte/peer/session/ID de message afin que les livraisons en double ne déclenchent pas une autre exécution de l'agent.

## Anti-rebond des messages entrants

Des messages consécutifs rapides du **même expéditeur** peuvent être regroupés en un seul tour d'agent via `messages.inbound`. L'anti-rebond est limité au canal + conversation et utilise le message le plus récent pour le threading/les ID des réponses. Configuration (valeur par défaut globale + surcharges par canal) :

```json
{
  messages: {
    inbound: {
      debounceMs: 2000,
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
        discord: 1500,
      },
    },
  },
}
```

Notes :

-   L'anti-rebond s'applique aux messages **uniquement textuels** ; les médias/pièces jointes sont envoyés immédiatement.
-   Les commandes de contrôle contournent l'anti-rebond pour rester autonomes.

## Sessions et appareils

Les sessions sont gérées par la passerelle, et non par les clients.

-   Les discussions directes sont fusionnées dans la clé de session principale de l'agent.
-   Les groupes/canaux obtiennent leurs propres clés de session.
-   Le stockage des sessions et les transcriptions résident sur l'hôte de la passerelle.

Plusieurs appareils/canaux peuvent correspondre à la même session, mais l'historique n'est pas entièrement synchronisé vers chaque client. Recommandation : utilisez un appareil principal pour les conversations longues afin d'éviter un contexte divergent. L'interface de contrôle et le TUI affichent toujours la transcription de session gérée par la passerelle, ils sont donc la source de vérité. Détails : [Gestion des sessions](./session.md).

## Corps des messages entrants et contexte de l'historique

OpenClaw sépare le **corps de l'invite** du **corps de la commande** :

-   `Body` : texte de l'invite envoyé à l'agent. Cela peut inclure des enveloppes de canal et des encapsulations d'historique optionnelles.
-   `CommandBody` : texte brut de l'utilisateur pour l'analyse des directives/commandes.
-   `RawBody` : alias historique pour `CommandBody` (conservé pour la compatibilité).

Lorsqu'un canal fournit un historique, il utilise un wrapper partagé :

-   `[Messages de discussion depuis votre dernière réponse - pour le contexte]`
-   `[Message actuel - répondez à celui-ci]`

Pour les **discussions non directes** (groupes/canaux/salles), le **corps du message actuel** est préfixé par l'étiquette de l'expéditeur (même style utilisé pour les entrées d'historique). Cela garantit la cohérence des messages en temps réel et des messages en file d'attente/dans l'historique dans l'invite de l'agent. Les tampons d'historique sont **uniquement en attente** : ils incluent les messages de groupe qui n'ont *pas* déclenché une exécution (par exemple, les messages conditionnés par une mention) et **excluent** les messages déjà présents dans la transcription de la session. Le retrait des directives ne s'applique qu'à la section **message actuel**, l'historique reste donc intact. Les canaux qui encapsulent l'historique doivent définir `CommandBody` (ou `RawBody`) sur le texte original du message et conserver `Body` comme l'invite combinée. Les tampons d'historique sont configurables via `messages.groupChat.historyLimit` (valeur par défaut globale) et des surcharges par canal comme `channels.slack.historyLimit` ou `channels.telegram.accounts..historyLimit` (définir `0` pour désactiver).

## Mise en file d'attente et suivis

Si une exécution est déjà active, les messages entrants peuvent être mis en file d'attente, intégrés à l'exécution en cours ou collectés pour un tour de suivi.

-   Configurable via `messages.queue` (et `messages.queue.byChannel`).
-   Modes : `interrupt`, `steer`, `followup`, `collect`, ainsi que des variantes d'arriéré.

Détails : [Mise en file d'attente](./queue.md).

## Streaming, fragmentation et regroupement

Le streaming par blocs envoie des réponses partielles au fur et à mesure que le modèle produit des blocs de texte. La fragmentation respecte les limites de texte du canal et évite de diviser les blocs de code délimités. Paramètres clés :

-   `agents.defaults.blockStreamingDefault` (`on|off`, désactivé par défaut)
-   `agents.defaults.blockStreamingBreak` (`text_end|message_end`)
-   `agents.defaults.blockStreamingChunk` (`minChars|maxChars|breakPreference`)
-   `agents.defaults.blockStreamingCoalesce` (regroupement basé sur l'inactivité)
-   `agents.defaults.humanDelay` (pause humaine entre les réponses par blocs)
-   Surcharges par canal : `*.blockStreaming` et `*.blockStreamingCoalesce` (les canaux non-Telegram nécessitent un `*.blockStreaming: true` explicite)

Détails : [Streaming + fragmentation](./streaming.md).

## Visibilité du raisonnement et jetons

OpenClaw peut exposer ou masquer le raisonnement du modèle :

-   `/reasoning on|off|stream` contrôle la visibilité.
-   Le contenu du raisonnement compte toujours dans l'utilisation des jetons lorsqu'il est produit par le modèle.
-   Telegram prend en charge le flux de raisonnement dans la bulle de brouillon.

Détails : [Directives de réflexion + raisonnement](../tools/thinking.md) et [Utilisation des jetons](../reference/token-use.md).

## Préfixes, threading et réponses

Le formatage des messages sortants est centralisé dans `messages` :

-   `messages.responsePrefix`, `channels..responsePrefix`, et `channels..accounts..responsePrefix` (cascade de préfixes sortants), plus `channels.whatsapp.messagePrefix` (préfixe entrant WhatsApp)
-   Threading des réponses via `replyToMode` et les valeurs par défaut par canal

Détails : [Configuration](../gateway/configuration.md#messages) et documentation des canaux.

[Présence](./presence.md)[Streaming et Fragmentation](./streaming.md)