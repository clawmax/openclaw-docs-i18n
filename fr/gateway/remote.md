

  Accès à distance

  
# Accès à distance

Ce dépôt prend en charge le « distant via SSH » en maintenant un seul Gateway (le maître) en cours d'exécution sur un hôte dédié (bureau/serveur) et en y connectant les clients.

-   Pour les **opérateurs (vous / l'application macOS)** : le tunneling SSH est la solution de secours universelle.
-   Pour les **nœuds (iOS/Android et futurs appareils)** : connectez-vous au **WebSocket** du Gateway (LAN/tailnet ou tunnel SSH selon les besoins).

## L'idée centrale

-   Le WebSocket du Gateway se lie à **loopback** sur le port configuré (par défaut 18789).
-   Pour une utilisation à distance, vous transmettez ce port loopback via SSH (ou utilisez un tailnet/VPN et tunnel moins).

## Configurations courantes VPN/tailnet (où vit l'agent)

Considérez l'**hôte Gateway** comme « l'endroit où vit l'agent ». Il possède les sessions, profils d'authentification, canaux et état. Votre ordinateur portable/de bureau (et les nœuds) se connectent à cet hôte.

### 1) Gateway permanent dans votre tailnet (VPS ou serveur domestique)

Exécutez le Gateway sur un hôte persistant et atteignez-le via **Tailscale** ou SSH.

-   **Meilleure UX :** conservez `gateway.bind: "loopback"` et utilisez **Tailscale Serve** pour l'interface de contrôle.
-   **Solution de secours :** conservez loopback + tunnel SSH depuis toute machine nécessitant un accès.
-   **Exemples :** [exe.dev](../install/exe-dev.md) (VM facile) ou [Hetzner](../install/hetzner.md) (VPS de production).

C'est idéal lorsque votre ordinateur portable se met souvent en veille mais que vous voulez que l'agent soit toujours actif.

### 2) L'ordinateur de bureau domestique exécute le Gateway, l'ordinateur portable est la télécommande

L'ordinateur portable **n'exécute pas** l'agent. Il se connecte à distance :

-   Utilisez le mode **Distant via SSH** de l'application macOS (Paramètres → Général → « OpenClaw s'exécute »).
-   L'application ouvre et gère le tunnel, donc WebChat + vérifications de santé « fonctionnent simplement ».

Guide opérationnel : [Accès à distance macOS](../platforms/mac/remote.md).

### 3) L'ordinateur portable exécute le Gateway, accès à distance depuis d'autres machines

Gardez le Gateway local mais exposez-le en toute sécurité :

-   Tunnel SSH vers l'ordinateur portable depuis d'autres machines, ou
-   Tailscale Serve l'interface de contrôle et gardez le Gateway en loopback uniquement.

Guide : [Tailscale](./tailscale.md) et [Vue d'ensemble Web](../web.md).

## Flux de commandes (ce qui s'exécute où)

Un service gateway possède l'état + les canaux. Les nœuds sont des périphériques. Exemple de flux (Telegram → nœud) :

-   Un message Telegram arrive au **Gateway**.
-   Le Gateway exécute l'**agent** et décide s'il faut appeler un outil de nœud.
-   Le Gateway appelle le **nœud** via le WebSocket du Gateway (RPC `node.*`).
-   Le nœud renvoie le résultat ; le Gateway répond à Telegram.

Notes :

-   **Les nœuds n'exécutent pas le service gateway.** Un seul gateway doit s'exécuter par hôte, sauf si vous exécutez intentionnellement des profils isolés (voir [Gateways multiples](./multiple-gateways.md)).
-   Le « mode nœud » de l'application macOS est simplement un client nœud via le WebSocket du Gateway.

## Tunnel SSH (CLI + outils)

Créez un tunnel local vers le WS du Gateway distant :

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

Avec le tunnel actif :

-   `openclaw health` et `openclaw status --deep` atteignent maintenant le gateway distant via `ws://127.0.0.1:18789`.
-   `openclaw gateway {status,health,send,agent,call}` peuvent également cibler l'URL transmise via `--url` si nécessaire.

Note : remplacez `18789` par votre `gateway.port` configuré (ou `--port`/`OPENCLAW_GATEWAY_PORT`). Note : lorsque vous passez `--url`, la CLI ne revient pas aux identifiants de configuration ou d'environnement. Incluez `--token` ou `--password` explicitement. L'absence d'identifiants explicites est une erreur.

## Valeurs par défaut CLI à distance

Vous pouvez persister une cible distante pour que les commandes CLI l'utilisent par défaut :

```json
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://127.0.0.1:18789",
      token: "your-token",
    },
  },
}
```

Lorsque le gateway est en loopback uniquement, gardez l'URL à `ws://127.0.0.1:18789` et ouvrez d'abord le tunnel SSH.

## Priorité des identifiants

La résolution des identifiants d'appel/sonde du Gateway suit maintenant un contrat partagé :

-   Les identifiants explicites (`--token`, `--password`, ou `gatewayToken` de l'outil) l'emportent toujours.
-   Valeurs par défaut du mode local :
    -   token : `OPENCLAW_GATEWAY_TOKEN` -> `gateway.auth.token` -> `gateway.remote.token`
    -   password : `OPENCLAW_GATEWAY_PASSWORD` -> `gateway.auth.password` -> `gateway.remote.password`
-   Valeurs par défaut du mode distant :
    -   token : `gateway.remote.token` -> `OPENCLAW_GATEWAY_TOKEN` -> `gateway.auth.token`
    -   password : `OPENCLAW_GATEWAY_PASSWORD` -> `gateway.remote.password` -> `gateway.auth.password`
-   Les vérifications de token de sonde/statut distant sont strictes par défaut : elles utilisent uniquement `gateway.remote.token` (pas de secours avec le token local) lors du ciblage du mode distant.
-   Les variables d'environnement héritées `CLAWDBOT_GATEWAY_*` ne sont utilisées que par les chemins d'appel de compatibilité ; la résolution de sonde/statut/auth utilise uniquement `OPENCLAW_GATEWAY_*`.

## Interface de chat via SSH

WebChat n'utilise plus de port HTTP séparé. L'interface de chat SwiftUI se connecte directement au WebSocket du Gateway.

-   Transmettez `18789` via SSH (voir ci-dessus), puis connectez les clients à `ws://127.0.0.1:18789`.
-   Sur macOS, préférez le mode « Distant via SSH » de l'application, qui gère le tunnel automatiquement.

## Application macOS « Distant via SSH »

L'application de la barre de menus macOS peut piloter la même configuration de bout en bout (vérifications d'état à distance, WebChat et transmission Voice Wake). Guide opérationnel : [Accès à distance macOS](../platforms/mac/remote.md).

## Règles de sécurité (distant/VPN)

Version courte : **gardez le Gateway en loopback uniquement** sauf si vous êtes sûr de besoin d'une liaison.

-   **Loopback + SSH/Tailscale Serve** est l'option par défaut la plus sûre (pas d'exposition publique).
-   Le texte clair `ws://` est en loopback uniquement par défaut. Pour les réseaux privés de confiance, définissez `OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1` sur le processus client comme solution d'urgence.
-   Les **liaisons non-loopback** (`lan`/`tailnet`/`custom`, ou `auto` lorsque loopback n'est pas disponible) doivent utiliser des jetons/mots de passe d'authentification.
-   `gateway.remote.token` / `.password` sont des sources d'identifiants client. Elles ne configurent **pas** l'authentification du serveur par elles-mêmes.
-   Les chemins d'appel locaux peuvent utiliser `gateway.remote.*` comme secours lorsque `gateway.auth.*` n'est pas défini.
-   `gateway.remote.tlsFingerprint` épingle le certificat TLS distant lors de l'utilisation de `wss://`.
-   **Tailscale Serve** peut authentifier le trafic de l'interface de contrôle/WebSocket via des en-têtes d'identité lorsque `gateway.auth.allowTailscale: true` ; les points de terminaison de l'API HTTP nécessitent toujours une authentification par jeton/mot de passe. Ce flux sans jeton suppose que l'hôte gateway est de confiance. Définissez-le sur `false` si vous voulez des jetons/mots de passe partout.
-   Traitez le contrôle via navigateur comme un accès opérateur : tailnet uniquement + appairage délibéré de nœuds.

Plongée approfondie : [Sécurité](./security.md).

[Découverte Bonjour](./bonjour.md)[Configuration de Gateway à distance](./remote-gateway-readme.md)