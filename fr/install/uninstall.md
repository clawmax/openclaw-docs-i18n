title: "Comment désinstaller OpenClaw AI Gateway et CLI"
description: "Apprenez à désinstaller complètement OpenClaw AI. Suivez les guides étape par étape pour la méthode CLI facile ou la suppression manuelle des services sur macOS, Linux et Windows."
keywords: ["désinstaller openclaw", "supprimer openclaw", "suppression du service gateway", "désinstaller openclaw cli", "effacer openclaw", "nettoyage openclaw", "guide de désinstallation", "suppression de service"]
---

  Maintenance

  
# Désinstaller

Deux chemins :

-   **Chemin facile** si `openclaw` est toujours installé.
-   **Suppression manuelle du service** si le CLI a disparu mais que le service fonctionne toujours.

## Chemin facile (CLI toujours installé)

Recommandé : utiliser le désinstalleur intégré :

```bash
openclaw uninstall
```

Non interactif (automatisation / npx) :

```bash
openclaw uninstall --all --yes --non-interactive
npx -y openclaw uninstall --all --yes --non-interactive
```

Étapes manuelles (même résultat) :

1.  Arrêtez le service gateway :

```bash
openclaw gateway stop
```

2.  Désinstallez le service gateway (launchd/systemd/schtasks) :

```bash
openclaw gateway uninstall
```

3.  Supprimez l'état + la configuration :

```bash
rm -rf "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"
```

Si vous avez défini `OPENCLAW_CONFIG_PATH` vers un emplacement personnalisé en dehors du répertoire d'état, supprimez également ce fichier.

4.  Supprimez votre espace de travail (optionnel, supprime les fichiers des agents) :

```bash
rm -rf ~/.openclaw/workspace
```

5.  Supprimez l'installation du CLI (choisissez celle que vous avez utilisée) :

```bash
npm rm -g openclaw
pnpm remove -g openclaw
bun remove -g openclaw
```

6.  Si vous avez installé l'application macOS :

```bash
rm -rf /Applications/OpenClaw.app
```

Notes :

-   Si vous avez utilisé des profils (`--profile` / `OPENCLAW_PROFILE`), répétez l'étape 3 pour chaque répertoire d'état (les valeurs par défaut sont `~/.openclaw-`).
-   En mode distant, le répertoire d'état se trouve sur **l'hôte de la passerelle**, exécutez donc les étapes 1 à 4 là-bas également.

## Suppression manuelle du service (CLI non installé)

Utilisez cette méthode si le service gateway continue de fonctionner mais que `openclaw` est manquant.

### macOS (launchd)

L'étiquette par défaut est `ai.openclaw.gateway` (ou `ai.openclaw.` ; les anciennes `com.openclaw.*` peuvent encore exister) :

```bash
launchctl bootout gui/$UID/ai.openclaw.gateway
rm -f ~/Library/LaunchAgents/ai.openclaw.gateway.plist
```

Si vous avez utilisé un profil, remplacez l'étiquette et le nom du fichier plist par `ai.openclaw.`. Supprimez tous les anciens fichiers plist `com.openclaw.*` s'ils sont présents.

### Linux (unité utilisateur systemd)

Le nom d'unité par défaut est `openclaw-gateway.service` (ou `openclaw-gateway-.service`) :

```bash
systemctl --user disable --now openclaw-gateway.service
rm -f ~/.config/systemd/user/openclaw-gateway.service
systemctl --user daemon-reload
```

### Windows (Tâche planifiée)

Le nom de tâche par défaut est `OpenClaw Gateway` (ou `OpenClaw Gateway ()`). Le script de la tâche se trouve dans votre répertoire d'état.

```bash
schtasks /Delete /F /TN "OpenClaw Gateway"
Remove-Item -Force "$env:USERPROFILE\.openclaw\gateway.cmd"
```

Si vous avez utilisé un profil, supprimez le nom de tâche correspondant et `~\.openclaw-\gateway.cmd`.

## Installation normale vs dépôt source

### Installation normale (install.sh / npm / pnpm / bun)

Si vous avez utilisé `https://openclaw.ai/install.sh` ou `install.ps1`, le CLI a été installé avec `npm install -g openclaw@latest`. Supprimez-le avec `npm rm -g openclaw` (ou `pnpm remove -g` / `bun remove -g` si vous avez installé de cette façon).

### Dépôt source (git clone)

Si vous exécutez à partir d'un dépôt cloné (`git clone` + `openclaw ...` / `bun run openclaw ...`) :

1.  Désinstallez le service gateway **avant** de supprimer le dépôt (utilisez le chemin facile ci-dessus ou la suppression manuelle du service).
2.  Supprimez le répertoire du dépôt.
3.  Supprimez l'état + l'espace de travail comme indiqué ci-dessus.

[Guide de migration](./migrating.md)[Hébergement VPS](../vps.md)

---