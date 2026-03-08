

  Guides

  
# Configuration de l'assistant personnel

OpenClaw est une passerelle pour les agents **Pi** sur WhatsApp + Telegram + Discord + iMessage. Les plugins ajoutent Mattermost. Ce guide concerne la configuration « assistant personnel » : un numéro WhatsApp dédié qui se comporte comme votre agent toujours actif.

## ⚠️ Sécurité d'abord

Vous placez un agent dans une position où il peut :

-   exécuter des commandes sur votre machine (selon votre configuration des outils Pi)
-   lire/écrire des fichiers dans votre espace de travail
-   envoyer des messages via WhatsApp/Telegram/Discord/Mattermost (plugin)

Commencez de manière conservatrice :

-   Toujours définir `channels.whatsapp.allowFrom` (ne jamais exécuter ouvert au monde entier sur votre Mac personnel).
-   Utilisez un numéro WhatsApp dédié pour l'assistant.
-   Les battements de cœur sont maintenant par défaut toutes les 30 minutes. Désactivez-les jusqu'à ce que vous fassiez confiance à la configuration en définissant `agents.defaults.heartbeat.every: "0m"`.

## Prérequis

-   OpenClaw installé et intégré — voir [Premiers pas](./getting-started.md) si vous ne l'avez pas encore fait
-   Un deuxième numéro de téléphone (SIM/eSIM/prépayé) pour l'assistant

## La configuration à deux téléphones (recommandée)

C'est ce que vous voulez : Si vous liez votre WhatsApp personnel à OpenClaw, chaque message qui vous est adressé devient une « entrée d'agent ». Ce n'est rarement ce que vous souhaitez.

## Démarrage rapide en 5 minutes

1.  Associez WhatsApp Web (affiche un QR ; scannez avec le téléphone de l'assistant) :

```bash
openclaw channels login
```

2.  Démarrez la Passerelle (laissez-la tourner) :

```bash
openclaw gateway --port 18789
```

3.  Placez une configuration minimale dans `~/.openclaw/openclaw.json` :

```json
{
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

Maintenant, envoyez un message au numéro de l'assistant depuis votre téléphone autorisé. Lorsque l'intégration se termine, nous ouvrons automatiquement le tableau de bord et affichons un lien propre (non tokenisé). S'il demande une authentification, collez le jeton de `gateway.auth.token` dans les paramètres de l'interface de contrôle. Pour le rouvrir plus tard : `openclaw dashboard`.

## Donnez un espace de travail à l'agent (AGENTS)

OpenClaw lit les instructions opérationnelles et la « mémoire » depuis son répertoire d'espace de travail. Par défaut, OpenClaw utilise `~/.openclaw/workspace` comme espace de travail de l'agent, et le créera (ainsi que les fichiers de démarrage `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`) automatiquement lors de la configuration/première exécution de l'agent. `BOOTSTRAP.md` n'est créé que lorsque l'espace de travail est entièrement nouveau (il ne devrait pas réapparaître après l'avoir supprimé). `MEMORY.md` est optionnel (non créé automatiquement) ; lorsqu'il est présent, il est chargé pour les sessions normales. Les sessions de sous-agent n'injectent que `AGENTS.md` et `TOOLS.md`. Astuce : traitez ce dossier comme la « mémoire » d'OpenClaw et faites-en un dépôt git (idéalement privé) pour que vos fichiers `AGENTS.md` + mémoire soient sauvegardés. Si git est installé, les espaces de travail entièrement nouveaux sont auto-initialisés.

```bash
openclaw setup
```

Disposition complète de l'espace de travail + guide de sauvegarde : [Espace de travail de l'agent](../concepts/agent-workspace.md) Flux de travail de la mémoire : [Mémoire](../concepts/memory.md) Optionnel : choisissez un espace de travail différent avec `agents.defaults.workspace` (supporte `~`).

```json
{
  agent: {
    workspace: "~/.openclaw/workspace",
  },
}
```

Si vous fournissez déjà vos propres fichiers d'espace de travail depuis un dépôt, vous pouvez désactiver complètement la création de fichiers de démarrage :

```json
{
  agent: {
    skipBootstrap: true,
  },
}
```

## La configuration qui en fait « un assistant »

OpenClaw utilise par défaut une bonne configuration d'assistant, mais vous voudrez généralement ajuster :

-   la persona/les instructions dans `SOUL.md`
-   les paramètres de réflexion par défaut (si souhaité)
-   les battements de cœur (une fois que vous lui faites confiance)

Exemple :

```json
{
  logging: { level: "info" },
  agent: {
    model: "anthropic/claude-opus-4-6",
    workspace: "~/.openclaw/workspace",
    thinkingDefault: "high",
    timeoutSeconds: 1800,
    // Commencez par 0 ; activez plus tard.
    heartbeat: { every: "0m" },
  },
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true },
      },
    },
  },
  routing: {
    groupChat: {
      mentionPatterns: ["@openclaw", "openclaw"],
    },
  },
  session: {
    scope: "per-sender",
    resetTriggers: ["/new", "/reset"],
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 10080,
    },
  },
}
```

## Sessions et mémoire

-   Fichiers de session : `~/.openclaw/agents//sessions/{{SessionId}}.jsonl`
-   Métadonnées de session (utilisation de jetons, dernière route, etc.) : `~/.openclaw/agents//sessions/sessions.json` (hérité : `~/.openclaw/sessions/sessions.json`)
-   `/new` ou `/reset` démarre une nouvelle session pour ce chat (configurable via `resetTriggers`). S'il est envoyé seul, l'agent répond par un bref message de confirmation de la réinitialisation.
-   `/compact [instructions]` compacte le contexte de la session et rapporte le budget de contexte restant.

## Battements de cœur (mode proactif)

Par défaut, OpenClaw exécute un battement de cœur toutes les 30 minutes avec l'invite : `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.` Définissez `agents.defaults.heartbeat.every: "0m"` pour désactiver.

-   Si `HEARTBEAT.md` existe mais est effectivement vide (seulement des lignes vides et des en-têtes markdown comme `# Heading`), OpenClaw saute l'exécution du battement de cœur pour économiser des appels API.
-   Si le fichier est manquant, le battement de cœur s'exécute quand même et le modèle décide de ce qu'il faut faire.
-   Si l'agent répond par `HEARTBEAT_OK` (éventuellement avec un court remplissage ; voir `agents.defaults.heartbeat.ackMaxChars`), OpenClaw supprime l'envoi sortant pour ce battement de cœur.
-   Par défaut, la livraison des battements de cœur aux cibles de type DM `user:` est autorisée. Définissez `agents.defaults.heartbeat.directPolicy: "block"` pour supprimer la livraison directe tout en gardant les battements de cœur actifs.
-   Les battements de cœur exécutent des tours d'agent complets — des intervalles plus courts consomment plus de jetons.

```json
{
  agent: {
    heartbeat: { every: "30m" },
  },
}
```

## Médias entrants et sortants

Les pièces jointes entrantes (images/audio/documents) peuvent être exposées à votre commande via des modèles :

-   `{{MediaPath}}` (chemin du fichier temporaire local)
-   `{{MediaUrl}}` (URL pseudo)
-   `{{Transcript}}` (si la transcription audio est activée)

Pièces jointes sortantes de l'agent : incluez `MEDIA:<path-or-url>` sur sa propre ligne (sans espaces). Exemple :

```
Voici la capture d'écran.
MEDIA:https://example.com/screenshot.png
```

OpenClaw extrait ces éléments et les envoie en tant que médias avec le texte.

## Liste de contrôle des opérations

```bash
openclaw status          # statut local (identifiants, sessions, événements en file d'attente)
openclaw status --all    # diagnostic complet (lecture seule, collable)
openclaw status --deep   # ajoute des sondes de santé de la passerelle (Telegram + Discord)
openclaw health --json   # instantané de santé de la passerelle (WS)
```

Les journaux se trouvent sous `/tmp/openclaw/` (par défaut : `openclaw-YYYY-MM-DD.log`).

## Prochaines étapes

-   WebChat : [WebChat](../web/webchat.md)
-   Opérations de la passerelle : [Runbook de la passerelle](../gateway.md)
-   Cron + réveils : [Tâches Cron](../automation/cron-jobs.md)
-   Compagnon de la barre de menus macOS : [Application macOS OpenClaw](../platforms/macos.md)
-   Application node iOS : [Application iOS](../platforms/ios.md)
-   Application node Android : [Application Android](../platforms/android.md)
-   Statut Windows : [Windows (WSL2)](../platforms/windows.md)
-   Statut Linux : [Application Linux](../platforms/linux.md)
-   Sécurité : [Sécurité](../gateway/security.md)

[Intégration : Application macOS](./onboarding.md)[Référence CLI](./wizard-cli-reference.md)