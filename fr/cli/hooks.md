

  Commandes CLI

  
# hooks

Gérez les hooks des agents (automatisations pilotées par événements pour des commandes comme `/new`, `/reset`, et le démarrage de la passerelle). Liens utiles :

-   Hooks : [Hooks](../automation/hooks.md)
-   Hooks de plugins : [Plugins](../tools/plugin.md#plugin-hooks)

## Lister Tous les Hooks

```bash
openclaw hooks list
```

Liste tous les hooks découverts dans les répertoires workspace, managed et bundled. **Options :**

-   `--eligible` : Afficher uniquement les hooks éligibles (prérequis satisfaits)
-   `--json` : Sortie au format JSON
-   `-v, --verbose` : Afficher des informations détaillées incluant les prérequis manquants

**Exemple de sortie :**

```
Hooks (4/4 prêts)

Prêts :
  🚀 boot-md ✓ - Exécute BOOT.md au démarrage de la passerelle
  📎 bootstrap-extra-files ✓ - Injecte des fichiers de bootstrap supplémentaires dans l'espace de travail pendant le bootstrap de l'agent
  📝 command-logger ✓ - Journalise tous les événements de commande dans un fichier d'audit centralisé
  💾 session-memory ✓ - Sauvegarde le contexte de session en mémoire quand la commande /new est exécutée
```

**Exemple (verbose) :**

```bash
openclaw hooks list --verbose
```

Affiche les prérequis manquants pour les hooks non éligibles. **Exemple (JSON) :**

```bash
openclaw hooks list --json
```

Retourne un JSON structuré pour une utilisation programmatique.

## Obtenir les Informations d'un Hook

```bash
openclaw hooks info <nom>
```

Affiche des informations détaillées sur un hook spécifique. **Arguments :**

-   `` : Nom du hook (ex. : `session-memory`)

**Options :**

-   `--json` : Sortie au format JSON

**Exemple :**

```bash
openclaw hooks info session-memory
```

**Sortie :**

```
💾 session-memory ✓ Prêt

Sauvegarde le contexte de session en mémoire quand la commande /new est exécutée

Détails :
  Source : openclaw-bundled
  Chemin : /path/to/openclaw/hooks/bundled/session-memory/HOOK.md
  Gestionnaire : /path/to/openclaw/hooks/bundled/session-memory/handler.ts
  Page d'accueil : https://docs.openclaw.ai/automation/hooks#session-memory
  Événements : command:new

Prérequis :
  Config : ✓ workspace.dir
```

## Vérifier l'Éligibilité des Hooks

```bash
openclaw hooks check
```

Affiche un résumé du statut d'éligibilité des hooks (combien sont prêts vs non prêts). **Options :**

-   `--json` : Sortie au format JSON

**Exemple de sortie :**

```
Statut des Hooks

Total hooks : 4
Prêts : 4
Non prêts : 0
```

## Activer un Hook

```bash
openclaw hooks enable <nom>
```

Active un hook spécifique en l'ajoutant à votre configuration (`~/.openclaw/config.json`). **Note :** Les hooks gérés par des plugins affichent `plugin:` dans `openclaw hooks list` et ne peuvent pas être activés/désactivés ici. Activez/désactivez le plugin à la place. **Arguments :**

-   `` : Nom du hook (ex. : `session-memory`)

**Exemple :**

```bash
openclaw hooks enable session-memory
```

**Sortie :**

```
✓ Hook activé : 💾 session-memory
```

**Ce que cela fait :**

-   Vérifie si le hook existe et est éligible
-   Met à jour `hooks.internal.entries..enabled = true` dans votre configuration
-   Sauvegarde la configuration sur le disque

**Après activation :**

-   Redémarrez la passerelle pour que les hooks se rechargent (redémarrage de l'application de la barre des tâches sur macOS, ou redémarrage de votre processus de passerelle en dev).

## Désactiver un Hook

```bash
openclaw hooks disable <nom>
```

Désactive un hook spécifique en mettant à jour votre configuration. **Arguments :**

-   `` : Nom du hook (ex. : `command-logger`)

**Exemple :**

```bash
openclaw hooks disable command-logger
```

**Sortie :**

```
⏸ Hook désactivé : 📝 command-logger
```

**Après désactivation :**

-   Redémarrez la passerelle pour que les hooks se rechargent

## Installer des Hooks

```bash
openclaw hooks install <chemin-ou-spec>
openclaw hooks install <npm-spec> --pin
```

Installe un pack de hooks depuis un dossier/archive local ou npm. Les spécifications npm sont **uniquement du registre** (nom du package + **version exacte** optionnelle ou **dist-tag**). Les spécifications Git/URL/fichier et les plages semver sont rejetées. Les installations de dépendances s'exécutent avec `--ignore-scripts` pour la sécurité. Les spécifications nues et `@latest` restent sur la piste stable. Si npm résout l'une d'elles en une préversion, OpenClaw s'arrête et vous demande d'accepter explicitement avec un tag de préversion tel que `@beta`/`@rc` ou une version de préversion exacte. **Ce que cela fait :**

-   Copie le pack de hooks dans `~/.openclaw/hooks/`
-   Active les hooks installés dans `hooks.internal.entries.*`
-   Enregistre l'installation sous `hooks.internal.installs`

**Options :**

-   `-l, --link` : Lie un répertoire local au lieu de le copier (l'ajoute à `hooks.internal.load.extraDirs`)
-   `--pin` : Enregistre les installations npm comme `name@version` exacte résolue dans `hooks.internal.installs`

**Archives supportées :** `.zip`, `.tgz`, `.tar.gz`, `.tar` **Exemples :**

```bash
# Répertoire local
openclaw hooks install ./my-hook-pack

# Archive locale
openclaw hooks install ./my-hook-pack.zip

# Package NPM
openclaw hooks install @openclaw/my-hook-pack

# Lier un répertoire local sans copie
openclaw hooks install -l ./my-hook-pack
```

## Mettre à Jour les Hooks

```bash
openclaw hooks update <id>
openclaw hooks update --all
```

Met à jour les packs de hooks installés (installations npm uniquement). **Options :**

-   `--all` : Met à jour tous les packs de hooks suivis
-   `--dry-run` : Affiche ce qui changerait sans écrire

Lorsqu'un hachage d'intégrité stocké existe et que le hachage de l'artefact récupéré change, OpenClaw affiche un avertissement et demande une confirmation avant de continuer. Utilisez le `--yes` global pour contourner les invites dans les exécutions CI/non interactives.

## Hooks Intégrés

### session-memory

Sauvegarde le contexte de session en mémoire quand vous exécutez `/new`. **Activer :**

```bash
openclaw hooks enable session-memory
```

**Sortie :** `~/.openclaw/workspace/memory/YYYY-MM-DD-slug.md` **Voir :** [documentation session-memory](../automation/hooks.md#session-memory)

### bootstrap-extra-files

Injecte des fichiers de bootstrap supplémentaires (par exemple `AGENTS.md` / `TOOLS.md` locaux au monorepo) pendant `agent:bootstrap`. **Activer :**

```bash
openclaw hooks enable bootstrap-extra-files
```

**Voir :** [documentation bootstrap-extra-files](../automation/hooks.md#bootstrap-extra-files)

### command-logger

Journalise tous les événements de commande dans un fichier d'audit centralisé. **Activer :**

```bash
openclaw hooks enable command-logger
```

**Sortie :** `~/.openclaw/logs/commands.log` **Voir les journaux :**

```bash
# Commandes récentes
tail -n 20 ~/.openclaw/logs/commands.log

# Affichage formaté
cat ~/.openclaw/logs/commands.log | jq .

# Filtrer par action
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**Voir :** [documentation command-logger](../automation/hooks.md#command-logger)

### boot-md

Exécute `BOOT.md` au démarrage de la passerelle (après le démarrage des canaux). **Événements :** `gateway:startup` **Activer :**

```bash
openclaw hooks enable boot-md
```

**Voir :** [documentation boot-md](../automation/hooks.md#boot-md)

[health](./health.md)[logs](./logs.md)