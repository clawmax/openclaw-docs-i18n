

  Référence technique

  
# Utilisation et coûts de l'API

Ce document liste les **fonctionnalités qui peuvent invoquer des clés API** et où leurs coûts apparaissent. Il se concentre sur les fonctionnalités d'OpenClaw qui peuvent générer une utilisation chez le fournisseur ou des appels API payants.

## Où les coûts apparaissent (chat + CLI)

**Instantané des coûts par session**

-   `/status` affiche le modèle de la session en cours, l'utilisation du contexte et les tokens de la dernière réponse.
-   Si le modèle utilise une **authentification par clé API**, `/status` affiche également le **coût estimé** pour la dernière réponse.

**Pied de page de coût par message**

-   `/usage full` ajoute un pied de page d'utilisation à chaque réponse, incluant le **coût estimé** (clé API uniquement).
-   `/usage tokens` n'affiche que les tokens ; les flux OAuth masquent le coût en dollars.

**Fenêtres d'utilisation CLI (quotas fournisseur)**

-   `openclaw status --usage` et `openclaw channels list` affichent les **fenêtres d'utilisation** du fournisseur (instantanés de quota, pas les coûts par message).

Voir [Utilisation et coûts des tokens](./token-use.md) pour plus de détails et des exemples.

## Comment les clés sont découvertes

OpenClaw peut récupérer les identifiants depuis :

-   **Profils d'authentification** (par agent, stockés dans `auth-profiles.json`).
-   **Variables d'environnement** (par ex. `OPENAI_API_KEY`, `BRAVE_API_KEY`, `FIRECRAWL_API_KEY`).
-   **Configuration** (`models.providers.*.apiKey`, `tools.web.search.*`, `tools.web.fetch.firecrawl.*`, `memorySearch.*`, `talk.apiKey`).
-   **Compétences** (`skills.entries..apiKey`) qui peuvent exporter des clés vers l'environnement du processus de la compétence.

## Fonctionnalités qui peuvent consommer des clés

### 1) Réponses du modèle principal (chat + outils)

Chaque réponse ou appel d'outil utilise le **fournisseur de modèle actuel** (OpenAI, Anthropic, etc.). C'est la source principale d'utilisation et de coût. Voir [Modèles](../providers/models.md) pour la configuration des prix et [Utilisation et coûts des tokens](./token-use.md) pour l'affichage.

### 2) Compréhension des médias (audio/image/vidéo)

Les médias entrants peuvent être résumés/transcrits avant l'exécution de la réponse. Cela utilise les API des modèles/fournisseurs.

-   Audio : OpenAI / Groq / Deepgram (maintenant **activé automatiquement** lorsque les clés existent).
-   Image : OpenAI / Anthropic / Google.
-   Vidéo : Google.

Voir [Compréhension des médias](../nodes/media-understanding.md).

### 3) Intégrations mémoire + recherche sémantique

La recherche sémantique en mémoire utilise des **API d'intégration** lorsqu'elle est configurée pour des fournisseurs distants :

-   `memorySearch.provider = "openai"` → intégrations OpenAI
-   `memorySearch.provider = "gemini"` → intégrations Gemini
-   `memorySearch.provider = "voyage"` → intégrations Voyage
-   `memorySearch.provider = "mistral"` → intégrations Mistral
-   `memorySearch.provider = "ollama"` → intégrations Ollama (local/auto-hébergé ; généralement pas de facturation d'API hébergée)
-   Retour facultatif vers un fournisseur distant si les intégrations locales échouent

Vous pouvez la garder locale avec `memorySearch.provider = "local"` (pas d'utilisation d'API). Voir [Mémoire](../concepts/memory.md).

### 4) Outil de recherche web

`web_search` utilise des clés API et peut entraîner des frais d'utilisation selon votre fournisseur :

-   **API Perplexity Search** : `PERPLEXITY_API_KEY`
-   **API Brave Search** : `BRAVE_API_KEY` ou `tools.web.search.apiKey`
-   **Gemini (Google Search)** : `GEMINI_API_KEY`
-   **Grok (xAI)** : `XAI_API_KEY`
-   **Kimi (Moonshot)** : `KIMI_API_KEY` ou `MOONSHOT_API_KEY`

Voir [Outils web](../tools/web.md).

### 5) Outil de récupération web (Firecrawl)

`web_fetch` peut appeler **Firecrawl** lorsqu'une clé API est présente :

-   `FIRECRAWL_API_KEY` ou `tools.web.fetch.firecrawl.apiKey`

Si Firecrawl n'est pas configuré, l'outil revient à une récupération directe + readability (pas d'API payante). Voir [Outils web](../tools/web.md).

### 6) Instantanés d'utilisation fournisseur (status/health)

Certaines commandes d'état appellent les **points de terminaison d'utilisation du fournisseur** pour afficher les fenêtres de quota ou l'état de l'authentification. Ce sont généralement des appels à faible volume mais qui atteignent tout de même les API des fournisseurs :

-   `openclaw status --usage`
-   `openclaw models status --json`

Voir [CLI Modèles](../cli/models.md).

### 7) Résumé de la sauvegarde de compactage

La sauvegarde de compactage peut résumer l'historique de la session en utilisant le **modèle actuel**, ce qui invoque les API du fournisseur lorsqu'elle s'exécute. Voir [Gestion de session + compactage](./session-management-compaction.md).

### 8) Analyse / sondage de modèle

`openclaw models scan` peut sonder les modèles OpenRouter et utilise `OPENROUTER_API_KEY` lorsque le sondage est activé. Voir [CLI Modèles](../cli/models.md).

### 9) Talk (parole)

Le mode Talk peut invoquer **ElevenLabs** lorsqu'il est configuré :

-   `ELEVENLABS_API_KEY` ou `talk.apiKey`

Voir [Mode Talk](../nodes/talk.md).

### 10) Compétences (API tierces)

Les compétences peuvent stocker `apiKey` dans `skills.entries..apiKey`. Si une compétence utilise cette clé pour des API externes, elle peut entraîner des coûts selon le fournisseur de la compétence. Voir [Compétences](../tools/skills.md).

[Mise en cache des invites](./prompt-caching.md)[Hygiène des transcriptions](./transcript-hygiene.md)

---