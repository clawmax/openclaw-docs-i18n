

  Autres méthodes d'installation

  
# Nix

La méthode recommandée pour exécuter OpenClaw avec Nix est via **[nix-openclaw](https://github.com/openclaw/nix-openclaw)** — un module Home Manager tout-en-un.

## Démarrage rapide

Collez ceci à votre agent IA (Claude, Cursor, etc.) :

```
I want to set up nix-openclaw on my Mac.
Repository: github:openclaw/nix-openclaw

What I need you to do:
1. Check if Determinate Nix is installed (if not, install it)
2. Create a local flake at ~/code/openclaw-local using templates/agent-first/flake.nix
3. Help me create a Telegram bot (@BotFather) and get my chat ID (@userinfobot)
4. Set up secrets (bot token, model provider API key) - plain files at ~/.secrets/ is fine
5. Fill in the template placeholders and run home-manager switch
6. Verify: launchd running, bot responds to messages

Reference the nix-openclaw README for module options.
```

> **📦 Guide complet : [github.com/openclaw/nix-openclaw](https://github.com/openclaw/nix-openclaw)** Le dépôt nix-openclaw est la source de référence pour l'installation Nix. Cette page n'est qu'un aperçu rapide.

## Ce que vous obtenez

-   Gateway + application macOS + outils (whisper, spotify, cameras) — tous épinglés
-   Service launchd qui survit aux redémarrages
-   Système de plugins avec configuration déclarative
-   Retour arrière instantané : `home-manager switch --rollback`

* * *

## Comportement d'exécution du mode Nix

Lorsque `OPENCLAW_NIX_MODE=1` est défini (automatique avec nix-openclaw) : OpenClaw prend en charge un **mode Nix** qui rend la configuration déterministe et désactive les flux d'auto-installation. Activez-le en exportant :

```
OPENCLAW_NIX_MODE=1
```

Sur macOS, l'application GUI n'hérite pas automatiquement des variables d'environnement du shell. Vous pouvez également activer le mode Nix via defaults :

```bash
defaults write ai.openclaw.mac openclaw.nixMode -bool true
```

### Chemins de configuration + état

OpenClaw lit la configuration JSON5 depuis `OPENCLAW_CONFIG_PATH` et stocke les données modifiables dans `OPENCLAW_STATE_DIR`. Si nécessaire, vous pouvez également définir `OPENCLAW_HOME` pour contrôler le répertoire de base utilisé pour la résolution des chemins internes.

-   `OPENCLAW_HOME` (priorité par défaut : `HOME` / `USERPROFILE` / `os.homedir()`)
-   `OPENCLAW_STATE_DIR` (par défaut : `~/.openclaw`)
-   `OPENCLAW_CONFIG_PATH` (par défaut : `$OPENCLAW_STATE_DIR/openclaw.json`)

Lors de l'exécution sous Nix, définissez ces chemins explicitement vers des emplacements gérés par Nix afin que l'état d'exécution et la configuration restent en dehors du store immuable.

### Comportement d'exécution en mode Nix

-   Les flux d'auto-installation et d'auto-mutation sont désactivés
-   Les dépendances manquantes affichent des messages de correction spécifiques à Nix
-   L'interface affiche une bannière en mode Nix en lecture seule lorsqu'elle est présente

## Note d'empaquetage (macOS)

Le flux d'empaquetage macOS attend un modèle Info.plist stable à l'emplacement :

```
apps/macos/Sources/OpenClaw/Resources/Info.plist
```

[`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) copie ce modèle dans le bundle de l'application et corrige les champs dynamiques (ID de bundle, version/build, SHA Git, clés Sparkle). Cela maintient le fichier plist déterministe pour l'empaquetage SwiftPM et les builds Nix (qui ne dépendent pas d'une chaîne d'outils Xcode complète).

## Liens connexes

-   [nix-openclaw](https://github.com/openclaw/nix-openclaw) — guide de configuration complet
-   [Assistant](../start/wizard.md) — configuration CLI non-Nix
-   [Docker](./docker.md) — configuration conteneurisée

[Podman](./podman.md)[Ansible](./ansible.md)

---