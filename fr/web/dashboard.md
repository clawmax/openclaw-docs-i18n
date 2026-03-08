

  Interfaces web

  
# Tableau de bord

Le tableau de bord de la passerelle est l'interface de contrôle (Control UI) accessible via un navigateur et servie par défaut à l'URL `/` (peut être modifiée avec `gateway.controlUi.basePath`). Accès rapide (passerelle locale) :

-   [http://127.0.0.1:18789/](http://127.0.0.1:18789/) (ou [http://localhost:18789/](http://localhost:18789/))

Références clés :

-   [Interface de contrôle](./control-ui.md) pour son utilisation et ses fonctionnalités.
-   [Tailscale](../gateway/tailscale.md) pour l'automatisation Serve/Funnel.
-   [Surfaces web](../web.md) pour les modes de liaison et les notes de sécurité.

L'authentification est appliquée lors de la poignée de main WebSocket via `connect.params.auth` (jeton ou mot de passe). Voir `gateway.auth` dans la [configuration de la passerelle](../gateway/configuration.md). Note de sécurité : l'interface de contrôle est une **surface d'administration** (chat, configuration, approbations d'exécution). Ne l'exposez pas publiquement. L'interface conserve les jetons d'URL du tableau de bord en mémoire pour l'onglet courant et les retire de l'URL après le chargement. Privilégiez l'accès localhost, Tailscale Serve, ou un tunnel SSH.

## Chemin rapide (recommandé)

-   Après l'intégration, l'interface CLI ouvre automatiquement le tableau de bord et affiche un lien propre (sans jeton).
-   Pour le rouvrir à tout moment : `openclaw dashboard` (copie le lien, ouvre le navigateur si possible, affiche un indice SSH si en mode sans tête).
-   Si l'interface demande une authentification, collez le jeton de `gateway.auth.token` (ou `OPENCLAW_GATEWAY_TOKEN`) dans les paramètres de l'interface de contrôle.

## Bases du jeton (local vs distant)

-   **Localhost** : ouvrez `http://127.0.0.1:18789/`.
-   **Source du jeton** : `gateway.auth.token` (ou `OPENCLAW_GATEWAY_TOKEN`) ; `openclaw dashboard` peut le transmettre via le fragment d'URL pour un amorçage unique, mais l'interface de contrôle ne conserve pas les jetons de la passerelle dans le localStorage.
-   Si `gateway.auth.token` est géré par SecretRef, `openclaw dashboard` imprime/copie/ouvre une URL sans jeton par conception. Cela évite d'exposer des jetons gérés extérieurement dans les journaux du shell, l'historique du presse-papiers ou les arguments de lancement du navigateur.
-   Si `gateway.auth.token` est configuré comme un SecretRef et n'est pas résolu dans votre shell actuel, `openclaw dashboard` affiche tout de même une URL sans jeton ainsi que des instructions pratiques pour configurer l'authentification.
-   **Pas en localhost** : utilisez Tailscale Serve (sans jeton pour l'interface de contrôle/WebSocket si `gateway.auth.allowTailscale: true`, suppose un hôte de passerelle de confiance ; les API HTTP nécessitent toujours un jeton/mot de passe), une liaison tailnet avec un jeton, ou un tunnel SSH. Voir [Surfaces web](../web.md).

## Si vous voyez "unauthorized" / 1008

-   Assurez-vous que la passerelle est joignable (local : `openclaw status` ; distant : tunnel SSH `ssh -N -L 18789:127.0.0.1:18789 user@host` puis ouvrez `http://127.0.0.1:18789/`).
-   Récupérez ou fournissez le jeton depuis l'hôte de la passerelle :
    -   Configuration en texte clair : `openclaw config get gateway.auth.token`
    -   Configuration gérée par SecretRef : résolvez le fournisseur de secret externe ou exportez `OPENCLAW_GATEWAY_TOKEN` dans ce shell, puis relancez `openclaw dashboard`
    -   Aucun jeton configuré : `openclaw doctor --generate-gateway-token`
-   Dans les paramètres du tableau de bord, collez le jeton dans le champ d'authentification, puis connectez-vous.

[Interface de contrôle](./control-ui.md)[WebChat](./webchat.md)

---