

  Application compagnon macOS

  
# Contrôle à distance

Ce flux permet à l'application macOS d'agir comme un contrôle à distance complet pour une passerelle OpenClaw fonctionnant sur un autre hôte (bureau/serveur). C'est la fonctionnalité **Contrôle à distance via SSH** (exécution distante) de l'application. Toutes les fonctionnalités — vérifications de santé, relais de Voice Wake et Web Chat — réutilisent la même configuration SSH distante depuis *Paramètres → Général*.

## Modes

-   **Local (ce Mac)** : Tout s'exécute sur l'ordinateur portable. Aucun SSH impliqué.
-   **Contrôle à distance via SSH (par défaut)** : Les commandes OpenClaw sont exécutées sur l'hôte distant. L'application mac ouvre une connexion SSH avec `-o BatchMode` plus votre identité/clé choisie et un transfert de port local.
-   **Contrôle à distance direct (ws/wss)** : Aucun tunnel SSH. L'application mac se connecte directement à l'URL de la passerelle (par exemple, via Tailscale Serve ou un proxy inverse HTTPS public).

## Transports distants

Le mode distant prend en charge deux transports :

-   **Tunnel SSH** (par défaut) : Utilise `ssh -N -L ...` pour transférer le port de la passerelle vers localhost. La passerelle verra l'IP du nœud comme `127.0.0.1` car le tunnel est en boucle locale.
-   **Direct (ws/wss)** : Se connecte directement à l'URL de la passerelle. La passerelle voit la véritable IP du client.

## Prérequis sur l'hôte distant

1.  Installez Node + pnpm et compilez/installez l'interface CLI OpenClaw (`pnpm install && pnpm build && pnpm link --global`).
2.  Assurez-vous que `openclaw` est dans le PATH pour les shells non interactifs (créez un lien symbolique vers `/usr/local/bin` ou `/opt/homebrew/bin` si nécessaire).
3.  Ouvrez SSH avec authentification par clé. Nous recommandons les IPs **Tailscale** pour une accessibilité stable hors LAN.

## Configuration de l'application macOS

1.  Ouvrez *Paramètres → Général*.
2.  Sous **Exécution d'OpenClaw**, choisissez **Contrôle à distance via SSH** et définissez :
    -   **Transport** : **Tunnel SSH** ou **Direct (ws/wss)**.
    -   **Cible SSH** : `utilisateur@hôte` (`:port` optionnel).
        -   Si la passerelle est sur le même LAN et diffuse en Bonjour, sélectionnez-la dans la liste découverte pour remplir automatiquement ce champ.
    -   **URL de la passerelle** (Direct uniquement) : `wss://gateway.example.ts.net` (ou `ws://...` pour local/LAN).
    -   **Fichier d'identité** (avancé) : chemin vers votre clé.
    -   **Racine du projet** (avancé) : chemin du dépôt distant utilisé pour les commandes.
    -   **Chemin de l'interface CLI** (avancé) : chemin optionnel vers un point d'entrée/binaire `openclaw` exécutable (rempli automatiquement lorsqu'il est diffusé).
3.  Cliquez sur **Tester la connexion distante**. Un succès indique que la commande `openclaw status --json` distante s'exécute correctement. Les échecs signifient généralement des problèmes de PATH/CLI ; le code de sortie 127 signifie que l'interface CLI n'est pas trouvée à distance.
4.  Les vérifications de santé et le Web Chat utiliseront désormais automatiquement ce tunnel SSH.

## Web Chat

-   **Tunnel SSH** : Web Chat se connecte à la passerelle via le port de contrôle WebSocket transféré (par défaut 18789).
-   **Direct (ws/wss)** : Web Chat se connecte directement à l'URL de la passerelle configurée.
-   Il n'y a plus de serveur HTTP WebChat séparé.

## Permissions

-   L'hôte distant a besoin des mêmes approbations TCC que localement (Automatisation, Accessibilité, Enregistrement d'écran, Microphone, Reconnaissance vocale, Notifications). Exécutez l'intégration sur cette machine pour les accorder une fois.
-   Les nœuds diffusent leur état de permission via `node.list` / `node.describe` afin que les agents sachent ce qui est disponible.

## Notes de sécurité

-   Préférez les liaisons en boucle locale sur l'hôte distant et connectez-vous via SSH ou Tailscale.
-   Le tunneling SSH utilise une vérification stricte de la clé hôte ; faites confiance d'abord à la clé hôte pour qu'elle existe dans `~/.ssh/known_hosts`.
-   Si vous liez la passerelle à une interface non locale, exigez une authentification par jeton/mot de passe.
-   Voir [Sécurité](../../gateway/security.md) et [Tailscale](../../gateway/tailscale.md).

## Flux de connexion WhatsApp (distant)

-   Exécutez `openclaw channels login --verbose` **sur l'hôte distant**. Scannez le QR code avec WhatsApp sur votre téléphone.
-   Re-exécutez la connexion sur cet hôte si l'authentification expire. La vérification de santé signalera les problèmes de lien.

## Dépannage

-   **Code de sortie 127 / non trouvé** : `openclaw` n'est pas dans le PATH pour les shells non interactifs. Ajoutez-le à `/etc/paths`, votre fichier rc du shell, ou créez un lien symbolique vers `/usr/local/bin`/`/opt/homebrew/bin`.
-   **Échec de la sonde de santé** : vérifiez l'accessibilité SSH, le PATH, et que Baileys est connecté (`openclaw status --json`).
-   **Web Chat bloqué** : confirmez que la passerelle fonctionne sur l'hôte distant et que le port transféré correspond au port WS de la passerelle ; l'interface utilisateur nécessite une connexion WS saine.
-   **L'IP du nœud affiche 127.0.0.1** : attendu avec le tunnel SSH. Changez le **Transport** pour **Direct (ws/wss)** si vous voulez que la passerelle voie la véritable IP du client.
-   **Voice Wake** : les phrases de déclenchement sont relayées automatiquement en mode distant ; aucun relais séparé n'est nécessaire.

## Sons de notification

Choisissez des sons par notification depuis les scripts avec `openclaw` et `node.invoke`, par exemple :

```bash
openclaw nodes notify --node <id> --title "Ping" --body "Passerelle distante prête" --sound Glass
```

Il n'y a plus de bouton "son par défaut" global dans l'application ; les appelants choisissent un son (ou aucun) par requête.

[Permissions macOS](./permissions.md)[Signature macOS](./signing.md)