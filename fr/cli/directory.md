

  Commandes CLI

  
# directory

Recherches dans le répertoire pour les canaux qui le prennent en charge (contacts/pairs, groupes et "moi").

## Drapeaux courants

-   `--channel ` : identifiant/alias du canal (requis lorsque plusieurs canaux sont configurés ; automatique lorsqu'un seul est configuré)
-   `--account ` : identifiant du compte (par défaut : celui par défaut du canal)
-   `--json` : sortie au format JSON

## Notes

-   `directory` est conçu pour vous aider à trouver des identifiants que vous pouvez coller dans d'autres commandes (en particulier `openclaw message send --target ...`).
-   Pour de nombreux canaux, les résultats sont basés sur la configuration (listes autorisées / groupes configurés) plutôt que sur un répertoire en direct du fournisseur.
-   La sortie par défaut est `id` (et parfois `name`) séparés par une tabulation ; utilisez `--json` pour les scripts.

## Utilisation des résultats avec message send

```bash
openclaw directory peers list --channel slack --query "U0"
openclaw message send --channel slack --target user:U012ABCDEF --message "hello"
```

## Formats d'identifiants (par canal)

-   WhatsApp : `+15551234567` (MP), `1234567890-1234567890@g.us` (groupe)
-   Telegram : `@username` ou identifiant numérique de chat ; les groupes ont des identifiants numériques
-   Slack : `user:U…` et `channel:C…`
-   Discord : `user:` et `channel:`
-   Matrix (plugin) : `user:@user:server`, `room:!roomId:server`, ou `#alias:server`
-   Microsoft Teams (plugin) : `user:` et `conversation:`
-   Zalo (plugin) : identifiant utilisateur (API Bot)
-   Zalo Personnel / `zalouser` (plugin) : identifiant de fil de discussion (MP/groupe) depuis `zca` (`me`, `friend list`, `group list`)

## Soi ("me")

```bash
openclaw directory self --channel zalouser
```

## Pairs (contacts/utilisateurs)

```bash
openclaw directory peers list --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory peers list --channel zalouser --limit 50
```

## Groupes

```bash
openclaw directory groups list --channel zalouser
openclaw directory groups list --channel zalouser --query "work"
openclaw directory groups members --channel zalouser --group-id <id>
```

[devices](./devices.md)[dns](./dns.md)

---