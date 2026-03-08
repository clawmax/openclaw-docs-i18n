title: "Commandes d'appairage OpenClaw CLI pour l'approbation et l'inspection des canaux"
description: "Apprenez à utiliser les commandes d'appairage de l'interface CLI OpenClaw pour lister, inspecter et approuver les demandes d'appairage en DM pour les canaux pris en charge comme Telegram."
keywords: ["appairage openclaw", "commandes cli", "appairage dm", "approbation de canal", "appairage telegram", "liste d'appairage", "approuver l'appairage", "canaux multi-comptes"]
---

  Commandes CLI

  
# pairing

Approuvez ou inspectez les demandes d'appairage en DM (pour les canaux qui prennent en charge l'appairage). Liens connexes :

-   Flux d'appairage : [Appairage](../channels/pairing.md)

## Commandes

```bash
openclaw pairing list telegram
openclaw pairing list --channel telegram --account work
openclaw pairing list telegram --json

openclaw pairing approve telegram <code>
openclaw pairing approve --channel telegram --account work <code> --notify
```

## Notes

-   Saisie du canal : passez-le positionnellement (`pairing list telegram`) ou avec `--channel `.
-   `pairing list` prend en charge `--account ` pour les canaux multi-comptes.
-   `pairing approve` prend en charge `--account ` et `--notify`.
-   Si un seul canal compatible avec l'appairage est configuré, `pairing approve ` est autorisé.

[onboard](./onboard.md)[plugins](./plugins.md)

---