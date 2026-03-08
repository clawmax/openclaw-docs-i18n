title: "Guide de configuration et de sécurité des canaux IRC OpenClaw"
description: "Apprenez à configurer OpenClaw pour IRC, définir des politiques de sécurité, gérer le contrôle d'accès et résoudre les problèmes courants pour Libera.Chat et autres réseaux."
keywords: ["openclaw irc", "configuration bot irc", "sécurité canal irc", "contrôle d'accès irc", "gestion des mentions irc", "configuration nickserv", "dépannage irc", "politique de groupe irc"]
---

  Plateformes de messagerie

  
# IRC

Utilisez IRC lorsque vous souhaitez qu'OpenClaw soit présent dans des canaux classiques (`#room`) et des messages directs. IRC est fourni en tant qu'extension, mais il est configuré dans la configuration principale sous `channels.irc`.

## Démarrage rapide

1.  Activez la configuration IRC dans `~/.openclaw/openclaw.json`.
2.  Définissez au minimum :

```json
{
  "channels": {
    "irc": {
      "enabled": true,
      "host": "irc.libera.chat",
      "port": 6697,
      "tls": true,
      "nick": "openclaw-bot",
      "channels": ["#openclaw"]
    }
  }
}
```

3.  Démarrez/redémarrez la passerelle :

```bash
openclaw gateway run
```

## Valeurs par défaut de sécurité

-   `channels.irc.dmPolicy` est par défaut `"pairing"`.
-   `channels.irc.groupPolicy` est par défaut `"allowlist"`.
-   Avec `groupPolicy="allowlist"`, définissez `channels.irc.groups` pour spécifier les canaux autorisés.
-   Utilisez TLS (`channels.irc.tls=true`) sauf si vous acceptez intentionnellement un transport en clair.

## Contrôle d'accès

Il existe deux "portes" distinctes pour les canaux IRC :

1.  **Accès au canal** (`groupPolicy` + `groups`) : détermine si le bot accepte les messages provenant d'un canal.
2.  **Accès de l'expéditeur** (`groupAllowFrom` / `groups["#channel"].allowFrom` par canal) : détermine qui est autorisé à déclencher le bot dans ce canal.

Clés de configuration :

-   Liste d'autorisation pour les messages directs (accès expéditeur MP) : `channels.irc.allowFrom`
-   Liste d'autorisation des expéditeurs de groupe (accès expéditeur canal) : `channels.irc.groupAllowFrom`
-   Contrôles par canal (canal + expéditeur + règles de mention) : `channels.irc.groups["#channel"]`
-   `channels.irc.groupPolicy="open"` autorise les canaux non configurés (**toujours soumis aux mentions par défaut**)

Les entrées de la liste d'autorisation doivent utiliser des identités d'expéditeur stables (`nick!user@host`). La correspondance sur le simple pseudonyme est mutable et n'est activée que lorsque `channels.irc.dangerouslyAllowNameMatching: true`.

### Piège courant : allowFrom est pour les MP, pas pour les canaux

Si vous voyez des logs comme :

-   `irc: drop group sender alice!ident@host (policy=allowlist)`

…cela signifie que l'expéditeur n'était pas autorisé pour les messages de **groupe/canal**. Corrigez-le en :

-   définissant `channels.irc.groupAllowFrom` (global pour tous les canaux), ou
-   définissant des listes d'autorisation d'expéditeur par canal : `channels.irc.groups["#channel"].allowFrom`

Exemple (autoriser n'importe qui dans `#tuirc-dev` à parler au bot) :

```json
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": { allowFrom: ["*"] },
      },
    },
  },
}
```

## Déclenchement des réponses (mentions)

Même si un canal est autorisé (via `groupPolicy` + `groups`) et que l'expéditeur est autorisé, OpenClaw applique par défaut la **gestion par mentions** dans les contextes de groupe. Cela signifie que vous pouvez voir des logs comme `drop channel … (missing-mention)` sauf si le message inclut un motif de mention correspondant au bot. Pour que le bot réponde dans un canal IRC **sans avoir besoin d'une mention**, désactivez la gestion par mentions pour ce canal :

```json
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": {
          requireMention: false,
          allowFrom: ["*"],
        },
      },
    },
  },
}
```

Ou pour autoriser **tous** les canaux IRC (pas de liste d'autorisation par canal) et toujours répondre sans mentions :

```json
{
  channels: {
    irc: {
      groupPolicy: "open",
      groups: {
        "*": { requireMention: false, allowFrom: ["*"] },
      },
    },
  },
}
```

## Note de sécurité (recommandé pour les canaux publics)

Si vous autorisez `allowFrom: ["*"]` dans un canal public, n'importe qui peut solliciter le bot. Pour réduire les risques, restreignez les outils pour ce canal.

### Mêmes outils pour tout le monde dans le canal

```json
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          tools: {
            deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
          },
        },
      },
    },
  },
}
```

### Outils différents par expéditeur (le propriétaire a plus de pouvoir)

Utilisez `toolsBySender` pour appliquer une politique plus stricte à `"*"` et une plus souple à votre pseudonyme :

```json
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          toolsBySender: {
            "*": {
              deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
            },
            "id:eigen": {
              deny: ["gateway", "nodes", "cron"],
            },
          },
        },
      },
    },
  },
}
```

Notes :

-   Les clés de `toolsBySender` doivent utiliser `id:` pour les valeurs d'identité d'expéditeur IRC : `id:eigen` ou `id:eigen!~eigen@174.127.248.171` pour une correspondance plus forte.
-   Les clés non préfixées (héritées) sont toujours acceptées et correspondront uniquement en tant que `id:`.
-   La première politique d'expéditeur correspondante l'emporte ; `"*"` est la valeur par défaut générique.

Pour en savoir plus sur l'accès aux groupes vs la gestion par mentions (et leur interaction), consultez : [/channels/groups](./groups.md).

## NickServ

Pour s'identifier auprès de NickServ après la connexion :

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "enabled": true,
        "service": "NickServ",
        "password": "votre-mot-de-passe-nickserv"
      }
    }
  }
}
```

Inscription unique facultative lors de la connexion :

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "register": true,
        "registerEmail": "bot@example.com"
      }
    }
  }
}
```

Désactivez `register` après l'enregistrement du pseudonyme pour éviter des tentatives répétées de REGISTER.

## Variables d'environnement

Le compte par défaut prend en charge :

-   `IRC_HOST`
-   `IRC_PORT`
-   `IRC_TLS`
-   `IRC_NICK`
-   `IRC_USERNAME`
-   `IRC_REALNAME`
-   `IRC_PASSWORD`
-   `IRC_CHANNELS` (séparés par des virgules)
-   `IRC_NICKSERV_PASSWORD`
-   `IRC_NICKSERV_REGISTER_EMAIL`

## Dépannage

-   Si le bot se connecte mais ne répond jamais dans les canaux, vérifiez `channels.irc.groups` **et** si la gestion par mentions ignore les messages (`missing-mention`). Si vous voulez qu'il réponde sans ping, définissez `requireMention:false` pour le canal.
-   Si la connexion échoue, vérifiez la disponibilité du pseudonyme et le mot de passe du serveur.
-   Si TLS échoue sur un réseau personnalisé, vérifiez l'hôte/le port et la configuration du certificat.

[iMessage](./imessage.md)[LINE](./line.md)