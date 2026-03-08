

  Automatisation

  
# Webhooks

La passerelle peut exposer un petit point de terminaison HTTP webhook pour des déclencheurs externes.

## Activer

```json
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    // Optionnel : restreindre le routage explicite `agentId` à cette liste d'autorisation.
    // Omettre ou inclure "*" pour autoriser n'importe quel agent.
    // Définir [] pour refuser tout routage explicite `agentId`.
    allowedAgentIds: ["hooks", "main"],
  },
}
```

Notes :

-   `hooks.token` est requis lorsque `hooks.enabled=true`.
-   `hooks.path` est par défaut `/hooks`.

## Authentification

Chaque requête doit inclure le jeton du webhook. Préférez les en-têtes :

-   `Authorization: Bearer ` (recommandé)
-   `x-openclaw-token: `
-   Les jetons dans la chaîne de requête sont rejetés (`?token=...` retourne `400`).

## Points de terminaison

### POST /hooks/wake

Charge utile :

```json
{ "text": "System line", "mode": "now" }
```

-   `text` **requis** (string) : La description de l'événement (par exemple, "Nouvel e-mail reçu").
-   `mode` optionnel (`now` | `next-heartbeat`) : Déclenche un battement de cœur immédiat (par défaut `now`) ou attend la prochaine vérification périodique.

Effet :

-   Met en file d'attente un événement système pour la session **principale**
-   Si `mode=now`, déclenche un battement de cœur immédiat

### POST /hooks/agent

Charge utile :

```json
{
  "message": "Run this",
  "name": "Email",
  "agentId": "hooks",
  "sessionKey": "hook:email:msg-123",
  "wakeMode": "now",
  "deliver": true,
  "channel": "last",
  "to": "+15551234567",
  "model": "openai/gpt-5.2-mini",
  "thinking": "low",
  "timeoutSeconds": 120
}
```

-   `message` **requis** (string) : L'invite ou le message que l'agent doit traiter.
-   `name` optionnel (string) : Nom lisible par un humain pour le webhook (par exemple, "GitHub"), utilisé comme préfixe dans les résumés de session.
-   `agentId` optionnel (string) : Achemine ce webhook vers un agent spécifique. Les ID inconnus reviennent à l'agent par défaut. Lorsqu'il est défini, le webhook s'exécute en utilisant l'espace de travail et la configuration de l'agent résolu.
-   `sessionKey` optionnel (string) : La clé utilisée pour identifier la session de l'agent. Par défaut, ce champ est rejeté sauf si `hooks.allowRequestSessionKey=true`.
-   `wakeMode` optionnel (`now` | `next-heartbeat`) : Déclenche un battement de cœur immédiat (par défaut `now`) ou attend la prochaine vérification périodique.
-   `deliver` optionnel (boolean) : Si `true`, la réponse de l'agent sera envoyée au canal de messagerie. Par défaut `true`. Les réponses qui ne sont que des accusés de réception de battement de cœur sont automatiquement ignorées.
-   `channel` optionnel (string) : Le canal de messagerie pour la livraison. Un parmi : `last`, `whatsapp`, `telegram`, `discord`, `slack`, `mattermost` (plugin), `signal`, `imessage`, `msteams`. Par défaut `last`.
-   `to` optionnel (string) : L'identifiant du destinataire pour le canal (par exemple, numéro de téléphone pour WhatsApp/Signal, ID de chat pour Telegram, ID de canal pour Discord/Slack/Mattermost (plugin), ID de conversation pour MS Teams). Par défaut le dernier destinataire dans la session principale.
-   `model` optionnel (string) : Surcharge du modèle (par exemple, `anthropic/claude-3-5-sonnet` ou un alias). Doit être dans la liste des modèles autorisés si restreint.
-   `thinking` optionnel (string) : Surcharge du niveau de réflexion (par exemple, `low`, `medium`, `high`).
-   `timeoutSeconds` optionnel (number) : Durée maximale de l'exécution de l'agent en secondes.

Effet :

-   Exécute un tour d'agent **isolé** (sa propre clé de session)
-   Poste toujours un résumé dans la session **principale**
-   Si `wakeMode=now`, déclenche un battement de cœur immédiat

## Politique de clé de session (changement cassant)

Les surcharges `sessionKey` dans la charge utile `/hooks/agent` sont désactivées par défaut.

-   Recommandé : définir une `hooks.defaultSessionKey` fixe et garder les surcharges par requête désactivées.
-   Optionnel : autoriser les surcharges par requête uniquement lorsque nécessaire, et restreindre les préfixes.

Configuration recommandée :

```json
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
  },
}
```

Configuration de compatibilité (comportement hérité) :

```json
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    allowRequestSessionKey: true,
    allowedSessionKeyPrefixes: ["hook:"], // fortement recommandé
  },
}
```

### POST /hooks/&lt;name&gt; (mappé)

Les noms de webhooks personnalisés sont résolus via `hooks.mappings` (voir configuration). Un mappage peut transformer des charges utiles arbitraires en actions `wake` ou `agent`, avec des modèles ou des transformations de code optionnels. Options de mappage (résumé) :

-   `hooks.presets: ["gmail"]` active le mappage Gmail intégré.
-   `hooks.mappings` permet de définir `match`, `action` et des modèles dans la configuration.
-   `hooks.transformsDir` + `transform.module` charge un module JS/TS pour une logique personnalisée.
    -   `hooks.transformsDir` (si défini) doit rester dans la racine des transformations sous votre répertoire de configuration OpenClaw (typiquement `~/.openclaw/hooks/transforms`).
    -   `transform.module` doit se résoudre dans le répertoire effectif des transformations (les chemins de traversée/échappement sont rejetés).
-   Utilisez `match.source` pour garder un point de terminaison d'ingestion générique (routage piloté par la charge utile).
-   Les transformations TS nécessitent un chargeur TS (par exemple `bun` ou `tsx`) ou un `.js` précompilé au moment de l'exécution.
-   Définissez `deliver: true` + `channel`/`to` sur les mappages pour acheminer les réponses vers une surface de chat (`channel` par défaut `last` et revient à WhatsApp).
-   `agentId` achemine le webhook vers un agent spécifique ; les ID inconnus reviennent à l'agent par défaut.
-   `hooks.allowedAgentIds` restreint le routage explicite `agentId`. Omettez-le (ou incluez `*`) pour autoriser n'importe quel agent. Définissez `[]` pour refuser le routage explicite `agentId`.
-   `hooks.defaultSessionKey` définit la session par défaut pour les exécutions d'agent webhook lorsqu'aucune clé explicite n'est fournie.
-   `hooks.allowRequestSessionKey` contrôle si les charges utiles `/hooks/agent` peuvent définir `sessionKey` (par défaut : `false`).
-   `hooks.allowedSessionKeyPrefixes` restreint optionnellement les valeurs `sessionKey` explicites provenant des charges utiles de requête et des mappages.
-   `allowUnsafeExternalContent: true` désactive l'enveloppe de sécurité du contenu externe pour ce webhook (dangereux ; uniquement pour des sources internes de confiance).
-   `openclaw webhooks gmail setup` écrit la configuration `hooks.gmail` pour `openclaw webhooks gmail run`. Voir [Gmail Pub/Sub](./gmail-pubsub.md) pour le flux complet de surveillance Gmail.

## Réponses

-   `200` pour `/hooks/wake`
-   `200` pour `/hooks/agent` (exécution asynchrone acceptée)
-   `401` en cas d'échec d'authentification
-   `429` après des échecs d'authentification répétés du même client (vérifiez `Retry-After`)
-   `400` en cas de charge utile invalide
-   `413` en cas de charge utile trop volumineuse

## Exemples

```bash
curl -X POST http://127.0.0.1:18789/hooks/wake \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"text":"New email received","mode":"now"}'
```

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","wakeMode":"next-heartbeat"}'
```

### Utiliser un modèle différent

Ajoutez `model` à la charge utile de l'agent (ou au mappage) pour surcharger le modèle pour cette exécution :

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","model":"openai/gpt-5.2-mini"}'
```

Si vous appliquez `agents.defaults.models`, assurez-vous que le modèle de surcharge y est inclus.

```bash
curl -X POST http://127.0.0.1:18789/hooks/gmail \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"source":"gmail","messages":[{"from":"Ada","subject":"Hello","snippet":"Hi"}]}'
```

## Sécurité

-   Gardez les points de terminaison webhook derrière loopback, tailnet ou un proxy inverse de confiance.
-   Utilisez un jeton webhook dédié ; ne réutilisez pas les jetons d'authentification de la passerelle.
-   Les échecs d'authentification répétés sont limités en débit par adresse client pour ralentir les tentatives de force brute.
-   Si vous utilisez le routage multi-agent, définissez `hooks.allowedAgentIds` pour limiter la sélection explicite `agentId`.
-   Gardez `hooks.allowRequestSessionKey=false` sauf si vous avez besoin de sessions sélectionnées par l'appelant.
-   Si vous activez `sessionKey` par requête, restreignez `hooks.allowedSessionKeyPrefixes` (par exemple, `["hook:"]`).
-   Évitez d'inclure des charges utiles brutes sensibles dans les journaux des webhooks.
-   Les charges utiles des webhooks sont traitées comme non fiables et enveloppées avec des limites de sécurité par défaut. Si vous devez désactiver cela pour un webhook spécifique, définissez `allowUnsafeExternalContent: true` dans le mappage de ce webhook (dangereux).

[Dépannage de l'automatisation](./troubleshooting.md)[Gmail PubSub](./gmail-pubsub.md)