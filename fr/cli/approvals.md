

  Commandes CLI

  
# approvals

Gérez les approbations d'exécution pour l'**hôte local**, l'**hôte de la passerelle** ou un **hôte de nœud**. Par défaut, les commandes ciblent le fichier d'approbations local sur le disque. Utilisez `--gateway` pour cibler la passerelle, ou `--node` pour cibler un nœud spécifique. Liens connexes :

-   Approbations d'exécution : [Approbations d'exécution](../tools/exec-approvals.md)
-   Nœuds : [Nœuds](../nodes.md)

## Commandes courantes

```bash
openclaw approvals get
openclaw approvals get --node <id|name|ip>
openclaw approvals get --gateway
```

## Remplacer les approbations depuis un fichier

```bash
openclaw approvals set --file ./exec-approvals.json
openclaw approvals set --node <id|name|ip> --file ./exec-approvals.json
openclaw approvals set --gateway --file ./exec-approvals.json
```

## Assistants de liste autorisée

```bash
openclaw approvals allowlist add "~/Projects/**/bin/rg"
openclaw approvals allowlist add --agent main --node <id|name|ip> "/usr/bin/uptime"
openclaw approvals allowlist add --agent "*" "/usr/bin/uname"

openclaw approvals allowlist remove "~/Projects/**/bin/rg"
```

## Notes

-   `--node` utilise le même résolveur que `openclaw nodes` (id, nom, ip ou préfixe d'id).
-   `--agent` a par défaut la valeur `"*"`, ce qui s'applique à tous les agents.
-   L'hôte du nœud doit annoncer `system.execApprovals.get/set` (application macOS ou hôte de nœud headless).
-   Les fichiers d'approbations sont stockés par hôte dans `~/.openclaw/exec-approvals.json`.

[agents](./agents.md)[browser](./browser.md)

---