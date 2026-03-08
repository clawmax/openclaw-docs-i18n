

  Réseau et découverte

  
# Découverte et Transports

OpenClaw a deux problèmes distincts qui semblent similaires en surface :

1.  **Contrôle à distance de l'opérateur** : l'application de la barre de menus macOS contrôlant une passerelle exécutée ailleurs.
2.  **Appairage des nœuds** : les nœuds iOS/Android (et futurs) trouvant une passerelle et s'appairant de manière sécurisée.

L'objectif de conception est de garder toute la découverte/publicité réseau dans la **Passerelle de Nœud** (`openclaw gateway`) et de garder les clients (application mac, iOS) comme consommateurs.

## Termes

-   **Passerelle** : un processus de passerelle unique de longue durée qui possède l'état (sessions, appairage, registre des nœuds) et exécute des canaux. La plupart des configurations en utilisent une par hôte ; des configurations multi-passerelles isolées sont possibles.
-   **Passerelle WS (plan de contrôle)** : le point de terminaison WebSocket sur `127.0.0.1:18789` par défaut ; peut être lié au LAN/tailnet via `gateway.bind`.
-   **Transport WS direct** : un point de terminaison Passerelle WS accessible sur le LAN/tailnet (sans SSH).
-   **Transport SSH (repli)** : contrôle à distance en redirigeant `127.0.0.1:18789` via SSH.
-   **Pont TCP hérité (obsolète/supprimé)** : ancien transport de nœud (voir [Protocole de pont](./bridge-protocol.md)) ; n'est plus publié pour la découverte.

Détails des protocoles :

-   [Protocole de la passerelle](./protocol.md)
-   [Protocole de pont (hérité)](./bridge-protocol.md)

## Pourquoi nous conservons à la fois le mode "direct" et SSH

-   **WS direct** offre la meilleure expérience utilisateur sur le même réseau et au sein d'un tailnet :
    -   auto-découverte sur le LAN via Bonjour
    -   jetons d'appairage + ACLs gérés par la passerelle
    -   aucun accès shell requis ; la surface du protocole peut rester restreinte et vérifiable
-   **SSH** reste la solution de repli universelle :
    -   fonctionne partout où vous avez un accès SSH (même à travers des réseaux non liés)
    -   survit aux problèmes multicast/mDNS
    -   ne nécessite aucun nouveau port entrant en plus de SSH

## Entrées de découverte (comment les clients apprennent où se trouve la passerelle)

### 1) Bonjour / mDNS (LAN uniquement)

Bonjour est basé sur le meilleur effort et ne traverse pas les réseaux. Il est utilisé uniquement pour le confort "même LAN". Cible directionnelle :

-   La **passerelle** publie son point de terminaison WS via Bonjour.
-   Les clients parcourent et affichent une liste "choisir une passerelle", puis stockent le point de terminaison choisi.

Dépannage et détails du balise : [Bonjour](./bonjour.md).

#### Détails du balise de service

-   Types de service :
    -   `_openclaw-gw._tcp` (balise de transport de passerelle)
-   Clés TXT (non secrètes) :
    -   `role=gateway`
    -   `lanHost=.local`
    -   `sshPort=22` (ou tout ce qui est publié)
    -   `gatewayPort=18789` (Passerelle WS + HTTP)
    -   `gatewayTls=1` (uniquement lorsque TLS est activé)
    -   `gatewayTlsSha256=` (uniquement lorsque TLS est activé et que l'empreinte est disponible)
    -   `canvasPort=` (port hôte du canevas ; actuellement le même que `gatewayPort` lorsque l'hôte du canevas est activé)
    -   `cliPath=` (optionnel ; chemin absolu vers un point d'entrée ou un binaire exécutable `openclaw`)
    -   `tailnetDns=` (indice optionnel ; détecté automatiquement lorsque Tailscale est disponible)

Notes de sécurité :

-   Les enregistrements TXT Bonjour/mDNS sont **non authentifiés**. Les clients doivent traiter les valeurs TXT uniquement comme des indices pour l'expérience utilisateur.
-   Le routage (hôte/port) doit préférer le **point de terminaison de service résolu** (SRV + A/AAAA) plutôt que le `lanHost`, `tailnetDns` ou `gatewayPort` fourni par TXT.
-   L'épinglage TLS ne doit jamais permettre à un `gatewayTlsSha256` publié de remplacer un épinglage précédemment stocké.
-   Les nœuds iOS/Android doivent traiter les connexions directes basées sur la découverte comme **TLS uniquement** et exiger une confirmation explicite "faire confiance à cette empreinte" avant de stocker un épinglage pour la première fois (vérification hors bande).

Désactiver/remplacer :

-   `OPENCLAW_DISABLE_BONJOUR=1` désactive la publication.
-   `gateway.bind` dans `~/.openclaw/openclaw.json` contrôle le mode de liaison de la Passerelle.
-   `OPENCLAW_SSH_PORT` remplace le port SSH publié dans TXT (par défaut 22).
-   `OPENCLAW_TAILNET_DNS` publie un indice `tailnetDns` (MagicDNS).
-   `OPENCLAW_CLI_PATH` remplace le chemin CLI publié.

### 2) Tailnet (inter-réseaux)

Pour les configurations de type Londres/Vienne, Bonjour n'aidera pas. La cible "directe" recommandée est :

-   Le nom Tailscale MagicDNS (préféré) ou une adresse IP stable sur le tailnet.

Si la passerelle peut détecter qu'elle s'exécute sous Tailscale, elle publie `tailnetDns` comme un indice optionnel pour les clients (y compris les balises à grande échelle).

### 3) Cible manuelle / SSH

Lorsqu'il n'y a pas de route directe (ou que le mode direct est désactivé), les clients peuvent toujours se connecter via SSH en redirigeant le port de la passerelle en boucle locale. Voir [Accès à distance](./remote.md).

## Sélection du transport (politique client)

Comportement client recommandé :

1.  Si un point de terminaison direct appairé est configuré et accessible, l'utiliser.
2.  Sinon, si Bonjour trouve une passerelle sur le LAN, proposer un choix "Utiliser cette passerelle" en un clic et l'enregistrer comme point de terminaison direct.
3.  Sinon, si un DNS/IP tailnet est configuré, essayer le mode direct.
4.  Sinon, revenir à SSH.

## Appairage + authentification (transport direct)

La passerelle est la source de vérité pour l'admission des nœuds/clients.

-   Les demandes d'appairage sont créées/approuvées/rejetées dans la passerelle (voir [Appairage de la passerelle](./pairing.md)).
-   La passerelle applique :
    -   l'authentification (jeton / paire de clés)
    -   les portées/ACL (la passerelle n'est pas un proxy brut vers chaque méthode)
    -   les limites de débit

## Responsabilités par composant

-   **Passerelle** : publie les balises de découverte, prend les décisions d'appairage et héberge le point de terminaison WS.
-   **Application macOS** : vous aide à choisir une passerelle, affiche les invites d'appairage et n'utilise SSH qu'en dernier recours.
-   **Nœuds iOS/Android** : parcourent Bonjour par commodité et se connectent à la Passerelle WS appairée.

[Appairage Géré par la Passerelle](./pairing.md)[Découverte Bonjour](./bonjour.md)