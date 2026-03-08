

  Réseau et découverte

  
# Découverte Bonjour

OpenClaw utilise Bonjour (mDNS / DNS‑SD) comme **commodité réservée au LAN** pour découvrir une passerelle active (point de terminaison WebSocket). C'est une méthode au mieux et ne **remplace pas** la connectivité basée sur SSH ou le Tailnet.

## Bonjour étendu (DNS‑SD Unicast) via Tailscale

Si le nœud et la passerelle sont sur des réseaux différents, le mDNS multicast ne traversera pas la frontière. Vous pouvez conserver la même expérience de découverte en passant au **DNS‑SD unicast** ("Bonjour étendu") via Tailscale. Étapes principales :

1.  Exécutez un serveur DNS sur l'hôte de la passerelle (accessible via le Tailnet).
2.  Publiez les enregistrements DNS‑SD pour `_openclaw-gw._tcp` sous une zone dédiée (exemple : `openclaw.internal.`).
3.  Configurez le **DNS fractionné** de Tailscale pour que votre domaine choisi soit résolu via ce serveur DNS pour les clients (y compris iOS).

OpenClaw prend en charge n'importe quel domaine de découverte ; `openclaw.internal.` n'est qu'un exemple. Les nœuds iOS/Android parcourent à la fois `local.` et votre domaine étendu configuré.

### Configuration de la passerelle (recommandée)

```json
{
  gateway: { bind: "tailnet" }, // tailnet uniquement (recommandé)
  discovery: { wideArea: { enabled: true } }, // active la publication DNS-SD étendue
}
```

### Configuration unique du serveur DNS (hôte de la passerelle)

```bash
openclaw dns setup --apply
```

Ceci installe CoreDNS et le configure pour :

-   écouter sur le port 53 uniquement sur les interfaces Tailscale de la passerelle
-   servir votre domaine choisi (exemple : `openclaw.internal.`) depuis `~/.openclaw/dns/.db`

Validez depuis une machine connectée au tailnet :

```bash
dns-sd -B _openclaw-gw._tcp openclaw.internal.
dig @<TAILNET_IPV4> -p 53 _openclaw-gw._tcp.openclaw.internal PTR +short
```

### Paramètres DNS Tailscale

Dans la console d'administration Tailscale :

-   Ajoutez un serveur de noms pointant vers l'IP tailnet de la passerelle (UDP/TCP 53).
-   Ajoutez un DNS fractionné pour que votre domaine de découverte utilise ce serveur de noms.

Une fois que les clients acceptent le DNS tailnet, les nœuds iOS peuvent parcourir `_openclaw-gw._tcp` dans votre domaine de découverte sans multicast.

### Sécurité de l'écouteur de la passerelle (recommandée)

Le port WS de la passerelle (par défaut `18789`) est lié à la boucle locale par défaut. Pour un accès LAN/tailnet, liez-le explicitement et gardez l'authentification activée. Pour les configurations tailnet uniquement :

-   Définissez `gateway.bind: "tailnet"` dans `~/.openclaw/openclaw.json`.
-   Redémarrez la passerelle (ou redémarrez l'application de la barre de menus macOS).

## Ce qui est annoncé

Seule la passerelle annonce `_openclaw-gw._tcp`.

## Types de service

-   `_openclaw-gw._tcp` — balise de transport de la passerelle (utilisée par les nœuds macOS/iOS/Android).

## Clés TXT (indices non secrets)

La passerelle annonce de petits indices non secrets pour faciliter les flux d'interface utilisateur :

-   `role=gateway`
-   `displayName=`
-   `lanHost=.local`
-   `gatewayPort=` (WS + HTTP de la passerelle)
-   `gatewayTls=1` (uniquement lorsque TLS est activé)
-   `gatewayTlsSha256=` (uniquement lorsque TLS est activé et que l'empreinte est disponible)
-   `canvasPort=` (uniquement lorsque l'hôte du canevas est activé ; actuellement le même que `gatewayPort`)
-   `sshPort=` (par défaut 22 si non modifié)
-   `transport=gateway`
-   `cliPath=` (optionnel ; chemin absolu vers un point d'entrée exécutable `openclaw`)
-   `tailnetDns=` (indice optionnel lorsque le Tailnet est disponible)

Notes de sécurité :

-   Les enregistrements TXT Bonjour/mDNS sont **non authentifiés**. Les clients ne doivent pas traiter le TXT comme un routage faisant autorité.
-   Les clients doivent router en utilisant le point de terminaison de service résolu (SRV + A/AAAA). Traitez `lanHost`, `tailnetDns`, `gatewayPort` et `gatewayTlsSha256` comme des indices uniquement.
-   L'épinglage TLS ne doit jamais permettre à un `gatewayTlsSha256` annoncé de remplacer un épinglage précédemment stocké.
-   Les nœuds iOS/Android doivent traiter les connexions directes basées sur la découverte comme **TLS uniquement** et exiger une confirmation explicite de l'utilisateur avant de faire confiance à une empreinte pour la première fois.

## Débogage sur macOS

Outils intégrés utiles :

-   Parcourir les instances :
    
    Copier
    
    ```bash
    dns-sd -B _openclaw-gw._tcp local.
    ```
    
-   Résoudre une instance (remplacez ``) :
    
    Copier
    
    ```bash
    dns-sd -L "<instance>" _openclaw-gw._tcp local.
    ```
    

Si la navigation fonctionne mais que la résolution échoue, vous rencontrez généralement une politique LAN ou un problème de résolveur mDNS.

## Débogage dans les journaux de la passerelle

La passerelle écrit un fichier journal rotatif (affiché au démarrage comme `gateway log file: ...`). Cherchez les lignes `bonjour:`, en particulier :

-   `bonjour: advertise failed ...`
-   `bonjour: ... name conflict resolved` / `hostname conflict resolved`
-   `bonjour: watchdog detected non-announced service ...`

## Débogage sur le nœud iOS

Le nœud iOS utilise `NWBrowser` pour découvrir `_openclaw-gw._tcp`. Pour capturer les journaux :

-   Paramètres → Passerelle → Avancé → **Journaux de débogage de la découverte**
-   Paramètres → Passerelle → Avancé → **Journaux de découverte** → reproduisez → **Copier**

Le journal inclut les transitions d'état du navigateur et les changements d'ensemble de résultats.

## Modes d'échec courants

-   **Bonjour ne traverse pas les réseaux** : utilisez Tailnet ou SSH.
-   **Multicast bloqué** : certains réseaux Wi‑Fi désactivent le mDNS.
-   **Veille / changement d'interface** : macOS peut temporairement abandonner les résultats mDNS ; réessayez.
-   **La navigation fonctionne mais la résolution échoue** : gardez les noms de machine simples (évitez les émojis ou la ponctuation), puis redémarrez la passerelle. Le nom de l'instance de service dérive du nom d'hôte, donc des noms trop complexes peuvent perturber certains résolveurs.

## Noms d'instance échappés (\\032)

Bonjour/DNS‑SD échappe souvent les octets dans les noms d'instance de service sous forme de séquences décimales `\DDD` (par exemple, les espaces deviennent `\032`).

-   C'est normal au niveau du protocole.
-   Les interfaces utilisateur doivent décoder pour l'affichage (iOS utilise `BonjourEscapes.decode`).

## Désactivation / configuration

-   `OPENCLAW_DISABLE_BONJOUR=1` désactive l'annonce (hérité : `OPENCLAW_DISABLE_BONJOUR`).
-   `gateway.bind` dans `~/.openclaw/openclaw.json` contrôle le mode de liaison de la passerelle.
-   `OPENCLAW_SSH_PORT` remplace le port SSH annoncé dans TXT (hérité : `OPENCLAW_SSH_PORT`).
-   `OPENCLAW_TAILNET_DNS` publie un indice MagicDNS dans TXT (hérité : `OPENCLAW_TAILNET_DNS`).
-   `OPENCLAW_CLI_PATH` remplace le chemin CLI annoncé (hérité : `OPENCLAW_CLI_PATH`).

## Documents connexes

-   Politique de découverte et sélection de transport : [Découverte](./discovery.md)
-   Appairage des nœuds + approbations : [Appairage de la passerelle](./pairing.md)

[Découverte et Transports](./discovery.md)[Accès à distance](./remote.md)