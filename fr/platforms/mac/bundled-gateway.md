

  Application compagnon macOS

  
# Passerelle sur macOS

OpenClaw.app n'intègre plus Node/Bun ni l'environnement d'exécution de la passerelle. L'application macOS s'attend à une installation **externe** de l'interface CLI `openclaw`, ne lance pas la passerelle en tant que processus enfant, et gère un service launchd par utilisateur pour maintenir la passerelle en fonctionnement (ou se connecte à une passerelle locale existante si une est déjà en cours d'exécution).

## Installer l'interface CLI (requis pour le mode local)

Vous avez besoin de Node 22+ sur le Mac, puis installez `openclaw` globalement :

```bash
npm install -g openclaw@<version>
```

Le bouton **Installer CLI** de l'application macOS exécute le même flux via npm/pnpm (bun n'est pas recommandé pour l'environnement d'exécution de la passerelle).

## Launchd (Passerelle en tant que LaunchAgent)

Étiquette :

-   `ai.openclaw.gateway` (ou `ai.openclaw.` ; l'ancienne `com.openclaw.*` peut subsister)

Emplacement du fichier Plist (par utilisateur) :

-   `~/Library/LaunchAgents/ai.openclaw.gateway.plist` (ou `~/Library/LaunchAgents/ai.openclaw..plist`)

Gestionnaire :

-   L'application macOS gère l'installation/la mise à jour du LaunchAgent en mode Local.
-   L'interface CLI peut également l'installer : `openclaw gateway install`.

Comportement :

-   « OpenClaw Active » active/désactive le LaunchAgent.
-   La fermeture de l'application **n'arrête pas** la passerelle (launchd la maintient en vie).
-   Si une passerelle est déjà en cours d'exécution sur le port configuré, l'application s'y attache au lieu d'en démarrer une nouvelle.

Journalisation :

-   sortie standard/erreur de launchd : `/tmp/openclaw/openclaw-gateway.log`

## Compatibilité des versions

L'application macOS vérifie la version de la passerelle par rapport à sa propre version. Si elles sont incompatibles, mettez à jour l'interface CLI globale pour qu'elle corresponde à la version de l'application.

## Vérification rapide

```bash
openclaw --version

OPENCLAW_SKIP_CHANNELS=1 \
OPENCLAW_SKIP_CANVAS_HOST=1 \
openclaw gateway --port 18999 --bind loopback
```

Puis :

```bash
openclaw gateway call health --url ws://127.0.0.1:18999 --timeout 3000
```

[Publication macOS](./release.md)[IPC macOS](./xpc.md)