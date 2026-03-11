

  Commandes CLI

  
# nodes

Gérez les nœuds (appareils) appairés et invoquez leurs capacités. Liens connexes :

-   Vue d'ensemble des nœuds : [Nœuds](../nodes.md)
-   Caméra : [Nœuds de caméra](../nodes/camera.md)
-   Images : [Nœuds d'images](../nodes/images.md)

Options communes :

-   `--url`, `--token`, `--timeout`, `--json`

## Commandes courantes

```bash
openclaw nodes list
openclaw nodes list --connected
openclaw nodes list --last-connected 24h
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes status
openclaw nodes status --connected
openclaw nodes status --last-connected 24h
```

`nodes list` affiche les tableaux des nœuds en attente/appairés. Les lignes des nœuds appairés incluent l'âge de la dernière connexion (Last Connect). Utilisez `--connected` pour n'afficher que les nœuds actuellement connectés. Utilisez `--last-connected <durée>` pour filtrer les nœuds qui se sont connectés dans une durée donnée (par ex. `24h`, `7d`).

## Invoquer / exécuter

```bash
openclaw nodes invoke --node <id|name|ip> --command <command> --params <json>
openclaw nodes run --node <id|name|ip> <command...>
openclaw nodes run --raw "git status"
openclaw nodes run --agent main --node <id|name|ip> --raw "git status"
```

Drapeaux d'invocation :

-   `--params ` : chaîne d'objet JSON (par défaut `{}`).
-   `--invoke-timeout ` : délai d'expiration de l'invocation du nœud (par défaut `15000`).
-   `--idempotency-key <clé>` : clé d'idempotence optionnelle.

### Valeurs par défaut de type exec

`nodes run` reflète le comportement exec du modèle (valeurs par défaut + approbations) :

-   Lit `tools.exec.*` (plus les surcharges `agents.list[].tools.exec.*`).
-   Utilise les approbations exec (`exec.approval.request`) avant d'invoquer `system.run`.
-   `--node` peut être omis lorsque `tools.exec.node` est défini.
-   Requiert un nœud qui propose `system.run` (application compagnon macOS ou hôte de nœud headless).

Drapeaux :

-   `--cwd ` : répertoire de travail.
-   `--env <clé=valeur>` : surcharge d'environnement (répétable). Note : les hôtes de nœud ignorent les surcharges `PATH` (et `tools.exec.pathPrepend` n'est pas appliqué aux hôtes de nœud).
-   `--command-timeout ` : délai d'expiration de la commande.
-   `--invoke-timeout ` : délai d'expiration de l'invocation du nœud (par défaut `30000`).
-   `--needs-screen-recording` : exige l'autorisation d'enregistrement d'écran.
-   `--raw ` : exécute une chaîne shell (`/bin/sh -lc` ou `cmd.exe /c`). En mode liste blanche sur les hôtes de nœud Windows, les exécutions avec wrapper shell `cmd.exe /c` nécessitent une approbation (l'entrée dans la liste blanche seule n'autorise pas automatiquement la forme avec wrapper).
-   `--agent ` : approbations/listes blanches limitées à l'agent (par défaut l'agent configuré).
-   `--ask <off|on-miss|always>`, `--security <deny|allowlist|full>` : surcharges.

[node](./node.md)[onboard](./onboard.md)

---