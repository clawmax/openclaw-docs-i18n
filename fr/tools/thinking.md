

  Outils intégrés

  
# Niveaux de Réflexion

## Fonctionnement

-   Directive en ligne dans tout corps de message entrant : `/t `, `/think:`, ou `/thinking `.
-   Niveaux (alias) : `off | minimal | low | medium | high | xhigh | adaptive`
    -   minimal → “think”
    -   low → “think hard”
    -   medium → “think harder”
    -   high → “ultrathink” (budget max)
    -   xhigh → “ultrathink+” (modèles GPT-5.2 + Codex uniquement)
    -   adaptive → budget de raisonnement adaptatif géré par le fournisseur (pris en charge pour la famille de modèles Anthropic Claude 4.6)
    -   `x-high`, `x_high`, `extra-high`, `extra high`, et `extra_high` correspondent à `xhigh`.
    -   `highest`, `max` correspondent à `high`.
-   Notes par fournisseur :
    -   Les modèles Anthropic Claude 4.6 utilisent par défaut `adaptive` lorsqu'aucun niveau de réflexion explicite n'est défini.
    -   Z.AI (`zai/*`) ne prend en charge que la réflexion binaire (`on`/`off`). Tout niveau autre que `off` est traité comme `on` (mappé sur `low`).
    -   Moonshot (`moonshot/*`) mappe `/think off` sur `thinking: { type: "disabled" }` et tout niveau autre que `off` sur `thinking: { type: "enabled" }`. Lorsque la réflexion est activée, Moonshot n'accepte que `tool_choice` `auto|none` ; OpenClaw normalise les valeurs incompatibles vers `auto`.

## Ordre de résolution

1.  Directive en ligne sur le message (s'applique uniquement à ce message).
2.  Remplacement de session (défini en envoyant un message contenant uniquement la directive).
3.  Valeur par défaut globale (`agents.defaults.thinkingDefault` dans la configuration).
4.  Solution de repli : `adaptive` pour les modèles Anthropic Claude 4.6, `low` pour les autres modèles capables de raisonnement, `off` sinon.

## Définir une valeur par défaut de session

-   Envoyez un message qui est **uniquement** la directive (les espaces sont autorisés), par exemple `/think:medium` ou `/t high`.
-   Cela reste actif pour la session en cours (par expéditeur par défaut) ; effacé par `/think:off` ou une réinitialisation d'inactivité de session.
-   Une réponse de confirmation est envoyée (`Niveau de réflexion défini sur high.` / `Réflexion désactivée.`). Si le niveau est invalide (par exemple `/thinking big`), la commande est rejetée avec un indice et l'état de la session reste inchangé.
-   Envoyez `/think` (ou `/think:`) sans argument pour voir le niveau de réflexion actuel.

## Application par agent

-   **Embedded Pi** : le niveau résolu est transmis à l'environnement d'exécution de l'agent Pi en cours de traitement.

## Directives verbeuses (/verbose ou /v)

-   Niveaux : `on` (minimal) | `full` | `off` (par défaut).
-   Un message contenant uniquement la directive bascule le mode verbeux de la session et répond `Journalisation verbeuse activée.` / `Journalisation verbeuse désactivée.` ; les niveaux invalides renvoient un indice sans changer l'état.
-   `/verbose off` stocke un remplacement de session explicite ; effacez-le via l'interface Sessions en choisissant `inherit`.
-   La directive en ligne n'affecte que ce message ; les valeurs par défaut de session/globale s'appliquent sinon.
-   Envoyez `/verbose` (ou `/verbose:`) sans argument pour voir le niveau verbeux actuel.
-   Lorsque le mode verbeux est activé, les agents qui émettent des résultats d'outils structurés (Pi, autres agents JSON) renvoient chaque appel d'outil sous forme de message propre, uniquement constitué de métadonnées, préfixé par ` <nom-outil>: ` lorsque disponible (chemin/commande). Ces résumés d'outils sont envoyés dès que chaque outil démarre (bulles séparées), et non sous forme de deltas de streaming.
-   Les résumés d'échec d'outil restent visibles en mode normal, mais les suffixes de détails d'erreur bruts sont masqués sauf si le mode verbeux est `on` ou `full`.
-   Lorsque le mode verbeux est `full`, les sorties d'outils sont également transmises après leur achèvement (bulle séparée, tronquée à une longueur sûre). Si vous basculez `/verbose on|full|off` pendant l'exécution d'une tâche, les bulles d'outils suivantes respectent le nouveau paramètre.

## Visibilité du raisonnement (/reasoning)

-   Niveaux : `on|off|stream`.
-   Un message contenant uniquement la directive bascule l'affichage des blocs de raisonnement dans les réponses.
-   Lorsqu'il est activé, le raisonnement est envoyé sous forme de **message séparé** préfixé par `Reasoning:`.
-   `stream` (Telegram uniquement) : diffuse le raisonnement dans la bulle de brouillon Telegram pendant la génération de la réponse, puis envoie la réponse finale sans le raisonnement.
-   Alias : `/reason`.
-   Envoyez `/reasoning` (ou `/reasoning:`) sans argument pour voir le niveau de raisonnement actuel.

## Liens connexes

-   La documentation du mode élevé se trouve dans [Mode élevé](./elevated.md).

## Pulsations (Heartbeats)

-   Le corps de la sonde de pulsation est l'invite configurée (par défaut : `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`). Les directives en ligne dans un message de pulsation s'appliquent comme d'habitude (mais évitez de changer les paramètres par défaut de session depuis les pulsations).
-   La livraison des pulsations utilise par défaut uniquement la charge utile finale. Pour envoyer également le message `Reasoning:` séparé (lorsqu'il est disponible), définissez `agents.defaults.heartbeat.includeReasoning: true` ou par agent `agents.list[].heartbeat.includeReasoning: true`.

## Interface de discussion web

-   Le sélecteur de réflexion dans le chat web reflète le niveau stocké de la session depuis le stockage/configuration de session entrant lors du chargement de la page.
-   Choisir un autre niveau s'applique uniquement au message suivant (`thinkingOnce`) ; après l'envoi, le sélecteur revient au niveau de session stocké.
-   Pour changer la valeur par défaut de la session, envoyez une directive `/think:` (comme auparavant) ; le sélecteur le reflétera après le prochain rechargement.

[Reactions](./reactions.md)[Outils Web](./web.md)