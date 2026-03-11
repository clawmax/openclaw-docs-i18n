

  Protocoles et APIs

  
# API d'invocation d'outils

La passerelle OpenClaw expose un simple point de terminaison HTTP pour invoquer un outil directement. Il est toujours activé, mais protégé par l'authentification de la passerelle et la politique des outils.

-   `POST /tools/invoke`
-   Même port que la passerelle (WS + HTTP multiplexé) : `http://<gateway-host>:/tools/invoke`

La taille maximale de charge utile par défaut est de 2 Mo.

## Authentification

Utilise la configuration d'authentification de la passerelle. Envoyez un jeton porteur :

-   `Authorization: Bearer `

Notes :

-   Lorsque `gateway.auth.mode="token"`, utilisez `gateway.auth.token` (ou `OPENCLAW_GATEWAY_TOKEN`).
-   Lorsque `gateway.auth.mode="password"`, utilisez `gateway.auth.password` (ou `OPENCLAW_GATEWAY_PASSWORD`).
-   Si `gateway.auth.rateLimit` est configuré et que trop d'échecs d'authentification se produisent, le point de terminaison renvoie `429` avec `Retry-After`.

## Corps de la requête

```json
{
  "tool": "sessions_list",
  "action": "json",
  "args": {},
  "sessionKey": "main",
  "dryRun": false
}
```

Champs :

-   `tool` (chaîne, requis) : nom de l'outil à invoquer.
-   `action` (chaîne, optionnel) : mappé dans args si le schéma de l'outil prend en charge `action` et que la charge utile args l'a omis.
-   `args` (objet, optionnel) : arguments spécifiques à l'outil.
-   `sessionKey` (chaîne, optionnel) : clé de session cible. Si omise ou `"main"`, la passerelle utilise la clé de session principale configurée (respecte `session.mainKey` et l'agent par défaut, ou `global` dans la portée globale).
-   `dryRun` (booléen, optionnel) : réservé à un usage futur ; actuellement ignoré.

## Comportement de politique et routage

La disponibilité des outils est filtrée via la même chaîne de politique utilisée par les agents de la passerelle :

-   `tools.profile` / `tools.byProvider.profile`
-   `tools.allow` / `tools.byProvider.allow`
-   `agents..tools.allow` / `agents..tools.byProvider.allow`
-   politiques de groupe (si la clé de session correspond à un groupe ou canal)
-   politique de sous-agent (lors de l'invocation avec une clé de session de sous-agent)

Si un outil n'est pas autorisé par la politique, le point de terminaison renvoie **404**. La passerelle HTTP applique également une liste de refus stricte par défaut (même si la politique de session autorise l'outil) :

-   `sessions_spawn`
-   `sessions_send`
-   `gateway`
-   `whatsapp_login`

Vous pouvez personnaliser cette liste de refus via `gateway.tools` :

```json
{
  gateway: {
    tools: {
      // Outils supplémentaires à bloquer via HTTP /tools/invoke
      deny: ["browser"],
      // Retirer des outils de la liste de refus par défaut
      allow: ["gateway"],
    },
  },
}
```

Pour aider les politiques de groupe à résoudre le contexte, vous pouvez optionnellement définir :

-   `x-openclaw-message-channel: ` (exemple : `slack`, `telegram`)
-   `x-openclaw-account-id: ` (lorsque plusieurs comptes existent)

## Réponses

-   `200` → `{ ok: true, result }`
-   `400` → `{ ok: false, error: { type, message } }` (requête invalide ou erreur d'entrée d'outil)
-   `401` → non autorisé
-   `429` → authentification limitée par taux (`Retry-After` défini)
-   `404` → outil non disponible (non trouvé ou non autorisé)
-   `405` → méthode non autorisée
-   `500` → `{ ok: false, error: { type, message } }` (erreur d'exécution d'outil inattendue ; message assaini)

## Exemple

```bash
curl -sS http://127.0.0.1:18789/tools/invoke \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
    "tool": "sessions_list",
    "action": "json",
    "args": {}
  }'
```

[API OpenResponses](./openresponses-http-api.md)[Backends CLI](./cli-backends.md)

---