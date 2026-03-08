title: "Aides au débogage d'OpenClaw pour la sortie en flux et l'environnement"
description: "Apprenez à déboguer OpenClaw avec les surcharges d'exécution, les profils de développement, la journalisation des flux bruts et le mode surveillance de la passerelle pour une itération rapide."
keywords: ["débogage openclaw", "débogage sortie en flux", "surcharges d'exécution", "profil de développement", "mode surveillance passerelle", "journalisation flux brut", "journalisation pi-mono", "configuration environnement"]
---

  Environnement et débogage

  
# Débogage

Cette page couvre les aides au débogage pour la sortie en flux, en particulier lorsqu'un fournisseur mélange du raisonnement dans du texte normal.

## Surcharges de débogage à l'exécution

Utilisez `/debug` dans le chat pour définir des surcharges de configuration **uniquement à l'exécution** (en mémoire, pas sur disque). `/debug` est désactivé par défaut ; activez-le avec `commands.debug: true`. C'est pratique lorsque vous devez basculer des paramètres obscurs sans éditer `openclaw.json`. Exemples :

```bash
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug unset messages.responsePrefix
/debug reset
```

`/debug reset` efface toutes les surcharges et revient à la configuration sur disque.

## Mode surveillance de la passerelle

Pour une itération rapide, exécutez la passerelle sous le surveillant de fichiers :

```bash
pnpm gateway:watch
```

Ceci correspond à :

```bash
node --watch-path src --watch-path tsconfig.json --watch-path package.json --watch-preserve-output scripts/run-node.mjs gateway --force
```

Ajoutez tous les drapeaux CLI de la passerelle après `gateway:watch` et ils seront transmis à chaque redémarrage.

## Profil de développement + passerelle de développement (—dev)

Utilisez le profil de développement pour isoler l'état et démarrer une configuration sûre et jetable pour le débogage. Il y a **deux** drapeaux `--dev` :

-   **`--dev` global (profil) :** isole l'état sous `~/.openclaw-dev` et définit par défaut le port de la passerelle sur `19001` (les ports dérivés se décalent avec lui).
-   **`gateway --dev` :** indique à la Passerelle de créer automatiquement une configuration + espace de travail par défaut** lorsqu'ils sont manquants (et d'ignorer BOOTSTRAP.md).

Flux recommandé (profil de développement + amorçage de développement) :

```bash
pnpm gateway:dev
OPENCLAW_PROFILE=dev openclaw tui
```

Si vous n'avez pas encore d'installation globale, exécutez la CLI via `pnpm openclaw ...`. Ce que cela fait :

1.  **Isolation du profil** (`--dev` global)
    -   `OPENCLAW_PROFILE=dev`
    -   `OPENCLAW_STATE_DIR=~/.openclaw-dev`
    -   `OPENCLAW_CONFIG_PATH=~/.openclaw-dev/openclaw.json`
    -   `OPENCLAW_GATEWAY_PORT=19001` (le navigateur/canvas se décale en conséquence)
2.  **Amorçage de développement** (`gateway --dev`)
    -   Écrit une configuration minimale si elle est manquante (`gateway.mode=local`, liaison en boucle locale).
    -   Définit `agent.workspace` sur l'espace de travail de développement.
    -   Définit `agent.skipBootstrap=true` (pas de BOOTSTRAP.md).
    -   Initialise les fichiers de l'espace de travail s'ils sont manquants : `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`.
    -   Identité par défaut : **C3‑PO** (droïde de protocole).
    -   Ignore les fournisseurs de canaux en mode développement (`OPENCLAW_SKIP_CHANNELS=1`).

Flux de réinitialisation (démarrage frais) :

```bash
pnpm gateway:dev:reset
```

Note : `--dev` est un drapeau de profil **global** et est consommé par certains lanceurs. Si vous devez l'écrire explicitement, utilisez la forme variable d'environnement :

```
OPENCLAW_PROFILE=dev openclaw gateway --dev --reset
```

`--reset` efface la configuration, les identifiants, les sessions et l'espace de travail de développement (en utilisant `trash`, pas `rm`), puis recrée la configuration de développement par défaut. Astuce : si une passerelle non‑développement est déjà en cours d'exécution (launchd/systemd), arrêtez-la d'abord :

```bash
openclaw gateway stop
```

## Journalisation des flux bruts (OpenClaw)

OpenClaw peut journaliser **le flux brut de l'assistant** avant tout filtrage/mise en forme. C'est le meilleur moyen de voir si le raisonnement arrive sous forme de deltas de texte brut (ou en blocs de pensée séparés). Activez-le via la CLI :

```bash
pnpm gateway:watch --raw-stream
```

Chemin de surcharge optionnel :

```bash
pnpm gateway:watch --raw-stream --raw-stream-path ~/.openclaw/logs/raw-stream.jsonl
```

Variables d'environnement équivalentes :

```
OPENCLAW_RAW_STREAM=1
OPENCLAW_RAW_STREAM_PATH=~/.openclaw/logs/raw-stream.jsonl
```

Fichier par défaut : `~/.openclaw/logs/raw-stream.jsonl`

## Journalisation des morceaux bruts (pi-mono)

Pour capturer **les morceaux bruts compatibles OpenAI** avant qu'ils ne soient analysés en blocs, pi-mono expose un journaliseur séparé :

```
PI_RAW_STREAM=1
```

Chemin optionnel :

```
PI_RAW_STREAM_PATH=~/.pi-mono/logs/raw-openai-completions.jsonl
```

Fichier par défaut : `~/.pi-mono/logs/raw-openai-completions.jsonl`

> Note : ceci n'est émis que par les processus utilisant le fournisseur `openai-completions` de pi-mono.

## Notes de sécurité

-   Les journaux de flux bruts peuvent inclure des invites complètes, des sorties d'outils et des données utilisateur.
-   Conservez les journaux localement et supprimez-les après débogage.
-   Si vous partagez des journaux, nettoyez d'abord les secrets et les données personnelles.

[Variables d'environnement](./environment.md)[Tests](./testing.md)

---