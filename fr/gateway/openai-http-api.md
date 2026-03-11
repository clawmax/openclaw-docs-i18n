

  Protocoles et APIs

  
# Chat Completions OpenAI

La passerelle d'OpenClaw peut servir un petit point de terminaison de Chat Completions compatible OpenAI. Ce point de terminaison est **désactivé par défaut**. Activez-le d'abord dans la configuration.

-   `POST /v1/chat/completions`
-   Même port que la passerelle (WS + HTTP multiplexé) : `http://<gateway-host>:/v1/chat/completions`

En interne, les requêtes sont exécutées comme une exécution normale d'agent de la passerelle (même chemin de code que `openclaw agent`), donc le routage/les permissions/la configuration correspondent à votre passerelle.

## Authentification

Utilise la configuration d'authentification de la passerelle. Envoyez un jeton porteur :

-   `Authorization: Bearer `

Notes :

-   Lorsque `gateway.auth.mode="token"`, utilisez `gateway.auth.token` (ou `OPENCLAW_GATEWAY_TOKEN`).
-   Lorsque `gateway.auth.mode="password"`, utilisez `gateway.auth.password` (ou `OPENCLAW_GATEWAY_PASSWORD`).
-   Si `gateway.auth.rateLimit` est configuré et que trop d'échecs d'authentification se produisent, le point de terminaison renvoie `429` avec `Retry-After`.

## Périmètre de sécurité (important)

Traitez ce point de terminaison comme une surface d'accès **opérateur complet** pour l'instance de la passerelle.

-   L'authentification HTTP par jeton porteur ici n'est pas un modèle de portée étroite par utilisateur.
-   Un jeton/mot de passe valide de passerelle pour ce point de terminaison doit être traité comme un identifiant propriétaire/opérateur.
-   Les requêtes passent par le même chemin d'agent de plan de contrôle que les actions d'opérateur de confiance.
-   Il n'y a pas de limite d'outil non-propriétaire/par utilisateur distincte sur ce point de terminaison ; une fois qu'un appelant passe l'authentification de la passerelle ici, OpenClaw traite cet appelant comme un opérateur de confiance pour cette passerelle.
-   Si la politique de l'agent cible autorise des outils sensibles, ce point de terminaison peut les utiliser.
-   Gardez ce point de terminaison uniquement sur loopback/tailnet/ingress privé ; ne l'exposez pas directement à l'internet public.

Voir [Sécurité](./security.md) et [Accès distant](./remote.md).

## Choisir un agent

Aucun en-tête personnalisé requis : encodez l'identifiant de l'agent dans le champ `model` d'OpenAI :

-   `model: "openclaw:"` (exemple : `"openclaw:main"`, `"openclaw:beta"`)
-   `model: "agent:"` (alias)

Ou ciblez un agent OpenClaw spécifique par en-tête :

-   `x-openclaw-agent-id: ` (par défaut : `main`)

Avancé :

-   `x-openclaw-session-key: ` pour contrôler entièrement le routage de session.

## Activation du point de terminaison

Définissez `gateway.http.endpoints.chatCompletions.enabled` sur `true` :

```json
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: true },
      },
    },
  },
}
```

## Désactivation du point de terminaison

Définissez `gateway.http.endpoints.chatCompletions.enabled` sur `false` :

```json
{
  gateway: {
    http: {
      endpoints: {
        chatCompletions: { enabled: false },
      },
    },
  },
}
```

## Comportement de session

Par défaut, le point de terminaison est **sans état par requête** (une nouvelle clé de session est générée à chaque appel). Si la requête inclut une chaîne `user` OpenAI, la passerelle dérive une clé de session stable à partir de celle-ci, de sorte que les appels répétés peuvent partager une session d'agent.

## Streaming (SSE)

Définissez `stream: true` pour recevoir des événements envoyés par le serveur (SSE) :

-   `Content-Type: text/event-stream`
-   Chaque ligne d'événement est `data: `
-   Le flux se termine par `data: [DONE]`

## Exemples

Non-streaming :

```bash
curl -sS http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "messages": [{"role":"user","content":"hi"}]
  }'
```

Streaming :

```bash
curl -N http://127.0.0.1:18789/v1/chat/completions \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "stream": true,
    "messages": [{"role":"user","content":"hi"}]
  }'
```

[Protocole Bridge](./bridge-protocol.md)[API OpenResponses](./openresponses-http-api.md)