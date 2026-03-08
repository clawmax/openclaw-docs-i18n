title: "Groupes de configuration OpenClaw pour WhatsApp Telegram Discord Slack"
description: "Apprenez à configurer l'accès aux groupes de discussion OpenClaw, le contrôle par mention et les politiques de session sur WhatsApp, Telegram, Discord, Slack et d'autres plateformes de messagerie."
keywords: ["groupes openclaw", "configuration de politique de groupe", "contrôle par mention", "groupes multi-canaux", "clés de session", "liste autorisée de groupes", "groupes sandbox", "intégration de plateforme de messagerie"]
---

  Configuration

  
# Groupes

OpenClaw traite les groupes de discussion de manière cohérente sur toutes les surfaces : WhatsApp, Telegram, Discord, Slack, Signal, iMessage, Microsoft Teams, Zalo.

## Introduction pour débutants (2 minutes)

OpenClaw "vit" sur vos propres comptes de messagerie. Il n'y a pas d'utilisateur bot WhatsApp séparé. Si **vous** êtes dans un groupe, OpenClaw peut voir ce groupe et y répondre. Comportement par défaut :

-   Les groupes sont restreints (`groupPolicy: "allowlist"`).
-   Les réponses nécessitent une mention sauf si vous désactivez explicitement le contrôle par mention.

Traduction : les expéditeurs autorisés peuvent déclencher OpenClaw en le mentionnant.

> TL;DR
>
> -   **Accès en MP** est contrôlé par `*.allowFrom`.
> -   **Accès aux groupes** est contrôlé par `*.groupPolicy` + les listes autorisées (`*.groups`, `*.groupAllowFrom`).
> -   **Déclenchement des réponses** est contrôlé par le contrôle par mention (`requireMention`, `/activation`).

Flux rapide (ce qui arrive à un message de groupe) :

```
groupPolicy? disabled -> drop
groupPolicy? allowlist -> group allowed? no -> drop
requireMention? yes -> mentioned? no -> store for context only
otherwise -> reply
```

![Flux des messages de groupe](../images/channels-groups-flow.svg.md) Si vous voulez…

| Objectif | Ce qu'il faut définir |
| --- | --- |
| Autoriser tous les groupes mais ne répondre que sur les @mentions | `groups: { "*": { requireMention: true } }` |
| Désactiver toutes les réponses de groupe | `groupPolicy: "disabled"` |
| Seulement des groupes spécifiques | `groups: { "<group-id>": { ... } }` (pas de clé `"*"`) |
| Seulement vous pouvez déclencher dans les groupes | `groupPolicy: "allowlist"`, `groupAllowFrom: ["+1555..."]` |

## Clés de session

-   Les sessions de groupe utilisent les clés de session `agent:::group:` (les salons/canaux utilisent `agent:::channel:`).
-   Les sujets de forum Telegram ajoutent `:topic:` à l'id de groupe pour que chaque sujet ait sa propre session.
-   Les discussions directes utilisent la session principale (ou par expéditeur si configuré).
-   Les heartbeats sont ignorés pour les sessions de groupe.

## Modèle : MPs personnels + groupes publics (agent unique)

Oui — cela fonctionne bien si votre trafic "personnel" est en **MP** et votre trafic "public" est dans des **groupes**. Pourquoi : en mode mono-agent, les MPs atterrissent généralement dans la clé de session **principale** (`agent:main:main`), tandis que les groupes utilisent toujours des clés de session **non principales** (`agent:main::group:`). Si vous activez le sandboxing avec `mode: "non-main"`, ces sessions de groupe s'exécutent dans Docker tandis que votre session principale de MP reste sur l'hôte. Cela vous donne un "cerveau" d'agent (espace de travail + mémoire partagés), mais deux postures d'exécution :

-   **MPs** : outils complets (hôte)
-   **Groupes** : sandbox + outils restreints (Docker)

> Si vous avez besoin d'espaces de travail/personas vraiment séparés ("personnel" et "public" ne doivent jamais se mélanger), utilisez un deuxième agent + des liaisons. Voir [Routage Multi-Agent](../concepts/multi-agent.md).

Exemple (MPs sur l'hôte, groupes en sandbox + outils de messagerie uniquement) :

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // les groupes/canaux sont non principaux -> sandboxés
        scope: "session", // isolation la plus forte (un conteneur par groupe/canal)
        workspaceAccess: "none",
      },
    },
  },
  tools: {
    sandbox: {
      tools: {
        // Si allow n'est pas vide, tout le reste est bloqué (deny l'emporte toujours).
        allow: ["group:messaging", "group:sessions"],
        deny: ["group:runtime", "group:fs", "group:ui", "nodes", "cron", "gateway"],
      },
    },
  },
}
```

Vous voulez "les groupes ne peuvent voir que le dossier X" au lieu de "pas d'accès à l'hôte" ? Gardez `workspaceAccess: "none"` et montez uniquement les chemins autorisés dans le sandbox :

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none",
        docker: {
          binds: [
            // hostPath:containerPath:mode
            "/home/user/FriendsShared:/data:ro",
          ],
        },
      },
    },
  },
}
```

Liens connexes :

-   Clés de configuration et valeurs par défaut : [Configuration de la passerelle](../gateway/configuration.md#agentsdefaultssandbox)
-   Déboguer pourquoi un outil est bloqué : [Sandbox vs Politique d'outil vs Élevé](../gateway/sandbox-vs-tool-policy-vs-elevated.md)
-   Détails des montages de liaison : [Sandboxing](../gateway/sandboxing.md#custom-bind-mounts)

## Étiquettes d'affichage

-   Les étiquettes de l'interface utilisateur utilisent `displayName` quand disponible, formatées comme `:`.
-   `#room` est réservé aux salons/canaux ; les groupes de discussion utilisent `g-` (minuscules, espaces -> `-`, garde `#@+._-`).

## Politique de groupe

Contrôlez comment les messages de groupe/salon sont traités par canal :

```json
{
  channels: {
    whatsapp: {
      groupPolicy: "disabled", // "open" | "disabled" | "allowlist"
      groupAllowFrom: ["+15551234567"],
    },
    telegram: {
      groupPolicy: "disabled",
      groupAllowFrom: ["123456789"], // identifiant utilisateur Telegram numérique (l'assistant peut résoudre @username)
    },
    signal: {
      groupPolicy: "disabled",
      groupAllowFrom: ["+15551234567"],
    },
    imessage: {
      groupPolicy: "disabled",
      groupAllowFrom: ["chat_id:123"],
    },
    msteams: {
      groupPolicy: "disabled",
      groupAllowFrom: ["user@org.com"],
    },
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        GUILD_ID: { channels: { help: { allow: true } } },
      },
    },
    slack: {
      groupPolicy: "allowlist",
      channels: { "#general": { allow: true } },
    },
    matrix: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["@owner:example.org"],
      groups: {
        "!roomId:example.org": { allow: true },
        "#alias:example.org": { allow: true },
      },
    },
  },
}
```

| Politique | Comportement |
| --- | --- |
| `"open"` | Les groupes contournent les listes autorisées ; le contrôle par mention s'applique toujours. |
| `"disabled"` | Bloque tous les messages de groupe entièrement. |
| `"allowlist"` | Autorise uniquement les groupes/salons qui correspondent à la liste autorisée configurée. |

Notes :

-   `groupPolicy` est séparé du contrôle par mention (qui nécessite des @mentions).
-   WhatsApp/Telegram/Signal/iMessage/Microsoft Teams/Zalo : utilisez `groupAllowFrom` (fallback : `allowFrom` explicite).
-   Les approbations d'appariement en MP (`*-allowFrom` entrées de stockage) s'appliquent uniquement à l'accès en MP ; l'autorisation de l'expéditeur de groupe reste explicite aux listes autorisées de groupe.
-   Discord : la liste autorisée utilise `channels.discord.guilds..channels`.
-   Slack : la liste autorisée utilise `channels.slack.channels`.
-   Matrix : la liste autorisée utilise `channels.matrix.groups` (IDs de salon, alias ou noms). Utilisez `channels.matrix.groupAllowFrom` pour restreindre les expéditeurs ; les listes autorisées `users` par salon sont également prises en charge.
-   Les MPs de groupe sont contrôlés séparément (`channels.discord.dm.*`, `channels.slack.dm.*`).
-   La liste autorisée Telegram peut correspondre aux IDs utilisateur (`"123456789"`, `"telegram:123456789"`, `"tg:123456789"`) ou aux noms d'utilisateur (`"@alice"` ou `"alice"`) ; les préfixes ne sont pas sensibles à la casse.
-   La valeur par défaut est `groupPolicy: "allowlist"` ; si votre liste autorisée de groupe est vide, les messages de groupe sont bloqués.
-   Sécurité d'exécution : quand un bloc de fournisseur est complètement absent (`channels.` absent), la politique de groupe revient à un mode fail-closed (typiquement `allowlist`) au lieu d'hériter de `channels.defaults.groupPolicy`.

Modèle mental rapide (ordre d'évaluation pour les messages de groupe) :

1.  `groupPolicy` (open/disabled/allowlist)
2.  Listes autorisées de groupe (`*.groups`, `*.groupAllowFrom`, liste autorisée spécifique au canal)
3.  Contrôle par mention (`requireMention`, `/activation`)

## Contrôle par mention (par défaut)

Les messages de groupe nécessitent une mention sauf remplacement par groupe. Les valeurs par défaut vivent par sous-système sous `*.groups."*"`. Répondre à un message du bot compte comme une mention implicite (quand le canal prend en charge les métadonnées de réponse). Cela s'applique à Telegram, WhatsApp, Slack, Discord et Microsoft Teams.

```json
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true },
        "123@g.us": { requireMention: false },
      },
    },
    telegram: {
      groups: {
        "*": { requireMention: true },
        "123456789": { requireMention: false },
      },
    },
    imessage: {
      groups: {
        "*": { requireMention: true },
        "123": { requireMention: false },
      },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          mentionPatterns: ["@openclaw", "openclaw", "\\+15555550123"],
          historyLimit: 50,
        },
      },
    ],
  },
}
```

Notes :

-   `mentionPatterns` sont des regex insensibles à la casse.
-   Les surfaces qui fournissent des mentions explicites passent toujours ; les motifs sont un fallback.
-   Remplacement par agent : `agents.list[].groupChat.mentionPatterns` (utile quand plusieurs agents partagent un groupe).
-   Le contrôle par mention n'est appliqué que lorsque la détection de mention est possible (mentions natives ou `mentionPatterns` configurés).
-   Les valeurs par défaut Discord vivent dans `channels.discord.guilds."*"` (remplaçable par serveur/canal).
-   Le contexte d'historique de groupe est encapsulé uniformément sur tous les canaux et est **uniquement en attente** (messages ignorés à cause du contrôle par mention) ; utilisez `messages.groupChat.historyLimit` pour la valeur par défaut globale et `channels..historyLimit` (ou `channels..accounts.*.historyLimit`) pour les remplacements. Mettez `0` pour désactiver.

## Restrictions d'outils par groupe/canal (optionnel)

Certaines configurations de canal prennent en charge la restriction des outils disponibles **à l'intérieur d'un groupe/salon/canal spécifique**.

-   `tools` : autoriser/interdire des outils pour tout le groupe.
-   `toolsBySender` : remplacements par expéditeur au sein du groupe. Utilisez des préfixes de clé explicites : `id:`, `e164:`, `username:`, `name:`, et le caractère générique `"*"`. Les clés non préfixées héritées sont toujours acceptées et correspondront uniquement à `id:`.

Ordre de résolution (le plus spécifique l'emporte) :

1.  Correspondance `toolsBySender` du groupe/canal
2.  `tools` du groupe/canal
3.  Correspondance `toolsBySender` par défaut (`"*"`)
4.  `tools` par défaut (`"*"`)

Exemple (Telegram) :

```json
{
  channels: {
    telegram: {
      groups: {
        "*": { tools: { deny: ["exec"] } },
        "-1001234567890": {
          tools: { deny: ["exec", "read", "write"] },
          toolsBySender: {
            "id:123456789": { alsoAllow: ["exec"] },
          },
        },
      },
    },
  },
}
```

Notes :

-   Les restrictions d'outils par groupe/canal sont appliquées en plus de la politique d'outils globale/agent (deny l'emporte toujours).
-   Certains canaux utilisent un emboîtement différent pour les salons/canaux (par ex., Discord `guilds.*.channels.*`, Slack `channels.*`, MS Teams `teams.*.channels.*`).

## Listes autorisées de groupe

Quand `channels.whatsapp.groups`, `channels.telegram.groups`, ou `channels.imessage.groups` est configuré, les clés agissent comme une liste autorisée de groupe. Utilisez `"*"` pour autoriser tous les groupes tout en définissant le comportement de mention par défaut. Intentions courantes (copier/coller) :

1.  Désactiver toutes les réponses de groupe

```json
{
  channels: { whatsapp: { groupPolicy: "disabled" } },
}
```

2.  Autoriser uniquement des groupes spécifiques (WhatsApp)

```json
{
  channels: {
    whatsapp: {
      groups: {
        "123@g.us": { requireMention: true },
        "456@g.us": { requireMention: false },
      },
    },
  },
}
```

3.  Autoriser tous les groupes mais nécessiter une mention (explicite)

```json
{
  channels: {
    whatsapp: {
      groups: { "*": { requireMention: true } },
    },
  },
}
```

4.  Seul le propriétaire peut déclencher dans les groupes (WhatsApp)

```json
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
      groups: { "*": { requireMention: true } },
    },
  },
}
```

## Activation (propriétaire uniquement)

Les propriétaires de groupe peuvent activer/désactiver l'activation par groupe :

-   `/activation mention`
-   `/activation always`

Le propriétaire est déterminé par `channels.whatsapp.allowFrom` (ou l'E.164 du bot lui-même quand non défini). Envoyez la commande comme un message autonome. Les autres surfaces ignorent actuellement `/activation`.

## Champs de contexte

Les charges utiles entrantes de groupe définissent :

-   `ChatType=group`
-   `GroupSubject` (si connu)
-   `GroupMembers` (si connu)
-   `WasMentioned` (résultat du contrôle par mention)
-   Les sujets de forum Telegram incluent également `MessageThreadId` et `IsForum`.

L'invite système de l'agent inclut une introduction de groupe au premier tour d'une nouvelle session de groupe. Elle rappelle au modèle de répondre comme un humain, d'éviter les tableaux Markdown et d'éviter de taper des séquences littérales `\n`.

## Spécificités iMessage

-   Préférez `chat_id:` pour le routage ou les listes autorisées.
-   Lister les discussions : `imsg chats --limit 20`.
-   Les réponses de groupe reviennent toujours au même `chat_id`.

## Spécificités WhatsApp

Voir [Messages de groupe](./group-messages.md) pour le comportement spécifique à WhatsApp (injection d'historique, détails de gestion des mentions).

[Messages de groupe](./group-messages.md)[Groupes de diffusion](./broadcast-groups.md)

---