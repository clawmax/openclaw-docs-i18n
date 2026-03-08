

  Protocoles et APIs

  
# Backends CLI

OpenClaw peut exécuter des **CLIs d'IA locale** comme **solution de repli en mode texte uniquement** lorsque les fournisseurs d'API sont hors service, limités par des quotas, ou présentent temporairement des dysfonctionnements. Cette approche est intentionnellement conservatrice :

-   **Les outils sont désactivés** (pas d'appels d'outils).
-   **Texte en entrée → texte en sortie** (fiable).
-   **Les sessions sont prises en charge** (pour que les tours de suivi restent cohérents).
-   **Les images peuvent être transmises** si le CLI accepte les chemins d'images.

Ceci est conçu comme un **filet de sécurité** plutôt que comme un chemin principal. Utilisez-le lorsque vous voulez des réponses textuelles "qui fonctionnent toujours" sans dépendre d'APIs externes.

## Démarrage rapide pour débutants

Vous pouvez utiliser Claude Code CLI **sans aucune configuration** (OpenClaw inclut une configuration par défaut) :

```bash
openclaw agent --message "hi" --model claude-cli/opus-4.6
```

Codex CLI fonctionne également sans configuration :

```bash
openclaw agent --message "hi" --model codex-cli/gpt-5.4
```

Si votre passerelle s'exécute sous launchd/systemd et que le PATH est minimal, ajoutez simplement le chemin de la commande :

```json
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude",
        },
      },
    },
  },
}
```

C'est tout. Pas de clés, pas de configuration d'authentification supplémentaire nécessaire au-delà du CLI lui-même.

## Utilisation en tant que solution de repli

Ajoutez un backend CLI à votre liste de repli pour qu'il ne s'exécute que lorsque les modèles principaux échouent :

```json
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["claude-cli/opus-4.6", "claude-cli/opus-4.5"],
      },
      models: {
        "anthropic/claude-opus-4-6": { alias: "Opus" },
        "claude-cli/opus-4.6": {},
        "claude-cli/opus-4.5": {},
      },
    },
  },
}
```

Notes :

-   Si vous utilisez `agents.defaults.models` (liste autorisée), vous devez inclure `claude-cli/...`.
-   Si le fournisseur principal échoue (authentification, limites de débit, timeouts), OpenClaw essaiera ensuite le backend CLI.

## Aperçu de la configuration

Tous les backends CLI se trouvent sous :

```
agents.defaults.cliBackends
```

Chaque entrée est identifiée par un **id de fournisseur** (par exemple `claude-cli`, `my-cli`). L'id du fournisseur devient la partie gauche de votre référence de modèle :

```
<fournisseur>/<modèle>
```

### Exemple de configuration

```json
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude",
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          input: "arg",
          modelArg: "--model",
          modelAliases: {
            "claude-opus-4-6": "opus",
            "claude-opus-4-5": "opus",
            "claude-sonnet-4-5": "sonnet",
          },
          sessionArg: "--session",
          sessionMode: "existing",
          sessionIdFields: ["session_id", "conversation_id"],
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat",
          serialize: true,
        },
      },
    },
  },
}
```

## Fonctionnement

1.  **Sélectionne un backend** basé sur le préfixe du fournisseur (`claude-cli/...`).
2.  **Construit un prompt système** en utilisant le même prompt OpenClaw + le contexte de l'espace de travail.
3.  **Exécute le CLI** avec un id de session (si pris en charge) pour que l'historique reste cohérent.
4.  **Analyse la sortie** (JSON ou texte brut) et renvoie le texte final.
5.  **Persiste les ids de session** par backend, de sorte que les suivis réutilisent la même session CLI.

## Sessions

-   Si le CLI prend en charge les sessions, définissez `sessionArg` (par exemple `--session-id`) ou `sessionArgs` (espace réservé `{sessionId}`) lorsque l'ID doit être inséré dans plusieurs drapeaux.
-   Si le CLI utilise une **sous-commande de reprise** avec des drapeaux différents, définissez `resumeArgs` (remplace `args` lors de la reprise) et éventuellement `resumeOutput` (pour les reprises non-JSON).
-   `sessionMode` :
    -   `always` : envoie toujours un id de session (nouvel UUID si aucun n'est stocké).
    -   `existing` : n'envoie un id de session que si un était stocké auparavant.
    -   `none` : n'envoie jamais d'id de session.

## Images (passage)

Si votre CLI accepte les chemins d'images, définissez `imageArg` :

```yaml
imageArg: "--image",
imageMode: "repeat"
```

OpenClaw écrira les images base64 dans des fichiers temporaires. Si `imageArg` est défini, ces chemins sont passés comme arguments CLI. Si `imageArg` est absent, OpenClaw ajoute les chemins de fichiers au prompt (injection de chemin), ce qui suffit pour les CLIs qui chargent automatiquement les fichiers locaux à partir de chemins simples (comportement de Claude Code CLI).

## Entrées / Sorties

-   `output: "json"` (par défaut) essaie d'analyser le JSON et d'extraire le texte + l'id de session.
-   `output: "jsonl"` analyse les flux JSONL (Codex CLI `--json`) et extrait le dernier message de l'agent ainsi que `thread_id` lorsqu'il est présent.
-   `output: "text"` traite la sortie standard comme la réponse finale.

Modes d'entrée :

-   `input: "arg"` (par défaut) passe le prompt comme dernier argument CLI.
-   `input: "stdin"` envoie le prompt via stdin.
-   Si le prompt est très long et que `maxPromptArgChars` est défini, stdin est utilisé.

## Valeurs par défaut (intégrées)

OpenClaw inclut une configuration par défaut pour `claude-cli` :

-   `command: "claude"`
-   `args: ["-p", "--output-format", "json", "--permission-mode", "bypassPermissions"]`
-   `resumeArgs: ["-p", "--output-format", "json", "--permission-mode", "bypassPermissions", "--resume", "{sessionId}"]`
-   `modelArg: "--model"`
-   `systemPromptArg: "--append-system-prompt"`
-   `sessionArg: "--session-id"`
-   `systemPromptWhen: "first"`
-   `sessionMode: "always"`

OpenClaw inclut également une configuration par défaut pour `codex-cli` :

-   `command: "codex"`
-   `args: ["exec","--json","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
-   `resumeArgs: ["exec","resume","{sessionId}","--color","never","--sandbox","read-only","--skip-git-repo-check"]`
-   `output: "jsonl"`
-   `resumeOutput: "text"`
-   `modelArg: "--model"`
-   `imageArg: "--image"`
-   `sessionMode: "existing"`

Remplacez uniquement si nécessaire (cas courant : chemin absolu pour `command`).

## Limitations

-   **Pas d'outils OpenClaw** (le backend CLI ne reçoit jamais d'appels d'outils). Certains CLIs peuvent toujours exécuter leurs propres outils d'agent.
-   **Pas de streaming** (la sortie du CLI est collectée puis renvoyée).
-   **Les sorties structurées** dépendent du format JSON du CLI.
-   **Les sessions Codex CLI** reprennent via une sortie texte (pas de JSONL), ce qui est moins structuré que l'exécution initiale `--json`. Les sessions OpenClaw fonctionnent toujours normalement.

## Dépannage

-   **CLI introuvable** : définissez `command` sur un chemin complet.
-   **Nom de modèle incorrect** : utilisez `modelAliases` pour mapper `fournisseur/modèle` → modèle CLI.
-   **Pas de continuité de session** : assurez-vous que `sessionArg` est défini et que `sessionMode` n'est pas `none` (Codex CLI ne peut actuellement pas reprendre avec une sortie JSON).
-   **Images ignorées** : définissez `imageArg` (et vérifiez que le CLI prend en charge les chemins de fichiers).

[API d'Invocation d'Outils](./tools-invoke-http-api.md)[Modèles Locaux](./local-models.md)