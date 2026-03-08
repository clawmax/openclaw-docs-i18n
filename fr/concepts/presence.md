

  Multi-agent

  
# Présence

La « présence » OpenClaw est une vue légère et au mieux-effort de :

-   la **Passerelle** elle-même, et
-   **des clients connectés à la Passerelle** (application mac, WebChat, CLI, etc.)

La présence est principalement utilisée pour afficher l'onglet **Instances** de l'application macOS et fournir une visibilité rapide à l'opérateur.

## Champs de présence (ce qui apparaît)

Les entrées de présence sont des objets structurés avec des champs comme :

-   `instanceId` (optionnel mais fortement recommandé) : identité stable du client (généralement `connect.client.instanceId`)
-   `host` : nom d'hôte convivial
-   `ip` : adresse IP au mieux-effort
-   `version` : chaîne de version du client
-   `deviceFamily` / `modelIdentifier` : indices matériels
-   `mode` : `ui`, `webchat`, `cli`, `backend`, `probe`, `test`, `node`, …
-   `lastInputSeconds` : « secondes depuis la dernière saisie utilisateur » (si connu)
-   `reason` : `self`, `connect`, `node-connected`, `periodic`, …
-   `ts` : horodatage de la dernière mise à jour (ms depuis l'époque)

## Producteurs (d'où vient la présence)

Les entrées de présence sont produites par plusieurs sources et **fusionnées**.

### 1) Entrée « self » de la Passerelle

La Passerelle crée toujours une entrée « self » au démarrage afin que les interfaces utilisateur affichent l'hôte de la passerelle même avant qu'aucun client ne se connecte.

### 2) Connexion WebSocket

Chaque client WS commence par une requête `connect`. Lors de la poignée de main réussie, la Passerelle met à jour ou insère une entrée de présence pour cette connexion.

#### Pourquoi les commandes CLI ponctuelles n'apparaissent pas

La CLI se connecte souvent pour des commandes ponctuelles et courtes. Pour éviter de polluer la liste des Instances, `client.mode === "cli"` n'est **pas** transformé en entrée de présence.

### 3) Balises system-event

Les clients peuvent envoyer des balises périodiques plus riches via la méthode `system-event`. L'application mac l'utilise pour rapporter le nom d'hôte, l'adresse IP et `lastInputSeconds`.

### 4) Connexions de nœuds (role: node)

Lorsqu'un nœud se connecte via le WebSocket de la Passerelle avec `role: node`, la Passerelle met à jour ou insère une entrée de présence pour ce nœud (même flux que les autres clients WS).

## Règles de fusion + déduplication (pourquoi instanceId est important)

Les entrées de présence sont stockées dans une seule carte en mémoire :

-   Les entrées sont indexées par une **clé de présence**.
-   La meilleure clé est un `instanceId` stable (provenant de `connect.client.instanceId`) qui survit aux redémarrages.
-   Les clés sont insensibles à la casse.

Si un client se reconnecte sans un `instanceId` stable, il peut apparaître comme une ligne **dupliquée**.

## TTL et taille limitée

La présence est intentionnellement éphémère :

-   **TTL :** les entrées vieilles de plus de 5 minutes sont supprimées
-   **Nombre maximum d'entrées :** 200 (les plus anciennes sont supprimées en premier)

Cela maintient la liste à jour et évite une croissance illimitée de la mémoire.

## Mise en garde tunnel/distant (IP de bouclage)

Lorsqu'un client se connecte via un tunnel SSH / une redirection de port local, la Passerelle peut voir l'adresse distante comme `127.0.0.1`. Pour éviter d'écraser une bonne adresse IP rapportée par le client, les adresses distantes de bouclage sont ignorées.

## Consommateurs

### Onglet Instances de macOS

L'application macOS affiche le résultat de `system-presence` et applique un petit indicateur de statut (Actif/Inactif/Stale) basé sur l'ancienneté de la dernière mise à jour.

## Conseils de débogage

-   Pour voir la liste brute, appelez `system-presence` sur la Passerelle.
-   Si vous voyez des doublons :
    -   vérifiez que les clients envoient un `client.instanceId` stable lors de la poignée de main
    -   vérifiez que les balises périodiques utilisent le même `instanceId`
    -   vérifiez si l'entrée dérivée de la connexion manque d'`instanceId` (les doublons sont alors attendus)

[Routage Multi-Agent](./multi-agent.md)[Messages](./messages.md)

---