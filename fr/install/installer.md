

  Aperçu de l'installation

  
# Fonctionnement interne de l'installateur

OpenClaw fournit trois scripts d'installation, servis depuis `openclaw.ai`.

| Script | Plateforme | Fonction |
| --- | --- | --- |
| [`install.sh`](#installsh) | macOS / Linux / WSL | Installe Node si nécessaire, installe OpenClaw via npm (par défaut) ou git, et peut exécuter l'intégration. |
| [`install-cli.sh`](#install-clish) | macOS / Linux / WSL | Installe Node + OpenClaw dans un préfixe local (`~/.openclaw`). Aucun privilège root requis. |
| [`install.ps1`](#installps1) | Windows (PowerShell) | Installe Node si nécessaire, installe OpenClaw via npm (par défaut) ou git, et peut exécuter l'intégration. |

## Commandes rapides

> **ℹ️** Si l'installation réussit mais que `openclaw` n'est pas trouvé dans un nouveau terminal, consultez le [dépannage Node.js](./node.md#troubleshooting).

* * *

## install.sh

> **💡** Recommandé pour la plupart des installations interactives sur macOS/Linux/WSL.

### Déroulement (install.sh)

### Étape 1 : Détection de l'OS

Prend en charge macOS et Linux (y compris WSL). Si macOS est détecté, installe Homebrew s'il est manquant.

### Étape 2 : Vérification de Node.js 22+

Vérifie la version de Node et installe Node 22 si nécessaire (Homebrew sur macOS, scripts de configuration NodeSource sur apt/dnf/yum Linux).

### Étape 3 : Vérification de Git

Installe Git s'il est manquant.

### Étape 4 : Installation d'OpenClaw

-   Méthode `npm` (par défaut) : installation globale via npm
-   Méthode `git` : clone/mise à jour du dépôt, installation des dépendances avec pnpm, construction, puis installation du wrapper à `~/.local/bin/openclaw`

### Étape 5 : Tâches post-installation

-   Exécute `openclaw doctor --non-interactive` lors des mises à niveau et des installations git (meilleur effort)
-   Tente l'intégration lorsque c'est approprié (TTY disponible, intégration non désactivée, et les vérifications bootstrap/config passent)
-   Définit par défaut `SHARP_IGNORE_GLOBAL_LIBVIPS=1`

### Détection d'un checkout source

Si exécuté dans un checkout OpenClaw (`package.json` + `pnpm-workspace.yaml`), le script propose :

-   utiliser le checkout (`git`), ou
-   utiliser l'installation globale (`npm`)

Si aucun TTY n'est disponible et qu'aucune méthode d'installation n'est définie, il utilise par défaut `npm` et avertit. Le script se termine avec le code `2` pour une sélection de méthode invalide ou des valeurs `--install-method` invalides.

### Exemples (install.sh)

| Drapeau | Description |
| --- | --- |
| `--install-method npm\|git` | Choisir la méthode d'installation (par défaut : `npm`). Alias : `--method` |
| `--npm` | Raccourci pour la méthode npm |
| `--git` | Raccourci pour la méthode git. Alias : `--github` |
| `--version <version\|dist-tag>` | Version npm ou dist-tag (par défaut : `latest`) |
| `--beta` | Utiliser le dist-tag beta s'il est disponible, sinon revenir à `latest` |
| `--git-dir ` | Répertoire du checkout (par défaut : `~/openclaw`). Alias : `--dir` |
| `--no-git-update` | Ignorer `git pull` pour un checkout existant |
| `--no-prompt` | Désactiver les invites |
| `--no-onboard` | Ignorer l'intégration |
| `--onboard` | Activer l'intégration |
| `--dry-run` | Afficher les actions sans les appliquer |
| `--verbose` | Activer la sortie de débogage (`set -x`, logs npm niveau notice) |
| `--help` | Afficher l'utilisation (`-h`) |

| Variable | Description |
| --- | --- |
| `OPENCLAW_INSTALL_METHOD=git\|npm` | Méthode d'installation |
| `OPENCLAW_VERSION=latest\|next\|` | Version npm ou dist-tag |
| `OPENCLAW_BETA=0\|1` | Utiliser beta si disponible |
| `OPENCLAW_GIT_DIR=` | Répertoire du checkout |
| `OPENCLAW_GIT_UPDATE=0\|1` | Activer/désactiver les mises à jour git |
| `OPENCLAW_NO_PROMPT=1` | Désactiver les invites |
| `OPENCLAW_NO_ONBOARD=1` | Ignorer l'intégration |
| `OPENCLAW_DRY_RUN=1` | Mode simulation |
| `OPENCLAW_VERBOSE=1` | Mode débogage |
| `OPENCLAW_NPM_LOGLEVEL=error\|warn\|notice` | Niveau de log npm |
| `SHARP_IGNORE_GLOBAL_LIBVIPS=0\|1` | Contrôler le comportement sharp/libvips (par défaut : `1`) |

* * *

## install-cli.sh

> **ℹ️** Conçu pour les environnements où vous voulez tout sous un préfixe local (par défaut `~/.openclaw`) et sans dépendance système Node.

### Déroulement (install-cli.sh)

### Étape 1 : Installation du runtime Node local

Télécharge l'archive Node (par défaut `22.22.0`) vers `/tools/node-v` et vérifie le SHA-256.

### Étape 2 : Vérification de Git

Si Git est manquant, tente une installation via apt/dnf/yum sur Linux ou Homebrew sur macOS.

### Étape 3 : Installation d'OpenClaw sous le préfixe

Installe avec npm en utilisant `--prefix `, puis écrit le wrapper dans `/bin/openclaw`.

### Exemples (install-cli.sh)

| Drapeau | Description |
| --- | --- |
| `--prefix ` | Préfixe d'installation (par défaut : `~/.openclaw`) |
| `--version ` | Version ou dist-tag d'OpenClaw (par défaut : `latest`) |
| `--node-version ` | Version de Node (par défaut : `22.22.0`) |
| `--json` | Émettre des événements NDJSON |
| `--onboard` | Exécuter `openclaw onboard` après l'installation |
| `--no-onboard` | Ignorer l'intégration (par défaut) |
| `--set-npm-prefix` | Sur Linux, forcer le préfixe npm à `~/.npm-global` si le préfixe actuel n'est pas accessible en écriture |
| `--help` | Afficher l'utilisation (`-h`) |

| Variable | Description |
| --- | --- |
| `OPENCLAW_PREFIX=` | Préfixe d'installation |
| `OPENCLAW_VERSION=` | Version ou dist-tag d'OpenClaw |
| `OPENCLAW_NODE_VERSION=` | Version de Node |
| `OPENCLAW_NO_ONBOARD=1` | Ignorer l'intégration |
| `OPENCLAW_NPM_LOGLEVEL=error\|warn\|notice` | Niveau de log npm |
| `OPENCLAW_GIT_DIR=` | Chemin de recherche pour le nettoyage hérité (utilisé lors de la suppression de l'ancien checkout de sous-module `Peekaboo`) |
| `SHARP_IGNORE_GLOBAL_LIBVIPS=0\|1` | Contrôler le comportement sharp/libvips (par défaut : `1`) |

* * *

## install.ps1

### Déroulement (install.ps1)

### Étape 1 : Vérification de PowerShell + environnement Windows

Nécessite PowerShell 5+.

### Étape 2 : Vérification de Node.js 22+

S'il est manquant, tente une installation via winget, puis Chocolatey, puis Scoop.

### Étape 3 : Installation d'OpenClaw

-   Méthode `npm` (par défaut) : installation globale via npm en utilisant le `-Tag` sélectionné
-   Méthode `git` : clone/mise à jour du dépôt, installation/construction avec pnpm, et installation du wrapper à `%USERPROFILE%\.local\bin\openclaw.cmd`

### Étape 4 : Tâches post-installation

Ajoute le répertoire bin nécessaire au PATH utilisateur lorsque possible, puis exécute `openclaw doctor --non-interactive` lors des mises à niveau et des installations git (meilleur effort).

### Exemples (install.ps1)

| Drapeau | Description |
| --- | --- |
| `-InstallMethod npm\|git` | Méthode d'installation (par défaut : `npm`) |
| `-Tag ` | Dist-tag npm (par défaut : `latest`) |
| `-GitDir ` | Répertoire du checkout (par défaut : `%USERPROFILE%\openclaw`) |
| `-NoOnboard` | Ignorer l'intégration |
| `-NoGitUpdate` | Ignorer `git pull` |
| `-DryRun` | Afficher uniquement les actions |

| Variable | Description |
| --- | --- |
| `OPENCLAW_INSTALL_METHOD=git\|npm` | Méthode d'installation |
| `OPENCLAW_GIT_DIR=` | Répertoire du checkout |
| `OPENCLAW_NO_ONBOARD=1` | Ignorer l'intégration |
| `OPENCLAW_GIT_UPDATE=0` | Désactiver git pull |
| `OPENCLAW_DRY_RUN=1` | Mode simulation |

> **ℹ️** Si `-InstallMethod git` est utilisé et que Git est manquant, le script se termine et affiche le lien vers Git pour Windows.

* * *

## CI et automatisation

Utilisez des drapeaux/variables d'environnement non interactifs pour des exécutions prévisibles.

* * *

## Dépannage

Git est requis pour la méthode d'installation `git`. Pour les installations `npm`, Git est toujours vérifié/installé pour éviter les échecs `spawn git ENOENT` lorsque les dépendances utilisent des URL git.

Certaines configurations Linux pointent le préfixe global npm vers des chemins appartenant à root. `install.sh` peut basculer le préfixe vers `~/.npm-global` et ajouter des exportations PATH aux fichiers rc du shell (lorsque ces fichiers existent).

Les scripts définissent par défaut `SHARP_IGNORE_GLOBAL_LIBVIPS=1` pour éviter que sharp ne se construise contre libvips système. Pour outrepasser :

```
SHARP_IGNORE_GLOBAL_LIBVIPS=0 curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash
```

Installez Git pour Windows, redémarrez PowerShell, réexécutez l'installateur.

Exécutez `npm config get prefix` et ajoutez ce répertoire à votre PATH utilisateur (pas besoin du suffixe `\bin` sur Windows), puis redémarrez PowerShell.

`install.ps1` n'expose pas actuellement de commutateur `-Verbose`. Utilisez le traçage PowerShell pour les diagnostics au niveau du script :

```powershell
Set-PSDebug -Trace 1
& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -NoOnboard
Set-PSDebug -Trace 0
```

Généralement un problème de PATH. Voir le [dépannage Node.js](./node.md#troubleshooting).

[Installation](../install.md)[Docker](./docker.md)