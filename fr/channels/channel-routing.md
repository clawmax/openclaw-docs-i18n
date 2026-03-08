

  Configuration

  
# Routage des canaux

OpenClaw route les réponses **vers le canal d'où provient un message**. Le modèle ne choisit pas de canal ; le routage est déterministe et contrôlé par la configuration de l'hôte.

## Termes clés

-   **Canal** : `whatsapp`, `telegram`, `discord`, `slack`, `signal`, `imessage`, `webchat`.
-   **AccountId** : instance de compte par canal (lorsque pris en charge).
-   Compte par défaut optionnel du canal : `channels..defaultAccount` choisit quel compte est utilisé lorsqu'un chemin sortant ne spécifie pas `accountId`.
    -   Dans les configurations multi-comptes, définissez un compte par défaut explicite (`defaultAccount` ou `accounts.default`) lorsque deux comptes ou plus sont configurés. Sans cela, le routage de secours peut sélectionner le premier ID de compte normalisé.
-   **AgentId** : un espace de travail isolé + magasin de sessions ("cerveau").
-   **SessionKey** : la clé de compartiment utilisée pour stocker le contexte et contrôler la concurrence.

## Formes des clés de session (exemples)

Les messages directs sont regroupés dans la session **principale** de l'agent :

-   `agent::` (par défaut : `agent:main:main`)

Les groupes et canaux restent isolés par canal :

-   Groupes : `agent:::group:`
-   Canaux/salles : `agent:::channel:`

Fils de discussion :

-   Les fils Slack/Discord ajoutent `:thread:` à la clé de base.
-   Les sujets de forum Telegram intègrent `:topic:` dans la clé de groupe.

Exemples :

-   `agent:main:telegram:group:-1001234567890:topic:42`
-   `agent:main:discord:channel:123456:thread:987654`

## Épinglage de la route principale des messages directs

Lorsque `session.dmScope` est `main`, les messages directs peuvent partager une session principale unique. Pour empêcher la `lastRoute` de la session d'être écrasée par des messages directs provenant de non-propriétaires, OpenClaw déduit un propriétaire épinglé à partir de `allowFrom` lorsque toutes ces conditions sont vraies :

-   `allowFrom` a exactement une entrée non générique (non-wildcard).
-   L'entrée peut être normalisée en un ID d'expéditeur concret pour ce canal.
-   L'expéditeur du message direct entrant ne correspond pas à ce propriétaire épinglé.

Dans ce cas de non-correspondance, OpenClaw enregistre toujours les métadonnées de session entrantes, mais il ignore la mise à jour de la `lastRoute` de la session principale.

## Règles de routage (comment un agent est choisi)

Le routage sélectionne **un agent** pour chaque message entrant :

1.  **Correspondance exacte de pair** (`bindings` avec `peer.kind` + `peer.id`).
2.  **Correspondance de pair parent** (héritage de fil de discussion).
3.  **Correspondance guilde + rôles** (Discord) via `guildId` + `roles`.
4.  **Correspondance guilde** (Discord) via `guildId`.
5.  **Correspondance équipe** (Slack) via `teamId`.
6.  **Correspondance compte** (`accountId` sur le canal).
7.  **Correspondance canal** (n'importe quel compte sur ce canal, `accountId: "*"`).
8.  **Agent par défaut** (`agents.list[].default`, sinon première entrée de la liste, dernier recours `main`).

Lorsqu'une liaison inclut plusieurs champs de correspondance (`peer`, `guildId`, `teamId`, `roles`), **tous les champs fournis doivent correspondre** pour que cette liaison s'applique. L'agent correspondant détermine quel espace de travail et quel magasin de sessions sont utilisés.

## Groupes de diffusion (exécuter plusieurs agents)

Les groupes de diffusion vous permettent d'exécuter **plusieurs agents** pour le même pair **lorsqu'OpenClaw répondrait normalement** (par exemple : dans les groupes WhatsApp, après le filtrage par mention/activation). Configuration :

```json
{
  broadcast: {
    strategy: "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"],
    "+15555550123": ["support", "logger"],
  },
}
```

Voir : [Groupes de diffusion](./broadcast-groups.md).

## Aperçu de la configuration

-   `agents.list` : définitions d'agents nommés (espace de travail, modèle, etc.).
-   `bindings` : mappe les canaux/comptes/pairs entrants vers des agents.

Exemple :

```json
{
  agents: {
    list: [{ id: "support", name: "Support", workspace: "~/.openclaw/workspace-support" }],
  },
  bindings: [
    { match: { channel: "slack", teamId: "T123" }, agentId: "support" },
    { match: { channel: "telegram", peer: { kind: "group", id: "-100123" } }, agentId: "support" },
  ],
}
```

## Stockage des sessions

Les magasins de sessions se trouvent sous le répertoire d'état (par défaut `~/.openclaw`) :

-   `~/.openclaw/agents//sessions/sessions.json`
-   Les transcriptions JSONL se trouvent à côté du magasin

Vous pouvez remplacer le chemin du magasin via `session.store` et le modèle `{agentId}`.

## Comportement de WebChat

WebChat s'attache à **l'agent sélectionné** et utilise par défaut la session principale de cet agent. Pour cette raison, WebChat vous permet de voir le contexte multi-canal pour cet agent en un seul endroit.

## Contexte de réponse

Les réponses entrantes incluent :

-   `ReplyToId`, `ReplyToBody` et `ReplyToSender` lorsqu'ils sont disponibles.
-   Le contexte cité est ajouté au `Body` sous forme de bloc `[En réponse à ...]`.

Ceci est cohérent sur tous les canaux.

[Groupes de diffusion](./broadcast-groups.md)[Analyse de l'emplacement du canal](./location.md)

---