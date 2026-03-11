

  Automatisation

  
# Gmail PubSub

Objectif : Surveillance Gmail -> Push Pub/Sub -> `gog gmail watch serve` -> Webhook OpenClaw.

## Prérequis

-   `gcloud` installé et connecté ([guide d'installation](https://docs.cloud.google.com/sdk/docs/install-sdk)).
-   `gog` (gogcli) installé et autorisé pour le compte Gmail ([gogcli.sh](https://gogcli.sh/)).
-   Hooks OpenClaw activés (voir [Webhooks](./webhook.md)).
-   `tailscale` connecté ([tailscale.com](https://tailscale.com/)). La configuration prise en charge utilise Tailscale Funnel pour le point de terminaison HTTPS public. D'autres services de tunnel peuvent fonctionner, mais sont DIY/non pris en charge et nécessitent un câblage manuel. Actuellement, Tailscale est ce que nous supportons.

Exemple de configuration de hook (activer le mapping prédéfini Gmail) :

```json
{
  hooks: {
    enabled: true,
    token: "OPENCLAW_HOOK_TOKEN",
    path: "/hooks",
    presets: ["gmail"],
  },
}
```

Pour livrer le résumé Gmail vers une surface de chat, remplacez le préréglage par un mapping qui définit `deliver` + optionnellement `channel`/`to` :

```json
{
  hooks: {
    enabled: true,
    token: "OPENCLAW_HOOK_TOKEN",
    presets: ["gmail"],
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate: "Nouvel email de {{messages[0].from}}\nObjet : {{messages[0].subject}}\n{{messages[0].snippet}}\n{{messages[0].body}}",
        model: "openai/gpt-5.2-mini",
        deliver: true,
        channel: "last",
        // to: "+15551234567"
      },
    ],
  },
}
```

Si vous voulez un canal fixe, définissez `channel` + `to`. Sinon, `channel: "last"` utilise la dernière route de livraison (retombe sur WhatsApp). Pour forcer un modèle moins coûteux pour les exécutions Gmail, définissez `model` dans le mapping (`provider/model` ou alias). Si vous appliquez `agents.defaults.models`, incluez-le là. Pour définir un modèle par défaut et un niveau de réflexion spécifiquement pour les hooks Gmail, ajoutez `hooks.gmail.model` / `hooks.gmail.thinking` dans votre configuration :

```json
{
  hooks: {
    gmail: {
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      thinking: "off",
    },
  },
}
```

Notes :

-   Le `model`/`thinking` par hook dans le mapping prime toujours sur ces valeurs par défaut.
-   Ordre de secours : `hooks.gmail.model` → `agents.defaults.model.fallbacks` → principal (auth/limite de débit/timeouts).
-   Si `agents.defaults.models` est défini, le modèle Gmail doit être dans la liste autorisée.
-   Le contenu du hook Gmail est encapsulé par des limites de sécurité de contenu externe par défaut. Pour désactiver (dangereux), définissez `hooks.gmail.allowUnsafeExternalContent: true`.

Pour personnaliser davantage le traitement des charges utiles, ajoutez `hooks.mappings` ou un module de transformation JS/TS sous `~/.openclaw/hooks/transforms` (voir [Webhooks](./webhook.md)).

## Assistant (recommandé)

Utilisez l'assistant OpenClaw pour tout connecter ensemble (installe les dépendances sur macOS via brew) :

```bash
openclaw webhooks gmail setup \
  --account openclaw@gmail.com
```

Par défaut :

-   Utilise Tailscale Funnel pour le point de terminaison push public.
-   Écrit la configuration `hooks.gmail` pour `openclaw webhooks gmail run`.
-   Active le préréglage de hook Gmail (`hooks.presets: ["gmail"]`).

Note sur le chemin : lorsque `tailscale.mode` est activé, OpenClaw définit automatiquement `hooks.gmail.serve.path` sur `/` et garde le chemin public à `hooks.gmail.tailscale.path` (par défaut `/gmail-pubsub`) car Tailscale supprime le préfixe de chemin défini avant de proxyfier. Si vous avez besoin que le backend reçoive le chemin préfixé, définissez `hooks.gmail.tailscale.target` (ou `--tailscale-target`) sur une URL complète comme `http://127.0.0.1:8788/gmail-pubsub` et faites correspondre `hooks.gmail.serve.path`. Vous voulez un point de terminaison personnalisé ? Utilisez `--push-endpoint ` ou `--tailscale off`. Note plateforme : sur macOS, l'assistant installe `gcloud`, `gogcli` et `tailscale` via Homebrew ; sur Linux, installez-les manuellement d'abord. Démarrage automatique de la passerelle (recommandé) :

-   Lorsque `hooks.enabled=true` et `hooks.gmail.account` est défini, la Passerelle démarre `gog gmail watch serve` au démarrage et renouvelle automatiquement la surveillance.
-   Définissez `OPENCLAW_SKIP_GMAIL_WATCHER=1` pour vous exclure (utile si vous exécutez le démon vous-même).
-   N'exécutez pas le démon manuel en même temps, sinon vous obtiendrez `listen tcp 127.0.0.1:8788: bind: address already in use`.

Démon manuel (démarre `gog gmail watch serve` + renouvellement automatique) :

```bash
openclaw webhooks gmail run
```

## Configuration unique

1.  Sélectionnez le projet GCP **qui possède le client OAuth** utilisé par `gog`.

```bash
gcloud auth login
gcloud config set project <project-id>
```

Note : La surveillance Gmail nécessite que le sujet Pub/Sub réside dans le même projet que le client OAuth.

2.  Activez les API :

```bash
gcloud services enable gmail.googleapis.com pubsub.googleapis.com
```

3.  Créez un sujet :

```bash
gcloud pubsub topics create gog-gmail-watch
```

4.  Autorisez Gmail push à publier :

```
gcloud pubsub topics add-iam-policy-binding gog-gmail-watch \
  --member=serviceAccount:gmail-api-push@system.gserviceaccount.com \
  --role=roles/pubsub.publisher
```

## Démarrer la surveillance

```
gog gmail watch start \
  --account openclaw@gmail.com \
  --label INBOX \
  --topic projects/<project-id>/topics/gog-gmail-watch
```

Sauvegardez le `history_id` de la sortie (pour le débogage).

## Exécuter le gestionnaire de push

Exemple local (authentification par token partagé) :

```
gog gmail watch serve \
  --account openclaw@gmail.com \
  --bind 127.0.0.1 \
  --port 8788 \
  --path /gmail-pubsub \
  --token <shared> \
  --hook-url http://127.0.0.1:18789/hooks/gmail \
  --hook-token OPENCLAW_HOOK_TOKEN \
  --include-body \
  --max-bytes 20000
```

Notes :

-   `--token` protège le point de terminaison push (`x-gog-token` ou `?token=`).
-   `--hook-url` pointe vers OpenClaw `/hooks/gmail` (mappé ; exécution isolée + résumé vers le principal).
-   `--include-body` et `--max-bytes` contrôlent l'extrait du corps envoyé à OpenClaw.

Recommandé : `openclaw webhooks gmail run` encapsule le même flux et renouvelle automatiquement la surveillance.

## Exposer le gestionnaire (avancé, non supporté)

Si vous avez besoin d'un tunnel non-Tailscale, câblez-le manuellement et utilisez l'URL publique dans l'abonnement push (non supporté, sans garde-fous) :

```bash
cloudflared tunnel --url http://127.0.0.1:8788 --no-autoupdate
```

Utilisez l'URL générée comme point de terminaison push :

```
gcloud pubsub subscriptions create gog-gmail-watch-push \
  --topic gog-gmail-watch \
  --push-endpoint "https://<public-url>/gmail-pubsub?token=<shared>"
```

Production : utilisez un point de terminaison HTTPS stable et configurez Pub/Sub OIDC JWT, puis exécutez :

```bash
gog gmail watch serve --verify-oidc --oidc-email <svc@...>
```

## Test

Envoyez un message à la boîte de réception surveillée :

```
gog gmail send \
  --account openclaw@gmail.com \
  --to openclaw@gmail.com \
  --subject "test surveillance" \
  --body "ping"
```

Vérifiez l'état de la surveillance et l'historique :

```bash
gog gmail watch status --account openclaw@gmail.com
gog gmail history --account openclaw@gmail.com --since <historyId>
```

## Dépannage

-   `Invalid topicName` : discordance de projet (le sujet n'est pas dans le projet du client OAuth).
-   `User not authorized` : manque `roles/pubsub.publisher` sur le sujet.
-   Messages vides : Gmail push ne fournit que `historyId` ; récupérez via `gog gmail history`.

## Nettoyage

```bash
gog gmail watch stop --account openclaw@gmail.com
gcloud pubsub subscriptions delete gog-gmail-watch-push
gcloud pubsub topics delete gog-gmail-watch
```

[Webhooks](./webhook.md)[Sondages](./poll.md)