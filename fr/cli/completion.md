

  Commandes CLI

  
# completion

Générez des scripts de complétion shell et installez-les éventuellement dans votre profil shell.

## Utilisation

```bash
openclaw completion
openclaw completion --shell zsh
openclaw completion --install
openclaw completion --shell fish --install
openclaw completion --write-state
openclaw completion --shell bash --write-state
```

## Options

-   `-s, --shell `: shell cible (`zsh`, `bash`, `powershell`, `fish` ; par défaut : `zsh`)
-   `-i, --install`: installe la complétion en ajoutant une ligne source à votre profil shell
-   `--write-state`: écrit le(s) script(s) de complétion dans `$OPENCLAW_STATE_DIR/completions` sans les afficher sur stdout
-   `-y, --yes`: ignore les invites de confirmation d'installation

## Notes

-   `--install` écrit un petit bloc "OpenClaw Completion" dans votre profil shell et le pointe vers le script mis en cache.
-   Sans `--install` ou `--write-state`, la commande affiche le script sur stdout.
-   La génération de complétion charge de manière proactive les arborescences de commandes afin que les sous-commandes imbriquées soient incluses.

[clawbot](./clawbot.md)[config](./config.md)

---