

  Commandes CLI

  
# channels

Gérez les comptes de canaux de discussion et leur état d'exécution sur la Passerelle. Documentation associée :

-   Guides des canaux : [Canaux](../channels/index.md)
-   Configuration de la passerelle : [Configuration](../gateway/configuration.md)

## Commandes courantes

```bash
openclaw channels list
openclaw channels status
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels logs --channel all
```

## Ajouter / supprimer des comptes

```bash
openclaw channels add --channel telegram --token <bot-token>
openclaw channels remove --channel telegram --delete
```

Astuce : `openclaw channels add --help` affiche les drapeaux spécifiques par canal (token, app token, chemins signal-cli, etc.). Lorsque vous exécutez `openclaw channels add` sans drapeaux, l'assistant interactif peut demander :

-   les identifiants de compte par canal sélectionné
-   les noms d'affichage optionnels pour ces comptes
-   `Lier les comptes de canaux configurés aux agents maintenant ?`

Si vous confirmez la liaison maintenant, l'assistant demande quel agent doit posséder chaque compte de canal configuré et écrit les règles de routage limitées au compte. Vous pouvez également gérer ces mêmes règles de routage plus tard avec `openclaw agents bindings`, `openclaw agents bind`, et `openclaw agents unbind` (voir [agents](./agents.md)). Lorsque vous ajoutez un compte non par défaut à un canal qui utilise encore les paramètres de niveau supérieur pour un seul compte (pas encore d'entrées `channels..accounts`), OpenClaw déplace les valeurs de niveau supérieur limitées au compte dans `channels..accounts.default`, puis écrit le nouveau compte. Cela préserve le comportement du compte original tout en passant à la structure multi-comptes. Le comportement du routage reste cohérent :

-   Les liaisons existantes limitées au canal (sans `accountId`) continuent de correspondre au compte par défaut.
-   `channels add` ne crée pas automatiquement ni ne réécrit les liaisons en mode non interactif.
-   La configuration interactive peut optionnellement ajouter des liaisons limitées au compte.

Si votre configuration était déjà dans un état mixte (comptes nommés présents, `default` manquant, et les valeurs de niveau supérieur pour un seul compte encore définies), exécutez `openclaw doctor --fix` pour déplacer les valeurs limitées au compte dans `accounts.default`.

## Connexion / déconnexion (interactif)

```bash
openclaw channels login --channel whatsapp
openclaw channels logout --channel whatsapp
```

## Dépannage

-   Exécutez `openclaw status --deep` pour une analyse large.
-   Utilisez `openclaw doctor` pour des corrections guidées.
-   `openclaw channels list` affiche `Claude: HTTP 403 ... user:profile` → l'instantané d'utilisation nécessite la portée `user:profile`. Utilisez `--no-usage`, ou fournissez une clé de session claude.ai (`CLAUDE_WEB_SESSION_KEY` / `CLAUDE_WEB_COOKIE`), ou ré-authentifiez-vous via Claude Code CLI.
-   `openclaw channels status` revient à des résumés basés uniquement sur la configuration lorsque la passerelle est injoignable. Si un identifiant de canal pris en charge est configuré via SecretRef mais indisponible dans le chemin de commande actuel, il rapporte ce compte comme configuré avec des notes de dégradation au lieu de l'afficher comme non configuré.

## Sondage des capacités

Récupérez les indications de capacité du fournisseur (intentions/portées là où disponibles) plus la prise en charge statique des fonctionnalités :

```bash
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
```

Notes :

-   `--channel` est optionnel ; omettez-le pour lister chaque canal (y compris les extensions).
-   `--target` accepte `channel:` ou un identifiant de canal numérique brut et s'applique uniquement à Discord.
-   Les sondages sont spécifiques au fournisseur : intentions Discord + permissions de canal optionnelles ; portées bot + utilisateur Slack ; drapeaux bot Telegram + webhook ; version du démon Signal ; token d'application MS Teams + rôles/portées Graph (annotés là où connus). Les canaux sans sondage rapportent `Probe: unavailable`.

## Résoudre les noms en ID

Résolvez les noms de canaux/utilisateurs en ID en utilisant l'annuaire du fournisseur :

```bash
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels resolve --channel discord "My Server/#support" "@someone"
openclaw channels resolve --channel matrix "Project Room"
```

Notes :

-   Utilisez `--kind user|group|auto` pour forcer le type de cible.
-   La résolution préfère les correspondances actives lorsque plusieurs entrées partagent le même nom.
-   `channels resolve` est en lecture seule. Si un compte sélectionné est configuré via SecretRef mais que cet identifiant est indisponible dans le chemin de commande actuel, la commande retourne des résultats non résolus dégradés avec des notes au lieu d'interrompre l'exécution complète.

[browser](./browser.md)[clawbot](./clawbot.md)

---