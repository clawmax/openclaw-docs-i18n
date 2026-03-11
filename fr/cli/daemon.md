

  Commandes CLI

  
# daemon

Alias legacy pour les commandes de gestion du service Gateway. `openclaw daemon ...` correspond à la même interface de contrôle de service que les commandes de service `openclaw gateway ...`.

## Utilisation

```bash
openclaw daemon status
openclaw daemon install
openclaw daemon start
openclaw daemon stop
openclaw daemon restart
openclaw daemon uninstall
```

## Sous-commandes

-   `status` : afficher l'état d'installation du service et sonder la santé du Gateway
-   `install` : installer le service (`launchd`/`systemd`/`schtasks`)
-   `uninstall` : désinstaller le service
-   `start` : démarrer le service
-   `stop` : arrêter le service
-   `restart` : redémarrer le service

## Options courantes

-   `status` : `--url`, `--token`, `--password`, `--timeout`, `--no-probe`, `--deep`, `--json`
-   `install` : `--port`, `--runtime <node|bun>`, `--token`, `--force`, `--json`
-   cycle de vie (`uninstall|start|stop|restart`) : `--json`

Notes :

-   `status` résout les SecretRefs d'authentification configurés pour l'authentification de la sonde lorsque c'est possible.
-   Lorsque l'authentification par token nécessite un jeton et que `gateway.auth.token` est géré par SecretRef, `install` valide que le SecretRef peut être résolu mais ne persiste pas le jeton résolu dans les métadonnées d'environnement du service.
-   Si l'authentification par token nécessite un jeton et que le SecretRef de jeton configuré n'est pas résolu, l'installation échoue en mode fermé.
-   Si `gateway.auth.token` et `gateway.auth.password` sont tous deux configurés et que `gateway.auth.mode` n'est pas défini, l'installation est bloquée jusqu'à ce que le mode soit défini explicitement.

## Préférez

Utilisez [`openclaw gateway`](./gateway.md) pour la documentation et les exemples actuels.

[cron](./cron.md)[dashboard](./dashboard.md)

---