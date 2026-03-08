

  Commandes CLI

  
# plugins

Gérez les plugins/extensions de la Passerelle (chargés en processus). Liens connexes :

-   Système de plugins : [Plugins](../tools/plugin.md)
-   Manifeste + schéma de plugin : [Manifeste de plugin](../plugins/manifest.md)
-   Renforcement de la sécurité : [Sécurité](../gateway/security.md)

## Commandes

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins uninstall <id>
openclaw plugins doctor
openclaw plugins update <id>
openclaw plugins update --all
```

Les plugins intégrés sont fournis avec OpenClaw mais démarrent désactivés. Utilisez `plugins enable` pour les activer. Tous les plugins doivent fournir un fichier `openclaw.plugin.json` avec un schéma JSON intégré (`configSchema`, même s'il est vide). Les manifestes ou schémas manquants/invalides empêchent le chargement du plugin et font échouer la validation de la configuration.

### Installer

```bash
openclaw plugins install <path-or-spec>
openclaw plugins install <npm-spec> --pin
```

Note de sécurité : traitez l'installation de plugins comme l'exécution de code. Préférez les versions épinglées. Les spécifications npm sont **uniquement pour le registre** (nom du package + **version exacte** ou **dist-tag** optionnels). Les spécifications Git/URL/fichier et les plages semver sont rejetées. Les installations de dépendances s'exécutent avec `--ignore-scripts` pour la sécurité. Les spécifications simples et `@latest` restent sur la voie stable. Si npm résout l'une d'entre elles en une préversion, OpenClaw s'arrête et vous demande d'accepter explicitement avec un tag de préversion tel que `@beta`/`@rc` ou une version de préversion exacte telle que `@1.2.3-beta.4`. Si une spécification d'installation simple correspond à un identifiant de plugin intégré (par exemple `diffs`), OpenClaw installe directement le plugin intégré. Pour installer un package npm du même nom, utilisez une spécification explicite avec scope (par exemple `@scope/diffs`). Archives prises en charge : `.zip`, `.tgz`, `.tar.gz`, `.tar`. Utilisez `--link` pour éviter de copier un répertoire local (ajoute à `plugins.load.paths`) :

```bash
openclaw plugins install -l ./my-plugin
```

Utilisez `--pin` sur les installations npm pour enregistrer la spécification exacte résolue (`name@version`) dans `plugins.installs` tout en conservant le comportement par défaut non épinglé.

### Désinstaller

```bash
openclaw plugins uninstall <id>
openclaw plugins uninstall <id> --dry-run
openclaw plugins uninstall <id> --keep-files
```

`uninstall` supprime les enregistrements de plugin de `plugins.entries`, `plugins.installs`, la liste autorisée des plugins, et les entrées liées dans `plugins.load.paths` le cas échéant. Pour les plugins mémoire actifs, l'emplacement mémoire est réinitialisé à `memory-core`. Par défaut, la désinstallation supprime également le répertoire d'installation du plugin sous la racine des extensions du répertoire d'état actif (`$OPENCLAW_STATE_DIR/extensions/`). Utilisez `--keep-files` pour conserver les fichiers sur le disque. `--keep-config` est pris en charge comme alias obsolète pour `--keep-files`.

### Mettre à jour

```bash
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins update <id> --dry-run
```

Les mises à jour s'appliquent uniquement aux plugins installés depuis npm (suivis dans `plugins.installs`). Lorsqu'un hachage d'intégrité stocké existe et que le hachage de l'artefact récupéré change, OpenClaw affiche un avertissement et demande une confirmation avant de continuer. Utilisez l'option globale `--yes` pour contourner les invites dans les exécutions CI/non interactives.

[pairing](./pairing.md)[qr](./qr.md)