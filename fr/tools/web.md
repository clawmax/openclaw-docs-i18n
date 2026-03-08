

  Outils intégrés

  
# Outils Web

OpenClaw inclut deux outils web légers :

-   `web_search` — Rechercher sur le web en utilisant l'API Perplexity Search, l'API Brave Search, Gemini avec l'ancrage Google Search, Grok ou Kimi.
-   `web_fetch` — Récupération HTTP + extraction lisible (HTML → markdown/texte).

Ce ne sont **pas** des outils d'automatisation de navigateur. Pour les sites lourds en JavaScript ou nécessitant une connexion, utilisez l'[Outil Navigateur](./browser.md).

## Fonctionnement

-   `web_search` appelle votre fournisseur configuré et retourne les résultats.
-   Les résultats sont mis en cache par requête pendant 15 minutes (configurable).
-   `web_fetch` effectue une requête HTTP GET simple et extrait le contenu lisible (HTML → markdown/texte). Il **n'exécute pas** JavaScript.
-   `web_fetch` est activé par défaut (sauf s'il est explicitement désactivé).

Voir [Configuration Perplexity Search](../perplexity.md) et [Configuration Brave Search](../brave-search.md) pour les détails spécifiques à chaque fournisseur.

## Choisir un fournisseur de recherche

| Fournisseur | Avantages | Inconvénients | Clé API |
| --- | --- | --- | --- |
| **API Perplexity Search** | Rapide, résultats structurés ; filtres de domaine, langue, région et fraîcheur ; extraction de contenu | — | `PERPLEXITY_API_KEY` |
| **API Brave Search** | Rapide, résultats structurés | Moins d'options de filtrage ; conditions d'utilisation IA applicables | `BRAVE_API_KEY` |
| **Gemini** | Ancrage Google Search, synthétisé par IA | Nécessite une clé API Gemini | `GEMINI_API_KEY` |
| **Grok** | Réponses ancrées au web xAI | Nécessite une clé API xAI | `XAI_API_KEY` |
| **Kimi** | Capacité de recherche web Moonshot | Nécessite une clé API Moonshot | `KIMI_API_KEY` / `MOONSHOT_API_KEY` |

### Détection automatique

Si aucun `provider` n'est explicitement défini, OpenClaw détecte automatiquement le fournisseur à utiliser en fonction des clés API disponibles, en vérifiant dans cet ordre :

1.  **Brave** — Variable d'environnement `BRAVE_API_KEY` ou config `tools.web.search.apiKey`
2.  **Gemini** — Variable d'environnement `GEMINI_API_KEY` ou config `tools.web.search.gemini.apiKey`
3.  **Kimi** — Variable d'environnement `KIMI_API_KEY` / `MOONSHOT_API_KEY` ou config `tools.web.search.kimi.apiKey`
4.  **Perplexity** — Variable d'environnement `PERPLEXITY_API_KEY` ou config `tools.web.search.perplexity.apiKey`
5.  **Grok** — Variable d'environnement `XAI_API_KEY` ou config `tools.web.search.grok.apiKey`

Si aucune clé n'est trouvée, il revient à Brave (vous obtiendrez une erreur de clé manquante vous invitant à en configurer une).

## Configurer la recherche web

Utilisez `openclaw configure --section web` pour configurer votre clé API et choisir un fournisseur.

### Perplexity Search

1.  Créez un compte Perplexity sur [perplexity.ai/settings/api](https://www.perplexity.ai/settings/api)
2.  Générez une clé API dans le tableau de bord
3.  Exécutez `openclaw configure --section web` pour stocker la clé dans la configuration, ou définissez `PERPLEXITY_API_KEY` dans votre environnement.

Voir [Documentation de l'API Perplexity Search](https://docs.perplexity.ai/guides/search-quickstart) pour plus de détails.

### Brave Search

1.  Créez un compte API Brave Search sur [brave.com/search/api](https://brave.com/search/api/)
2.  Dans le tableau de bord, choisissez le plan **Data for Search** (pas "Data for AI") et générez une clé API.
3.  Exécutez `openclaw configure --section web` pour stocker la clé dans la configuration (recommandé), ou définissez `BRAVE_API_KEY` dans votre environnement.

Brave propose des plans payants ; consultez le portail API Brave pour connaître les limites et tarifs actuels.

### Où stocker la clé

**Via la configuration (recommandé) :** exécutez `openclaw configure --section web`. Elle stocke la clé sous `tools.web.search.perplexity.apiKey` ou `tools.web.search.apiKey`. **Via l'environnement :** définissez `PERPLEXITY_API_KEY` ou `BRAVE_API_KEY` dans l'environnement du processus Gateway. Pour une installation gateway, placez-la dans `~/.openclaw/.env` (ou votre environnement de service). Voir [Variables d'environnement](../help/faq.md#how-does-openclaw-load-environment-variables).

### Exemples de configuration

**Perplexity Search :**

```json
{
  tools: {
    web: {
      search: {
        enabled: true,
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...", // optionnel si PERPLEXITY_API_KEY est défini
        },
      },
    },
  },
}
```

**Brave Search :**

```json
{
  tools: {
    web: {
      search: {
        enabled: true,
        provider: "brave",
        apiKey: "YOUR_BRAVE_API_KEY", // optionnel si BRAVE_API_KEY est défini // pragma: allowlist secret
      },
    },
  },
}
```

## Utiliser Gemini (ancrage Google Search)

Les modèles Gemini prennent en charge l'[ancrage Google Search](https://ai.google.dev/gemini-api/docs/grounding) intégré, qui retourne des réponses synthétisées par IA étayées par les résultats de recherche Google en direct avec des citations.

### Obtenir une clé API Gemini

1.  Allez sur [Google AI Studio](https://aistudio.google.com/apikey)
2.  Créez une clé API
3.  Définissez `GEMINI_API_KEY` dans l'environnement Gateway, ou configurez `tools.web.search.gemini.apiKey`

### Configurer la recherche Gemini

```json
{
  tools: {
    web: {
      search: {
        provider: "gemini",
        gemini: {
          // Clé API (optionnelle si GEMINI_API_KEY est définie)
          apiKey: "AIza...",
          // Modèle (par défaut "gemini-2.5-flash")
          model: "gemini-2.5-flash",
        },
      },
    },
  },
}
```

**Alternative par environnement :** définissez `GEMINI_API_KEY` dans l'environnement Gateway. Pour une installation gateway, placez-la dans `~/.openclaw/.env`.

### Notes

-   Les URL de citation provenant de l'ancrage Gemini sont automatiquement résolues des URL de redirection de Google vers des URL directes.
-   La résolution des redirections utilise le chemin de garde SSRF (vérifications HEAD + redirections + validation http/https) avant de retourner l'URL de citation finale.
-   La résolution des redirections utilise les paramètres SSRF stricts par défaut, donc les redirections vers des cibles privées/internes sont bloquées.
-   Le modèle par défaut (`gemini-2.5-flash`) est rapide et économique. Tout modèle Gemini prenant en charge l'ancrage peut être utilisé.

## web\_search

Rechercher sur le web en utilisant votre fournisseur configuré.

### Prérequis

-   `tools.web.search.enabled` ne doit pas être `false` (par défaut : activé)
-   Clé API pour votre fournisseur choisi :
    -   **Brave** : `BRAVE_API_KEY` ou `tools.web.search.apiKey`
    -   **Perplexity** : `PERPLEXITY_API_KEY` ou `tools.web.search.perplexity.apiKey`
    -   **Gemini** : `GEMINI_API_KEY` ou `tools.web.search.gemini.apiKey`
    -   **Grok** : `XAI_API_KEY` ou `tools.web.search.grok.apiKey`
    -   **Kimi** : `KIMI_API_KEY`, `MOONSHOT_API_KEY`, ou `tools.web.search.kimi.apiKey`

### Configuration

```json
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "BRAVE_API_KEY_HERE", // optionnel si BRAVE_API_KEY est défini
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
      },
    },
  },
}
```

### Paramètres de l'outil

Tous les paramètres fonctionnent à la fois pour Brave et Perplexity sauf indication contraire.

| Paramètre | Description |
| --- | --- |
| `query` | Requête de recherche (requise) |
| `count` | Nombre de résultats à retourner (1-10, défaut : 5) |
| `country` | Code pays ISO à 2 lettres (ex. "US", "DE") |
| `language` | Code langue ISO 639-1 (ex. "en", "de") |
| `freshness` | Filtre temporel : `day`, `week`, `month`, ou `year` |
| `date_after` | Résultats après cette date (AAAA-MM-JJ) |
| `date_before` | Résultats avant cette date (AAAA-MM-JJ) |
| `ui_lang` | Code langue de l'interface (Brave uniquement) |
| `domain_filter` | Tableau de liste d'autorisation/interdiction de domaines (Perplexity uniquement) |
| `max_tokens` | Budget total de contenu, défaut 25000 (Perplexity uniquement) |
| `max_tokens_per_page` | Limite de tokens par page, défaut 2048 (Perplexity uniquement) |

**Exemples :**

```
// Recherche spécifique à l'Allemagne
await web_search({
  query: "TV online schauen",
  country: "DE",
  language: "de",
});

// Résultats récents (semaine passée)
await web_search({
  query: "TMBG interview",
  freshness: "week",
});

// Recherche par plage de dates
await web_search({
  query: "AI developments",
  date_after: "2024-01-01",
  date_before: "2024-06-30",
});

// Filtrage par domaine (Perplexity uniquement)
await web_search({
  query: "climate research",
  domain_filter: ["nature.com", "science.org", ".edu"],
});

// Exclure des domaines (Perplexity uniquement)
await web_search({
  query: "product reviews",
  domain_filter: ["-reddit.com", "-pinterest.com"],
});

// Plus d'extraction de contenu (Perplexity uniquement)
await web_search({
  query: "detailed AI research",
  max_tokens: 50000,
  max_tokens_per_page: 4096,
});
```

## web\_fetch

Récupérer une URL et extraire le contenu lisible.

### Prérequis de web\_fetch

-   `tools.web.fetch.enabled` ne doit pas être `false` (par défaut : activé)
-   Solution de repli Firecrawl optionnelle : définissez `tools.web.fetch.firecrawl.apiKey` ou `FIRECRAWL_API_KEY`.

### Configuration de web\_fetch

```json
{
  tools: {
    web: {
      fetch: {
        enabled: true,
        maxChars: 50000,
        maxCharsCap: 50000,
        maxResponseBytes: 2000000,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
        maxRedirects: 3,
        userAgent: "Mozilla/5.0 (Macintosh; Intel Mac OS X 14_7_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36",
        readability: true,
        firecrawl: {
          enabled: true,
          apiKey: "FIRECRAWL_API_KEY_HERE", // optionnel si FIRECRAWL_API_KEY est défini
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 86400000, // ms (1 jour)
          timeoutSeconds: 60,
        },
      },
    },
  },
}
```

### Paramètres de l'outil web\_fetch

-   `url` (requise, http/https uniquement)
-   `extractMode` (`markdown` | `text`)
-   `maxChars` (tronquer les pages longues)

Notes :

-   `web_fetch` utilise d'abord Readability (extraction du contenu principal), puis Firecrawl (si configuré). Si les deux échouent, l'outil retourne une erreur.
-   Les requêtes Firecrawl utilisent le mode de contournement de bot et mettent les résultats en cache par défaut.
-   `web_fetch` envoie un User-Agent de type Chrome et `Accept-Language` par défaut ; remplacez `userAgent` si nécessaire.
-   `web_fetch` bloque les noms d'hôte privés/internes et revérifie les redirections (limitées par `maxRedirects`).
-   `maxChars` est limité à `tools.web.fetch.maxCharsCap`.
-   `web_fetch` limite la taille du corps de réponse téléchargé à `tools.web.fetch.maxResponseBytes` avant l'analyse ; les réponses trop volumineuses sont tronquées et incluent un avertissement.
-   `web_fetch` est une extraction au mieux ; certains sites nécessiteront l'outil navigateur.
-   Voir [Firecrawl](./firecrawl.md) pour la configuration de la clé et les détails du service.
-   Les réponses sont mises en cache (15 minutes par défaut) pour réduire les récupérations répétées.
-   Si vous utilisez des profils/listes d'autorisation d'outils, ajoutez `web_search`/`web_fetch` ou `group:web`.
-   Si la clé API est manquante, `web_search` retourne un court indice de configuration avec un lien vers la documentation.

[Niveaux de réflexion](./thinking.md)[Navigateur (géré par OpenClaw)](./browser.md)