

  Commandes CLI

  
# devices

Gérez les demandes d'appairage de périphériques et les jetons limités aux périphériques.

## Commandes

### openclaw devices list

Lister les demandes d'appairage en attente et les périphériques appairés.

```bash
openclaw devices list
openclaw devices list --json
```

### openclaw devices remove &lt;deviceId&gt;

Supprimer une entrée de périphérique appairé.

```bash
openclaw devices remove <deviceId>
openclaw devices remove <deviceId> --json
```

### openclaw devices clear --yes \[--pending\]

Effacer les périphériques appairés en bloc.

```bash
openclaw devices clear --yes
openclaw devices clear --yes --pending
openclaw devices clear --yes --pending --json
```

### openclaw devices approve \[requestId\] \[--latest\]

Approuver une demande d'appairage de périphérique en attente. Si `requestId` est omis, OpenClaw approuve automatiquement la demande en attente la plus récente.

```bash
openclaw devices approve
openclaw devices approve <requestId>
openclaw devices approve --latest
```

### openclaw devices reject &lt;requestId&gt;

Rejeter une demande d'appairage de périphérique en attente.

```bash
openclaw devices reject <requestId>
```

### openclaw devices rotate --device &lt;id&gt; --role &lt;role&gt; \[--scope &lt;scope...&gt;\]

Faire tourner un jeton de périphérique pour un rôle spécifique (en mettant à jour les portées optionnellement).

```bash
openclaw devices rotate --device <deviceId> --role operator --scope operator.read --scope operator.write
```

### openclaw devices revoke --device &lt;id&gt; --role &lt;role&gt;

Révoquer un jeton de périphérique pour un rôle spécifique.

```bash
openclaw devices revoke --device <deviceId> --role node
```

## Options communes

-   `--url `: URL WebSocket de la passerelle (par défaut `gateway.remote.url` si configuré).
-   `--token `: Jeton de la passerelle (si requis).
-   `--password `: Mot de passe de la passerelle (authentification par mot de passe).
-   `--timeout `: Délai d'expiration RPC.
-   `--json`: Sortie JSON (recommandée pour les scripts).

Note : lorsque vous définissez `--url`, le CLI ne revient pas aux identifiants de configuration ou d'environnement. Passez `--token` ou `--password` explicitement. L'absence d'identifiants explicites est une erreur.

## Notes

-   La rotation de jeton retourne un nouveau jeton (sensible). Traitez-le comme un secret.
-   Ces commandes nécessitent la portée `operator.pairing` (ou `operator.admin`).
-   `devices clear` est intentionnellement protégé par `--yes`.
-   Si la portée d'appairage n'est pas disponible en boucle locale (et qu'aucun `--url` explicite n'est passé), list/approve peut utiliser un secours d'appairage local.

[dashboard](./dashboard.md)[directory](./directory.md)

---