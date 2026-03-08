

  Commandes CLI

  
# agents

Gérez les agents isolés (espace de travail + authentification + routage). Liens connexes :

-   Routage multi-agent : [Routage Multi-Agent](../concepts/multi-agent.md)
-   Espace de travail agent : [Espace de travail agent](../concepts/agent-workspace.md)

## Exemples

```bash
openclaw agents list
openclaw agents add work --workspace ~/.openclaw/workspace-work
openclaw agents bindings
openclaw agents bind --agent work --bind telegram:ops
openclaw agents unbind --agent work --bind telegram:ops
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
openclaw agents set-identity --agent main --avatar avatars/openclaw.png
openclaw agents delete work
```

## Liaisons de routage

Utilisez les liaisons de routage pour diriger le trafic entrant d'un canal vers un agent spécifique. Lister les liaisons :

```bash
openclaw agents bindings
openclaw agents bindings --agent work
openclaw agents bindings --json
```

Ajouter des liaisons :

```bash
openclaw agents bind --agent work --bind telegram:ops --bind discord:guild-a
```

Si vous omettez `accountId` (`--bind `), OpenClaw le résout à partir des valeurs par défaut du canal et des hooks de configuration du plugin lorsque disponibles.

### Comportement de la portée des liaisons

-   Une liaison sans `accountId` correspond uniquement au compte par défaut du canal.
-   `accountId: "*"` est le fallback à l'échelle du canal (tous les comptes) et est moins spécifique qu'une liaison explicite à un compte.
-   Si le même agent a déjà une liaison de canal correspondante sans `accountId`, et que vous liez plus tard avec un `accountId` explicite ou résolu, OpenClaw met à niveau cette liaison existante sur place au lieu d'en ajouter une en double.

Exemple :

```bash
# liaison initiale uniquement au canal
openclaw agents bind --agent work --bind telegram

# mise à niveau ultérieure vers une liaison limitée au compte
openclaw agents bind --agent work --bind telegram:ops
```

Après la mise à niveau, le routage pour cette liaison est limité à `telegram:ops`. Si vous souhaitez également un routage pour le compte par défaut, ajoutez-le explicitement (par exemple `--bind telegram:default`). Supprimer des liaisons :

```bash
openclaw agents unbind --agent work --bind telegram:ops
openclaw agents unbind --agent work --all
```

## Fichiers d'identité

Chaque espace de travail d'agent peut inclure un fichier `IDENTITY.md` à la racine de l'espace de travail :

-   Exemple de chemin : `~/.openclaw/workspace/IDENTITY.md`
-   `set-identity --from-identity` lit depuis la racine de l'espace de travail (ou un `--identity-file` explicite)

Les chemins d'avatar sont résolus relativement à la racine de l'espace de travail.

## Définir l'identité

`set-identity` écrit les champs dans `agents.list[].identity` :

-   `name`
-   `theme`
-   `emoji`
-   `avatar` (chemin relatif à l'espace de travail, URL http(s) ou URI de données)

Charger depuis `IDENTITY.md` :

```bash
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
```

Surcharger les champs explicitement :

```bash
openclaw agents set-identity --agent main --name "OpenClaw" --emoji "🦞" --avatar avatars/openclaw.png
```

Exemple de configuration :

```json
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "OpenClaw",
          theme: "space lobster",
          emoji: "🦞",
          avatar: "avatars/openclaw.png",
        },
      },
    ],
  },
}
```

[agent](./agent.md)[approvals](./approvals.md)