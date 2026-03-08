

  Fondamentaux

  
# Architecture de la Passerelle

Dernière mise à jour : 2026-01-22

## Vue d'ensemble

-   Une seule **Passerelle** de longue durée possède toutes les surfaces de messagerie (WhatsApp via Baileys, Telegram via grammY, Slack, Discord, Signal, iMessage, WebChat).
-   Les clients du plan de contrôle (application macOS, CLI, interface web, automatisations) se connectent à la Passerelle via **WebSocket** sur l'hôte de liaison configuré (par défaut `127.0.0.1:18789`).
-   Les **Nœuds** (macOS/iOS/Android/sans tête) se connectent également via **WebSocket**, mais déclarent `role: node` avec des capacités/commandes explicites.
-   Une Passerelle par hôte ; c'est le seul endroit qui ouvre une session WhatsApp.
-   L'**hôte du canevas** est servi par le serveur HTTP de la Passerelle sous :
    -   `/__openclaw__/canvas/` (HTML/CSS/JS modifiable par l'agent)
    -   `/__openclaw__/a2ui/` (hôte A2UI) Il utilise le même port que la Passerelle (par défaut `18789`).

## Composants et flux

### Passerelle (démon)

-   Maintient les connexions aux fournisseurs.
-   Expose une API WS typée (requêtes, réponses, événements poussés par le serveur).
-   Valide les trames entrantes par rapport au schéma JSON.
-   Émet des événements comme `agent`, `chat`, `presence`, `health`, `heartbeat`, `cron`.

### Clients (application macOS / CLI / administration web)

-   Une connexion WS par client.
-   Envoient des requêtes (`health`, `status`, `send`, `agent`, `system-presence`).
-   Souscrivent aux événements (`tick`, `agent`, `presence`, `shutdown`).

### Nœuds (macOS / iOS / Android / sans tête)

-   Se connectent au **même serveur WS** avec `role: node`.
-   Fournissent une identité d'appareil dans `connect` ; l'appairage est **basé sur l'appareil** (rôle `node`) et l'approbation réside dans le magasin d'appairage de l'appareil.
-   Exposent des commandes comme `canvas.*`, `camera.*`, `screen.record`, `location.get`.

Détails du protocole :

-   [Protocole de la passerelle](../gateway/protocol.md)

### WebChat

-   Interface utilisateur statique qui utilise l'API WS de la Passerelle pour l'historique des discussions et l'envoi de messages.
-   Dans les configurations distantes, se connecte via le même tunnel SSH/Tailscale que les autres clients.

## Cycle de vie de la connexion (client unique)

## Protocole filaire (résumé)

-   Transport : WebSocket, trames texte avec des charges utiles JSON.
-   La première trame **doit** être `connect`.
-   Après la poignée de main :
    -   Requêtes : `{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
    -   Événements : `{type:"event", event, payload, seq?, stateVersion?}`
-   Si `OPENCLAW_GATEWAY_TOKEN` (ou `--token`) est défini, `connect.params.auth.token` doit correspondre ou la socket se ferme.
-   Les clés d'idempotence sont requises pour les méthodes à effet de bord (`send`, `agent`) afin de permettre des nouvelles tentatives sûres ; le serveur conserve un cache de déduplication de courte durée.
-   Les nœuds doivent inclure `role: "node"` ainsi que des capacités/commandes/autorisations dans `connect`.

## Appairage + confiance locale

-   Tous les clients WS (opérateurs + nœuds) incluent une **identité d'appareil** dans `connect`.
-   Les nouveaux identifiants d'appareil nécessitent une approbation d'appairage ; la Passerelle émet un **jeton d'appareil** pour les connexions ultérieures.
-   Les connexions **locales** (boucle locale ou l'adresse tailnet propre à l'hôte de la passerelle) peuvent être approuvées automatiquement pour maintenir une expérience utilisateur fluide sur le même hôte.
-   Toutes les connexions doivent signer le nonce `connect.challenge`.
-   La charge utile de signature `v3` lie également `platform` + `deviceFamily` ; la passerelle épingle les métadonnées appairées lors de la reconnexion et exige une réparation de l'appairage pour les changements de métadonnées.
-   Les connexions **non locales** nécessitent toujours une approbation explicite.
-   L'authentification de la passerelle (`gateway.auth.*`) s'applique toujours à **toutes** les connexions, locales ou distantes.

Détails : [Protocole de la passerelle](../gateway/protocol.md), [Appairage](../channels/pairing.md), [Sécurité](../gateway/security.md).

## Typage du protocole et génération de code

-   Les schémas TypeBox définissent le protocole.
-   Le schéma JSON est généré à partir de ces schémas.
-   Les modèles Swift sont générés à partir du schéma JSON.

## Accès distant

-   Préféré : Tailscale ou VPN.
-   Alternative : Tunnel SSH
    
    Copier
    
    ```bash
    ssh -N -L 18789:127.0.0.1:18789 user@host
    ```
    
-   La même poignée de main + jeton d'authentification s'appliquent via le tunnel.
-   TLS + épinglage optionnel peuvent être activés pour WS dans les configurations distantes.

## Instantané des opérations

-   Démarrer : `openclaw gateway` (premier plan, journaux vers stdout).
-   Santé : `health` via WS (également inclus dans `hello-ok`).
-   Supervision : launchd/systemd pour le redémarrage automatique.

## Invariants

-   Exactement une Passerelle contrôle une seule session Baileys par hôte.
-   La poignée de main est obligatoire ; toute première trame non-JSON ou non-connect entraîne une fermeture immédiate.
-   Les événements ne sont pas rejoués ; les clients doivent actualiser en cas d'interruption.

[Architecture d'intégration Pi](../pi.md)[Exécution de l'agent](./agent.md)