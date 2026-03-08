title: "Guide d'installation d'OpenClaw avec le runtime expérimental Bun"
description: "Apprenez à installer et exécuter OpenClaw en utilisant Bun pour le développement local. Couvre l'installation, les commandes de build, les scripts de cycle de vie et les mises en garde importantes."
keywords: ["bun", "installation openclaw", "runtime expérimental", "bun install", "scripts de cycle de vie bun", "alternative pnpm", "runtime typescript"]
---

  Autres méthodes d'installation

  
# Bun (Expérimental)

Objectif : exécuter ce dépôt avec **Bun** (optionnel, non recommandé pour WhatsApp/Telegram) sans diverger des workflows pnpm. ⚠️ **Non recommandé pour le runtime Gateway** (bugs WhatsApp/Telegram). Utilisez Node pour la production.

## État

-   Bun est un runtime local optionnel pour exécuter TypeScript directement (`bun run …`, `bun --watch …`).
-   `pnpm` est la valeur par défaut pour les builds et reste entièrement pris en charge (et utilisé par certains outils de documentation).
-   Bun ne peut pas utiliser `pnpm-lock.yaml` et l'ignorera.

## Installer

Par défaut :

```bash
bun install
```

Note : `bun.lock`/`bun.lockb` sont ignorés par git, donc il n'y a pas de modifications dans le dépôt de toute façon. Si vous ne voulez *aucune écriture de fichier de verrouillage* :

```bash
bun install --no-save
```

## Build / Test (Bun)

```bash
bun run build
bun run vitest run
```

## Scripts de cycle de vie Bun (bloqués par défaut)

Bun peut bloquer les scripts de cycle de vie des dépendances à moins qu'ils ne soient explicitement approuvés (`bun pm untrusted` / `bun pm trust`). Pour ce dépôt, les scripts couramment bloqués ne sont pas requis :

-   `@whiskeysockets/baileys` `preinstall` : vérifie Node major >= 20 (nous utilisons Node 22+).
-   `protobufjs` `postinstall` : émet des avertissements concernant des schémas de version incompatibles (aucun artefact de build).

Si vous rencontrez un problème d'exécution réel nécessitant ces scripts, approuvez-les explicitement :

```bash
bun pm trust @whiskeysockets/baileys protobufjs
```

## Mises en garde

-   Certains scripts codent encore en dur pnpm (par ex. `docs:build`, `ui:*`, `protocol:check`). Exécutez-les via pnpm pour l'instant.

[Ansible](./ansible.md)[Mise à jour](./updating.md)

---