

  Commandes CLI

  
# browser

Gérez le serveur de contrôle du navigateur d'OpenClaw et exécutez des actions (onglets, instantanés, captures d'écran, navigation, clics, saisie). Liens utiles :

-   Outil navigateur + API : [Outil navigateur](../tools/browser.md)
-   Relais d'extension Chrome : [Extension Chrome](../tools/chrome-extension.md)

## Options courantes

-   `--url ` : URL WebSocket de la passerelle (par défaut depuis la configuration).
-   `--token ` : Jeton de la passerelle (si requis).
-   `--timeout ` : délai d'expiration de la requête (ms).
-   `--browser-profile ` : choisir un profil de navigateur (par défaut depuis la configuration).
-   `--json` : sortie au format machine (là où c'est supporté).

## Démarrage rapide (local)

```bash
openclaw browser --browser-profile chrome tabs
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

## Profils

Les profils sont des configurations nommées de routage du navigateur. En pratique :

-   `openclaw` : lance/se connecte à une instance Chrome dédiée gérée par OpenClaw (répertoire de données utilisateur isolé).
-   `chrome` : contrôle vos onglets Chrome existants via le relais de l'extension Chrome.

```bash
openclaw browser profiles
openclaw browser create-profile --name work --color "#FF5A36"
openclaw browser delete-profile --name work
```

Utiliser un profil spécifique :

```bash
openclaw browser --browser-profile work tabs
```

## Onglets

```bash
openclaw browser tabs
openclaw browser open https://docs.openclaw.ai
openclaw browser focus <targetId>
openclaw browser close <targetId>
```

## Instantané / capture d'écran / actions

Instantané :

```bash
openclaw browser snapshot
```

Capture d'écran :

```bash
openclaw browser screenshot
```

Naviguer/cliquer/taper (automatisation d'interface par référence) :

```bash
openclaw browser navigate https://example.com
openclaw browser click <ref>
openclaw browser type <ref> "hello"
```

## Relais d'extension Chrome (attachement via le bouton de la barre d'outils)

Ce mode permet à l'agent de contrôler un onglet Chrome existant que vous attachez manuellement (il ne s'attache pas automatiquement). Installez l'extension non empaquetée dans un chemin stable :

```bash
openclaw browser extension install
openclaw browser extension path
```

Ensuite, dans Chrome → `chrome://extensions` → activez le "Mode développeur" → "Charger l'extension non empaquetée" → sélectionnez le dossier indiqué. Guide complet : [Extension Chrome](../tools/chrome-extension.md)

## Contrôle à distance du navigateur (proxy hôte nœud)

Si la passerelle s'exécute sur une machine différente de celle du navigateur, exécutez un **hôte nœud** sur la machine qui possède Chrome/Brave/Edge/Chromium. La passerelle redirigera les actions du navigateur vers ce nœud (aucun serveur de contrôle de navigateur séparé requis). Utilisez `gateway.nodes.browser.mode` pour contrôler le routage automatique et `gateway.nodes.browser.node` pour épingler un nœud spécifique si plusieurs sont connectés. Sécurité et configuration à distance : [Outil navigateur](../tools/browser.md), [Accès distant](../gateway/remote.md), [Tailscale](../gateway/tailscale.md), [Sécurité](../gateway/security.md)

[approvals](./approvals.md)[channels](./channels.md)

---