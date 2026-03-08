

  Interfaces Web

  
# WebChat

Statut : l'interface utilisateur de chat SwiftUI pour macOS/iOS communique directement avec le WebSocket de la passerelle.

## Ce que c'est

-   Une interface utilisateur de chat native pour la passerelle (pas de navigateur intégré et pas de serveur statique local).
-   Utilise les mêmes sessions et règles de routage que les autres canaux.
-   Routage déterministe : les réponses reviennent toujours à WebChat.

## Démarrage rapide

1.  Démarrez la passerelle.
2.  Ouvrez l'interface utilisateur WebChat (application macOS/iOS) ou l'onglet de chat de l'interface de contrôle.
3.  Assurez-vous que l'authentification de la passerelle est configurée (requise par défaut, même en boucle locale).

## Fonctionnement (comportement)

-   L'interface utilisateur se connecte au WebSocket de la passerelle et utilise `chat.history`, `chat.send` et `chat.inject`.
-   `chat.history` est limité pour la stabilité : la passerelle peut tronquer les champs texte longs, omettre les métadonnées lourdes et remplacer les entrées trop volumineuses par `[chat.history omitted: message too large]`.
-   `chat.inject` ajoute directement une note de l'assistant à la transcription et la diffuse à l'interface utilisateur (pas d'exécution d'agent).
-   Les exécutions interrompues peuvent laisser la sortie partielle de l'assistant visible dans l'interface utilisateur.
-   La passerelle conserve le texte partiel de l'assistant interrompu dans l'historique des transcriptions lorsqu'il existe une sortie mise en tampon, et marque ces entrées avec des métadonnées d'interruption.
-   L'historique est toujours récupéré depuis la passerelle (pas de surveillance de fichier local).
-   Si la passerelle est inaccessible, WebChat est en lecture seule.

## Panneau des outils des agents dans l'interface de contrôle

-   Le panneau Outils `/agents` de l'interface de contrôle récupère un catalogue d'exécution via `tools.catalog` et étiquette chaque outil comme `core` ou `plugin:` (plus `optional` pour les outils optionnels des plugins).
-   Si `tools.catalog` est indisponible, le panneau utilise une liste statique intégrée par défaut.
-   Le panneau modifie la configuration du profil et les dérogations, mais l'accès effectif à l'exécution suit toujours la priorité des politiques (`allow`/`deny`, dérogations par agent et par fournisseur/canal).

## Utilisation à distance

-   Le mode à distance tunnelise le WebSocket de la passerelle via SSH/Tailscale.
-   Vous n'avez pas besoin d'exécuter un serveur WebChat séparé.

## Référence de configuration (WebChat)

Configuration complète : [Configuration](../gateway/configuration.md) Options de canal :

-   Pas de bloc dédié `webchat.*`. WebChat utilise le point de terminaison de la passerelle + les paramètres d'authentification ci-dessous.

Options globales associées :

-   `gateway.port`, `gateway.bind` : hôte/port du WebSocket.
-   `gateway.auth.mode`, `gateway.auth.token`, `gateway.auth.password` : authentification WebSocket (jeton/mot de passe).
-   `gateway.auth.mode: "trusted-proxy"` : authentification par proxy inverse pour les clients navigateurs (voir [Authentification par proxy de confiance](../gateway/trusted-proxy-auth.md)).
-   `gateway.remote.url`, `gateway.remote.token`, `gateway.remote.password` : cible de la passerelle distante.
-   `session.*` : stockage des sessions et clés principales par défaut.

[Dashboard](./dashboard.md)[TUI](./tui.md)

---