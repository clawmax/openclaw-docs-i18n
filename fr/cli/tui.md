

  Commandes CLI

  
# tui

Ouvre l'interface utilisateur terminal connectée à la Gateway. Liens utiles :

-   Guide TUI : [TUI](../web/tui.md)

Notes :

-   `tui` résout les SecretRefs d'authentification configurés pour la gateway pour l'authentification par token/mot de passe lorsque cela est possible (fournisseurs `env`/`file`/`exec`).

## Exemples

```bash
openclaw tui
openclaw tui --url ws://127.0.0.1:18789 --token <token>
openclaw tui --session main --deliver
```

[system](./system.md)[uninstall](./uninstall.md)

---