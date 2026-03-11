

  Commandes CLI

  
# qr

Générez un QR d'appairage iOS et un code de configuration à partir de la configuration actuelle de votre passerelle.

## Utilisation

```bash
openclaw qr
openclaw qr --setup-code-only
openclaw qr --json
openclaw qr --remote
openclaw qr --url wss://gateway.example/ws --token '<token>'
```

## Options

-   `--remote` : utilise `gateway.remote.url` plus le jeton/mot de passe distant de la configuration
-   `--url ` : remplace l'URL de la passerelle utilisée dans la charge utile
-   `--public-url ` : remplace l'URL publique utilisée dans la charge utile
-   `--token ` : remplace le jeton de la passerelle pour la charge utile
-   `--password ` : remplace le mot de passe de la passerelle pour la charge utile
-   `--setup-code-only` : affiche uniquement le code de configuration
-   `--no-ascii` : ignore le rendu ASCII du QR
-   `--json` : émet du JSON (`setupCode`, `gatewayUrl`, `auth`, `urlSource`)

## Notes

-   `--token` et `--password` s'excluent mutuellement.
-   Avec `--remote`, si des identifiants distants effectivement actifs sont configurés en tant que SecretRefs et que vous ne passez pas `--token` ou `--password`, la commande les résout à partir de l'instantané de la passerelle active. Si la passerelle est indisponible, la commande échoue rapidement.
-   Sans `--remote`, les SecretRefs d'authentification locale de la passerelle sont résolus lorsqu'aucune substitution d'authentification CLI n'est passée :
    -   `gateway.auth.token` se résout lorsque l'authentification par jeton peut l'emporter (`gateway.auth.mode="token"` explicite ou mode inféré où aucune source de mot de passe ne l'emporte).
    -   `gateway.auth.password` se résout lorsque l'authentification par mot de passe peut l'emporter (`gateway.auth.mode="password"` explicite ou mode inféré sans jeton gagnant provenant de l'authentification/de l'environnement).
-   Si `gateway.auth.token` et `gateway.auth.password` sont tous deux configurés (y compris les SecretRefs) et que `gateway.auth.mode` n'est pas défini, la résolution du code de configuration échoue jusqu'à ce que le mode soit défini explicitement.
-   Note sur le décalage de version de la passerelle : ce chemin de commande nécessite une passerelle qui prend en charge `secrets.resolve` ; les passerelles plus anciennes renvoient une erreur de méthode inconnue.
-   Après le scan, approuvez l'appairage de l'appareil avec :
    -   `openclaw devices list`
    -   `openclaw devices approve `

[plugins](./plugins.md)[reset](./reset.md)