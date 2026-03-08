title: "API HTTP OpenResponses pour la configuration et l'utilisation de la passerelle OpenClaw"
description: "Apprenez à activer et utiliser le point de terminaison POST /v1/responses compatible OpenResponses dans la passerelle OpenClaw. Configurez l'authentification, la sécurité, les agents, les outils et la gestion des fichiers."
keywords: ["api openresponses", "passerelle openclaw", "api http", "intégration d'agent", "authentification api", "outils client", "téléchargement de fichier", "streaming sse"]
---

  Protocoles et APIs

  
# API OpenResponses

La passerelle OpenClaw peut servir un point de terminaison `POST /v1/responses` compatible OpenResponses. Ce point de terminaison est **désactivé par défaut**. Activez-le d'abord dans la configuration.

-   `POST /v1/responses`
-   Même port que la passerelle (WS + HTTP multiplexé) : `http://<gateway-host>:/v1/responses`

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
-   Un jeton/mot de passe valide de la passerelle pour ce point de terminaison doit être traité comme un identifiant propriétaire/opérateur.
-   Les requêtes passent par le même chemin d'agent de plan de contrôle que les actions d'opérateur de confiance.
-   Il n'y a pas de périmètre d'outil non-propriétaire/par utilisateur séparé sur ce point de terminaison ; une fois qu'un appelant passe l'authentification de la passerelle ici, OpenClaw traite cet appelant comme un opérateur de confiance pour cette passerelle.
-   Si la politique de l'agent cible autorise des outils sensibles, ce point de terminaison peut les utiliser.
-   Gardez ce point de terminaison uniquement sur loopback/tailnet/ingress privé ; ne l'exposez pas directement à l'internet public.

Voir [Sécurité](./security.md) et [Accès distant](./remote.md).

## Choisir un agent

Aucun en-tête personnalisé requis : encodez l'identifiant de l'agent dans le champ `model` d'OpenResponses :

-   `model: "openclaw:"` (exemple : `"openclaw:main"`, `"openclaw:beta"`)
-   `model: "agent:"` (alias)

Ou ciblez un agent OpenClaw spécifique par en-tête :

-   `x-openclaw-agent-id: ` (par défaut : `main`)

Avancé :

-   `x-openclaw-session-key: ` pour contrôler entièrement le routage de session.

## Activation du point de terminaison

Définissez `gateway.http.endpoints.responses.enabled` sur `true` :

```json
{
  gateway: {
    http: {
      endpoints: {
        responses: { enabled: true },
      },
    },
  },
}
```

## Désactivation du point de terminaison

Définissez `gateway.http.endpoints.responses.enabled` sur `false` :

```json
{
  gateway: {
    http: {
      endpoints: {
        responses: { enabled: false },
      },
    },
  },
}
```

## Comportement de session

Par défaut, le point de terminaison est **sans état par requête** (une nouvelle clé de session est générée à chaque appel). Si la requête inclut une chaîne `user` OpenResponses, la passerelle dérive une clé de session stable de celle-ci, de sorte que les appels répétés peuvent partager une session d'agent.

## Format de requête (supporté)

La requête suit l'API OpenResponses avec une entrée basée sur des éléments. Support actuel :

-   `input` : chaîne ou tableau d'objets d'éléments.
-   `instructions` : fusionnées dans l'invite système.
-   `tools` : définitions d'outils client (outils de fonction).
-   `tool_choice` : filtrer ou exiger des outils client.
-   `stream` : active le streaming SSE.
-   `max_output_tokens` : limite de sortie au mieux (dépend du fournisseur).
-   `user` : routage de session stable.

Acceptés mais **actuellement ignorés** :

-   `max_tool_calls`
-   `reasoning`
-   `metadata`
-   `store`
-   `previous_response_id`
-   `truncation`

## Éléments (input)

### message

Rôles : `system`, `developer`, `user`, `assistant`.

-   `system` et `developer` sont ajoutés à l'invite système.
-   L'élément `user` ou `function_call_output` le plus récent devient le « message actuel ».
-   Les messages user/assistant précédents sont inclus comme historique pour le contexte.

### function\_call\_output (outils tour par tour)

Renvoie les résultats des outils au modèle :

```json
{
  "type": "function_call_output",
  "call_id": "call_123",
  "output": "{\"temperature\": \"72F\"}"
}
```

### reasoning et item\_reference

Acceptés pour la compatibilité du schéma mais ignorés lors de la construction de l'invite.

## Outils (outils de fonction côté client)

Fournissez des outils avec `tools: [{ type: "function", function: { name, description?, parameters? } }]`. Si l'agent décide d'appeler un outil, la réponse renvoie un élément de sortie `function_call`. Vous envoyez ensuite une requête de suivi avec `function_call_output` pour continuer le tour.

## Images (input\_image)

Prend en charge les sources base64 ou URL :

```json
{
  "type": "input_image",
  "source": { "type": "url", "url": "https://example.com/image.png" }
}
```

Types MIME autorisés (actuels) : `image/jpeg`, `image/png`, `image/gif`, `image/webp`, `image/heic`, `image/heif`. Taille max (actuelle) : 10 Mo.

## Fichiers (input\_file)

Prend en charge les sources base64 ou URL :

```json
{
  "type": "input_file",
  "source": {
    "type": "base64",
    "media_type": "text/plain",
    "data": "SGVsbG8gV29ybGQh",
    "filename": "hello.txt"
  }
}
```

Types MIME autorisés (actuels) : `text/plain`, `text/markdown`, `text/html`, `text/csv`, `application/json`, `application/pdf`. Taille max (actuelle) : 5 Mo. Comportement actuel :

-   Le contenu du fichier est décodé et ajouté à l'**invite système**, pas au message utilisateur, donc il reste éphémère (non persistant dans l'historique de session).
-   Les PDF sont analysés pour en extraire le texte. Si peu de texte est trouvé, les premières pages sont rastérisées en images et transmises au modèle.

L'analyse PDF utilise la version héritée `pdfjs-dist` compatible Node (pas de worker). La version moderne de PDF.js attend des workers navigateur/globals DOM, donc elle n'est pas utilisée dans la passerelle. Récupération par URL par défaut :

-   `files.allowUrl` : `true`
-   `images.allowUrl` : `true`
-   `maxUrlParts` : `8` (total des parties `input_file` + `input_image` basées sur l'URL par requête)
-   Les requêtes sont protégées (résolution DNS, blocage d'IP privée, limite de redirections, délais d'attente).
-   Des listes blanches d'hôtes facultatives sont prises en charge par type d'entrée (`files.urlAllowlist`, `images.urlAllowlist`).
    -   Hôte exact : `"cdn.example.com"`
    -   Sous-domaines génériques : `"*.assets.example.com"` (ne correspond pas au domaine apex)

## Limites de fichiers + images (config)

Les valeurs par défaut peuvent être ajustées sous `gateway.http.endpoints.responses` :

```json
{
  gateway: {
    http: {
      endpoints: {
        responses: {
          enabled: true,
          maxBodyBytes: 20000000,
          maxUrlParts: 8,
          files: {
            allowUrl: true,
            urlAllowlist: ["cdn.example.com", "*.assets.example.com"],
            allowedMimes: [
              "text/plain",
              "text/markdown",
              "text/html",
              "text/csv",
              "application/json",
              "application/pdf",
            ],
            maxBytes: 5242880,
            maxChars: 200000,
            maxRedirects: 3,
            timeoutMs: 10000,
            pdf: {
              maxPages: 4,
              maxPixels: 4000000,
              minTextChars: 200,
            },
          },
          images: {
            allowUrl: true,
            urlAllowlist: ["images.example.com"],
            allowedMimes: [
              "image/jpeg",
              "image/png",
              "image/gif",
              "image/webp",
              "image/heic",
              "image/heif",
            ],
            maxBytes: 10485760,
            maxRedirects: 3,
            timeoutMs: 10000,
          },
        },
      },
    },
  },
}
```

Valeurs par défaut si omises :

-   `maxBodyBytes` : 20 Mo
-   `maxUrlParts` : 8
-   `files.maxBytes` : 5 Mo
-   `files.maxChars` : 200k
-   `files.maxRedirects` : 3
-   `files.timeoutMs` : 10s
-   `files.pdf.maxPages` : 4
-   `files.pdf.maxPixels` : 4 000 000
-   `files.pdf.minTextChars` : 200
-   `images.maxBytes` : 10 Mo
-   `images.maxRedirects` : 3
-   `images.timeoutMs` : 10s
-   Les sources `input_image` HEIC/HEIF sont acceptées et normalisées en JPEG avant livraison au fournisseur.

Note de sécurité :

-   Les listes blanches d'URL sont appliquées avant la récupération et sur les sauts de redirection.
-   L'ajout d'un nom d'hôte à la liste blanche ne contourne pas le blocage des IP privées/internes.
-   Pour les passerelles exposées à Internet, appliquez des contrôles de sortie réseau en plus des protections au niveau de l'application. Voir [Sécurité](./security.md).

## Streaming (SSE)

Définissez `stream: true` pour recevoir des événements envoyés par le serveur (SSE) :

-   `Content-Type: text/event-stream`
-   Chaque ligne d'événement est `event: ` et `data: `
-   Le flux se termine par `data: [DONE]`

Types d'événements actuellement émis :

-   `response.created`
-   `response.in_progress`
-   `response.output_item.added`
-   `response.content_part.added`
-   `response.output_text.delta`
-   `response.output_text.done`
-   `response.content_part.done`
-   `response.output_item.done`
-   `response.completed`
-   `response.failed` (en cas d'erreur)

## Utilisation

`usage` est renseigné lorsque le fournisseur sous-jacent rapporte des décomptes de jetons.

## Erreurs

Les erreurs utilisent un objet JSON comme :

```json
{ "error": { "message": "...", "type": "invalid_request_error" } }
```

Cas courants :

-   `401` authentification manquante/invalide
-   `400` corps de requête invalide
-   `405` mauvaise méthode

## Exemples

Non-streaming :

```bash
curl -sS http://127.0.0.1:18789/v1/responses \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "input": "hi"
  }'
```

Streaming :

```bash
curl -N http://127.0.0.1:18789/v1/responses \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'x-openclaw-agent-id: main' \
  -d '{
    "model": "openclaw",
    "stream": true,
    "input": "hi"
  }'
```

[Complétions de chat OpenAI](./openai-http-api.md)[API d'invocation d'outils](./tools-invoke-http-api.md)