

  Aperçu des plateformes

  
# Application Android

## Aperçu du support

-   Rôle : application nœud compagnon (Android n'héberge pas la passerelle).
-   Passerelle requise : oui (exécutez-la sur macOS, Linux ou Windows via WSL2).
-   Installation : [Démarrage](../start/getting-started.md) + [Appairage](../channels/pairing.md).
-   Passerelle : [Guide d'exploitation](../gateway.md) + [Configuration](../gateway/configuration.md).
    -   Protocoles : [Protocole de la passerelle](../gateway/protocol.md) (nœuds + plan de contrôle).

## Contrôle système

Le contrôle système (launchd/systemd) réside sur l'hôte de la passerelle. Voir [Passerelle](../gateway.md).

## Guide de connexion

Application nœud Android ⇄ (mDNS/NSD + WebSocket) ⇄ **Passerelle** Android se connecte directement au WebSocket de la passerelle (par défaut `ws://:18789`) et utilise l'appairage d'appareil (`role: node`).

### Prérequis

-   Vous pouvez exécuter la passerelle sur la machine « maître ».
-   L'appareil/émulateur Android peut atteindre le WebSocket de la passerelle :
    -   Même LAN avec mDNS/NSD, **ou**
    -   Même tailnet Tailscale en utilisant Wide-Area Bonjour / DNS-SD unicast (voir ci-dessous), **ou**
    -   Hôte/port manuel de la passerelle (solution de secours)
-   Vous pouvez exécuter la CLI (`openclaw`) sur la machine de la passerelle (ou via SSH).

### 1) Démarrer la passerelle

```bash
openclaw gateway --port 18789 --verbose
```

Confirmez dans les logs que vous voyez quelque chose comme :

-   `listening on ws://0.0.0.0:18789`

Pour les configurations tailnet uniquement (recommandé pour Vienne ⇄ Londres), liez la passerelle à l'IP du tailnet :

-   Définissez `gateway.bind: "tailnet"` dans `~/.openclaw/openclaw.json` sur l'hôte de la passerelle.
-   Redémarrez la passerelle / l'application de la barre de menus macOS.

### 2) Vérifier la découverte (optionnel)

Depuis la machine de la passerelle :

```bash
dns-sd -B _openclaw-gw._tcp local.
```

Plus de notes de débogage : [Bonjour](../gateway/bonjour.md).

#### Découverte via DNS-SD unicast sur Tailnet (Vienne ⇄ Londres)

La découverte NSD/mDNS d'Android ne traverse pas les réseaux. Si votre nœud Android et la passerelle sont sur des réseaux différents mais connectés via Tailscale, utilisez Wide-Area Bonjour / DNS-SD unicast à la place :

1.  Configurez une zone DNS-SD (exemple `openclaw.internal.`) sur l'hôte de la passerelle et publiez les enregistrements `_openclaw-gw._tcp`.
2.  Configurez le DNS fractionné Tailscale pour votre domaine choisi pointant vers ce serveur DNS.

Détails et exemple de configuration CoreDNS : [Bonjour](../gateway/bonjour.md).

### 3) Se connecter depuis Android

Dans l'application Android :

-   L'application maintient sa connexion à la passerelle via un **service au premier plan** (notification persistante).
-   Ouvrez l'onglet **Connexion**.
-   Utilisez le **Code de configuration** ou le mode **Manuel**.
-   Si la découverte est bloquée, utilisez l'hôte/port manuel (et TLS/jeton/mot de passe si requis) dans les **Contrôles avancés**.

Après le premier appairage réussi, Android se reconnecte automatiquement au lancement :

-   Point de terminaison manuel (si activé), sinon
-   La dernière passerelle découverte (au mieux).

### 4) Approuver l'appairage (CLI)

Sur la machine de la passerelle :

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
```

Détails de l'appairage : [Appairage](../channels/pairing.md).

### 5) Vérifier que le nœud est connecté

-   Via le statut des nœuds :
    
    Copier
    
    ```bash
    openclaw nodes status
    ```
    
-   Via la passerelle :
    
    Copier
    
    ```bash
    openclaw gateway call node.list --params "{}"
    ```
    

### 6) Chat + historique

L'onglet Chat Android prend en charge la sélection de session (par défaut `main`, plus d'autres sessions existantes) :

-   Historique : `chat.history`
-   Envoyer : `chat.send`
-   Mises à jour push (au mieux) : `chat.subscribe` → `event:"chat"`

### 7) Canevas + écran + appareil photo

#### Hôte de canevas de la passerelle (recommandé pour le contenu web)

Si vous voulez que le nœud affiche du vrai HTML/CSS/JS que l'agent peut modifier sur le disque, pointez le nœud vers l'hôte de canevas de la passerelle. Note : les nœuds chargent le canevas depuis le serveur HTTP de la passerelle (même port que `gateway.port`, par défaut `18789`).

1.  Créez `~/.openclaw/workspace/canvas/index.html` sur l'hôte de la passerelle.
2.  Naviguez le nœud vers celui-ci (LAN) :

```bash
openclaw nodes invoke --node "<Android Node>" --command canvas.navigate --params '{"url":"http://<gateway-hostname>.local:18789/__openclaw__/canvas/"}'
```

Tailnet (optionnel) : si les deux appareils sont sur Tailscale, utilisez un nom MagicDNS ou une IP tailnet au lieu de `.local`, par ex. `http://<gateway-magicdns>:18789/__openclaw__/canvas/`. Ce serveur injecte un client de rechargement en direct dans le HTML et recharge lors des changements de fichiers. L'hôte A2UI se trouve à `http://<gateway-host>:18789/__openclaw__/a2ui/`. Commandes de canevas (premier plan uniquement) :

-   `canvas.eval`, `canvas.snapshot`, `canvas.navigate` (utilisez `{"url":""}` ou `{"url":"/"}` pour revenir à l'échafaudage par défaut). `canvas.snapshot` retourne `{ format, base64 }` (par défaut `format="jpeg"`).
-   A2UI : `canvas.a2ui.push`, `canvas.a2ui.reset` (`canvas.a2ui.pushJSONL` alias historique)

Commandes appareil photo (premier plan uniquement ; soumis aux permissions) :

-   `camera.snap` (jpg)
-   `camera.clip` (mp4)

Voir [Nœud appareil photo](../nodes/camera.md) pour les paramètres et les aides CLI. Commandes écran :

-   `screen.record` (mp4 ; premier plan uniquement)

### 8) Voix + surface de commande Android étendue

-   Voix : Android utilise un flux unique d'activation/désactivation du micro dans l'onglet Voix avec capture de transcription et lecture TTS (ElevenLabs lorsqu'il est configuré, TTS système en secours).
-   Les bascules de réveil/mode conversation vocale sont actuellement retirées de l'UX/runtime Android.
-   Familles de commandes Android supplémentaires (disponibilité dépend de l'appareil + permissions) :
    -   `device.status`, `device.info`, `device.permissions`, `device.health`
    -   `notifications.list`, `notifications.actions`
    -   `photos.latest`
    -   `contacts.search`, `contacts.add`
    -   `calendar.events`, `calendar.add`
    -   `motion.activity`, `motion.pedometer`
    -   `app.update`

[Windows (WSL2)](./windows.md)[Application iOS](./ios.md)