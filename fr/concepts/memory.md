

  Sessions et mémoire

  
# Mémoire

La mémoire d'OpenClaw est constituée de **fichiers Markdown simples dans l'espace de travail de l'agent**. Les fichiers sont la source de vérité ; le modèle ne « se souvient » que de ce qui est écrit sur le disque. Les outils de recherche en mémoire sont fournis par le plugin de mémoire actif (par défaut : `memory-core`). Désactivez les plugins de mémoire avec `plugins.slots.memory = "none"`.

## Fichiers de mémoire (Markdown)

La disposition par défaut de l'espace de travail utilise deux couches de mémoire :

-   `memory/AAAA-MM-JJ.md`
    -   Journal quotidien (en ajout uniquement).
    -   Lu aujourd'hui + hier au démarrage de la session.
-   `MEMORY.md` (optionnel)
    -   Mémoire à long terme organisée.
    -   **Chargé uniquement dans la session principale, privée** (jamais dans les contextes de groupe).

Ces fichiers se trouvent sous l'espace de travail (`agents.defaults.workspace`, par défaut `~/.openclaw/workspace`). Voir [Espace de travail de l'agent](./agent-workspace.md) pour la disposition complète.

## Outils de mémoire

OpenClaw expose deux outils pour l'agent concernant ces fichiers Markdown :

-   `memory_search` — rappel sémantique sur des extraits indexés.
-   `memory_get` — lecture ciblée d'un fichier Markdown spécifique / d'une plage de lignes.

`memory_get` **dégradé désormais gracieusement lorsqu'un fichier n'existe pas** (par exemple, le journal quotidien d'aujourd'hui avant la première écriture). Le gestionnaire intégré et le backend QMD retournent tous deux `{ text: "", path }` au lieu de lever une erreur `ENOENT`, permettant ainsi aux agents de gérer le cas « rien n'a encore été enregistré » et de poursuivre leur flux de travail sans encapsuler l'appel d'outil dans une logique try/catch.

## Quand écrire en mémoire

-   Les décisions, préférences et faits durables vont dans `MEMORY.md`.
-   Les notes quotidiennes et le contexte en cours vont dans `memory/AAAA-MM-JJ.md`.
-   Si quelqu'un dit « souviens-toi de ça », notez-le (ne le gardez pas en RAM).
-   Ce domaine évolue encore. Il est utile de rappeler au modèle de stocker des souvenirs ; il saura quoi faire.
-   Si vous voulez que quelque chose reste, **demandez au bot de l'écrire** dans la mémoire.

## Purge automatique de la mémoire (pré-compaction)

Lorsqu'une session est **proche de l'auto-compaction**, OpenClaw déclenche un **tour agentique silencieux** qui rappelle au modèle d'écrire la mémoire durable **avant** que le contexte ne soit compacté. Les prompts par défaut indiquent explicitement que le modèle *peut répondre*, mais généralement `NO_REPLY` est la réponse correcte pour que l'utilisateur ne voie jamais ce tour. Ceci est contrôlé par `agents.defaults.compaction.memoryFlush` :

```json
{
  agents: {
    defaults: {
      compaction: {
        reserveTokensFloor: 20000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 4000,
          systemPrompt: "Session nearing compaction. Store durable memories now.",
          prompt: "Write any lasting notes to memory/YYYY-MM-DD.md; reply with NO_REPLY if nothing to store.",
        },
      },
    },
  },
}
```

Détails :

-   **Seuil souple** : la purge se déclenche lorsque l'estimation de tokens de la session dépasse `contextWindow - reserveTokensFloor - softThresholdTokens`.
-   **Silencieux** par défaut : les prompts incluent `NO_REPLY` pour ne rien délivrer.
-   **Deux prompts** : un prompt utilisateur plus un prompt système ajoutent le rappel.
-   **Une purge par cycle de compaction** (suivi dans `sessions.json`).
-   **L'espace de travail doit être accessible en écriture** : si la session s'exécute en mode sandbox avec `workspaceAccess: "ro"` ou `"none"`, la purge est ignorée.

Pour le cycle de vie complet de la compaction, voir [Gestion de session + compaction](../reference/session-management-compaction.md).

## Recherche mémoire vectorielle

OpenClaw peut construire un petit index vectoriel sur `MEMORY.md` et `memory/*.md` afin que les requêtes sémantiques puissent trouver des notes connexes même lorsque la formulation diffère. Valeurs par défaut :

-   Activé par défaut.
-   Surveille les fichiers de mémoire pour les changements (avec anti-rebond).
-   Configurez la recherche en mémoire sous `agents.defaults.memorySearch` (et non au niveau supérieur `memorySearch`).
-   Utilise des embeddings distants par défaut. Si `memorySearch.provider` n'est pas défini, OpenClaw sélectionne automatiquement :
    1.  `local` si un `memorySearch.local.modelPath` est configuré et que le fichier existe.
    2.  `openai` si une clé OpenAI peut être résolue.
    3.  `gemini` si une clé Gemini peut être résolue.
    4.  `voyage` si une clé Voyage peut être résolue.
    5.  `mistral` si une clé Mistral peut être résolue.
    6.  Sinon, la recherche en mémoire reste désactivée jusqu'à configuration.
-   Le mode local utilise node-llama-cpp et peut nécessiter `pnpm approve-builds`.
-   Utilise sqlite-vec (lorsqu'il est disponible) pour accélérer la recherche vectorielle dans SQLite.
-   `memorySearch.provider = "ollama"` est également pris en charge pour les embeddings Ollama locaux/auto-hébergés (`/api/embeddings`), mais il n'est pas sélectionné automatiquement.

Les embeddings distants **nécessitent** une clé API pour le fournisseur d'embeddings. OpenClaw résout les clés à partir des profils d'authentification, `models.providers.*.apiKey`, ou des variables d'environnement. L'OAuth Codex ne couvre que le chat/complétions et ne **satisfait pas** les embeddings pour la recherche en mémoire. Pour Gemini, utilisez `GEMINI_API_KEY` ou `models.providers.google.apiKey`. Pour Voyage, utilisez `VOYAGE_API_KEY` ou `models.providers.voyage.apiKey`. Pour Mistral, utilisez `MISTRAL_API_KEY` ou `models.providers.mistral.apiKey`. Ollama ne nécessite généralement pas de vraie clé API (un espace réservé comme `OLLAMA_API_KEY=ollama-local` suffit si requis par la politique locale). Lors de l'utilisation d'un endpoint personnalisé compatible OpenAI, définissez `memorySearch.remote.apiKey` (et optionnellement `memorySearch.remote.headers`).

### Backend QMD (expérimental)

Définissez `memory.backend = "qmd"` pour remplacer l'indexeur SQLite intégré par [QMD](https://github.com/tobi/qmd) : un sidecar de recherche local-first qui combine BM25 + vecteurs + reranking. Markdown reste la source de vérité ; OpenClaw exécute QMD en sous-processus pour la récupération. Points clés : **Prérequis**

-   Désactivé par défaut. Optez-y par configuration (`memory.backend = "qmd"`).
-   Installez séparément la CLI QMD (`bun install -g https://github.com/tobi/qmd` ou téléchargez une release) et assurez-vous que le binaire `qmd` est dans le `PATH` de la passerelle.
-   QMD nécessite une build SQLite qui autorise les extensions (`brew install sqlite` sur macOS).
-   QMD s'exécute entièrement en local via Bun + `node-llama-cpp` et télécharge automatiquement les modèles GGUF depuis HuggingFace au premier usage (aucun démon Ollama séparé requis).
-   La passerelle exécute QMD dans un répertoire XDG autonome sous `~/.openclaw/agents//qmd/` en définissant `XDG_CONFIG_HOME` et `XDG_CACHE_HOME`.
-   Support OS : macOS et Linux fonctionnent directement une fois Bun + SQLite installés. Windows est mieux supporté via WSL2.

**Comment le sidecar s'exécute**

-   La passerelle écrit un répertoire QMD autonome sous `~/.openclaw/agents//qmd/` (config + cache + base de données sqlite).
-   Les collections sont créées via `qmd collection add` à partir de `memory.qmd.paths` (plus les fichiers de mémoire par défaut de l'espace de travail), puis `qmd update` + `qmd embed` s'exécutent au démarrage et à un intervalle configurable (`memory.qmd.update.interval`, par défaut 5 m).
-   La passerelle initialise désormais le gestionnaire QMD au démarrage, donc les minuteries de mise à jour périodique sont armées avant même le premier appel `memory_search`.
-   Le rafraîchissement au démarrage s'exécute désormais en arrière-plan par défaut pour ne pas bloquer le démarrage du chat ; définissez `memory.qmd.update.waitForBootSync = true` pour conserver le comportement bloquant précédent.
-   Les recherches s'exécutent via `memory.qmd.searchMode` (par défaut `qmd search --json` ; supporte aussi `vsearch` et `query`). Si le mode sélectionné rejette les drapeaux sur votre build QMD, OpenClaw réessaie avec `qmd query`. Si QMD échoue ou si le binaire est manquant, OpenClaw revient automatiquement au gestionnaire SQLite intégré pour que les outils de mémoire continuent de fonctionner.
-   OpenClaw n'expose pas aujourd'hui le réglage de la taille de lot d'embedding QMD ; le comportement par lot est contrôlé par QMD lui-même.
-   **La première recherche peut être lente** : QMD peut télécharger des modèles GGUF locaux (reranker/expansion de requête) lors du premier exécution de `qmd query`.
    -   OpenClaw définit automatiquement `XDG_CONFIG_HOME`/`XDG_CACHE_HOME` lorsqu'il exécute QMD.
    -   Si vous souhaitez pré-télécharger manuellement les modèles (et réchauffer le même index qu'OpenClaw utilise), exécutez une requête ponctuelle avec les répertoires XDG de l'agent. L'état QMD d'OpenClaw se trouve sous votre **répertoire d'état** (par défaut `~/.openclaw`). Vous pouvez pointer `qmd` vers exactement le même index en exportant les mêmes variables XDG qu'OpenClaw utilise :
        
        Copier
        
        ```bash
        # Choisissez le même répertoire d'état qu'OpenClaw utilise
        STATE_DIR="${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"
        
        export XDG_CONFIG_HOME="$STATE_DIR/agents/main/qmd/xdg-config"
        export XDG_CACHE_HOME="$STATE_DIR/agents/main/qmd/xdg-cache"
        
        # (Optionnel) force un rafraîchissement d'index + embeddings
        qmd update
        qmd embed
        
        # Réchauffez / déclenchez les téléchargements de modèles pour la première fois
        qmd query "test" -c memory-root --json >/dev/null 2>&1
        ```
        

**Surface de configuration (`memory.qmd.*`)**

-   `command` (par défaut `qmd`) : remplace le chemin de l'exécutable.
-   `searchMode` (par défaut `search`) : choisit quelle commande QMD supporte `memory_search` (`search`, `vsearch`, `query`).
-   `includeDefaultMemory` (par défaut `true`) : indexe automatiquement `MEMORY.md` + `memory/**/*.md`.
-   `paths[]` : ajoute des répertoires/fichiers supplémentaires (`path`, `pattern` optionnel, `name` stable optionnel).
-   `sessions` : opte pour l'indexation JSONL des sessions (`enabled`, `retentionDays`, `exportDir`).
-   `update` : contrôle la cadence de rafraîchissement et l'exécution de la maintenance : (`interval`, `debounceMs`, `onBoot`, `waitForBootSync`, `embedInterval`, `commandTimeoutMs`, `updateTimeoutMs`, `embedTimeoutMs`).
-   `limits` : limite la charge utile de rappel (`maxResults`, `maxSnippetChars`, `maxInjectedChars`, `timeoutMs`).
-   `scope` : même schéma que [`session.sendPolicy`](../gateway/configuration.md#session). Par défaut, DM uniquement (`deny` tout, `allow` les chats directs) ; assouplissez-le pour afficher les résultats QMD dans les groupes/canaux.
    -   `match.keyPrefix` correspond à la clé de session **normalisée** (minuscules, avec tout préfixe `agent::` supprimé). Exemple : `discord:channel:`.
    -   `match.rawKeyPrefix` correspond à la clé de session **brute** (minuscules), incluant `agent::` . Exemple : `agent:main:discord:`.
    -   Héritage : `match.keyPrefix: "agent:..."` est toujours traité comme un préfixe de clé brute, mais préférez `rawKeyPrefix` pour plus de clarté.
-   Lorsque `scope` refuse une recherche, OpenClaw enregistre un avertissement avec le `channel`/`chatType` dérivé pour faciliter le débogage des résultats vides.
-   Les extraits provenant de l'extérieur de l'espace de travail apparaissent comme `qmd//<relative-path>` dans les résultats de `memory_search` ; `memory_get` comprend ce préfixe et lit depuis la racine de la collection QMD configurée.
-   Lorsque `memory.qmd.sessions.enabled = true`, OpenClaw exporte des transcriptions de session assainies (tours Utilisateur/Assistant) dans une collection QMD dédiée sous `~/.openclaw/agents//qmd/sessions/`, afin que `memory_search` puisse rappeler des conversations récentes sans toucher à l'index SQLite intégré.
-   Les extraits de `memory_search` incluent désormais un pied de page `Source: <path#line>` lorsque `memory.citations` est `auto`/`on` ; définissez `memory.citations = "off"` pour garder les métadonnées de chemin internes (l'agent reçoit toujours le chemin pour `memory_get`, mais le texte de l'extrait omet le pied de page et le prompt système avertit l'agent de ne pas le citer).

**Exemple**

```
memory: {
  backend: "qmd",
  citations: "auto",
  qmd: {
    includeDefaultMemory: true,
    update: { interval: "5m", debounceMs: 15000 },
    limits: { maxResults: 6, timeoutMs: 4000 },
    scope: {
      default: "deny",
      rules: [
        { action: "allow", match: { chatType: "direct" } },
        // Préfixe de clé de session normalisée (supprime `agent:<id>:`).
        { action: "deny", match: { keyPrefix: "discord:channel:" } },
        // Préfixe de clé de session brute (inclut `agent:<id>:`).
        { action: "deny", match: { rawKeyPrefix: "agent:main:discord:" } },
      ]
    },
    paths: [
      { name: "docs", path: "~/notes", pattern: "**/*.md" }
    ]
  }
}
```

**Citations & repli**

-   `memory.citations` s'applique quel que soit le backend (`auto`/`on`/`off`).
-   Lorsque `qmd` s'exécute, nous marquons `status().backend = "qmd"` pour que les diagnostics montrent quel moteur a servi les résultats. Si le sous-processus QMD se termine ou si la sortie JSON ne peut pas être analysée, le gestionnaire de recherche enregistre un avertissement et retourne le fournisseur intégré (embeddings Markdown existants) jusqu'à ce que QMD récupère.

### Chemins de mémoire supplémentaires

Si vous souhaitez indexer des fichiers Markdown en dehors de la disposition par défaut de l'espace de travail, ajoutez des chemins explicites :

```
agents: {
  defaults: {
    memorySearch: {
      extraPaths: ["../team-docs", "/srv/shared-notes/overview.md"]
    }
  }
}
```

Notes :

-   Les chemins peuvent être absolus ou relatifs à l'espace de travail.
-   Les répertoires sont parcourus récursivement pour les fichiers `.md`.
-   Seuls les fichiers Markdown sont indexés.
-   Les liens symboliques sont ignorés (fichiers ou répertoires).

### Embeddings Gemini (natif)

Définissez le fournisseur sur `gemini` pour utiliser directement l'API d'embeddings Gemini :

```
agents: {
  defaults: {
    memorySearch: {
      provider: "gemini",
      model: "gemini-embedding-001",
      remote: {
        apiKey: "YOUR_GEMINI_API_KEY"
      }
    }
  }
}
```

Notes :

-   `remote.baseUrl` est optionnel (par défaut l'URL de base de l'API Gemini).
-   `remote.headers` vous permet d'ajouter des en-têtes supplémentaires si nécessaire.
-   Modèle par défaut : `gemini-embedding-001`.

Si vous souhaitez utiliser un **endpoint personnalisé compatible OpenAI** (OpenRouter, vLLM, ou un proxy), vous pouvez utiliser la configuration `remote` avec le fournisseur OpenAI :

```
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      remote: {
        baseUrl: "https://api.example.com/v1/",
        apiKey: "YOUR_OPENAI_COMPAT_API_KEY",
        headers: { "X-Custom-Header": "value" }
      }
    }
  }
}
```

Si vous ne souhaitez pas définir de clé API, utilisez `memorySearch.provider = "local"` ou définissez `memorySearch.fallback = "none"`. Repli :

-   `memorySearch.fallback` peut être `openai`, `gemini`, `voyage`, `mistral`, `ollama`, `local`, ou `none`.
-   Le fournisseur de repli n'est utilisé que lorsque le fournisseur d'embeddings principal échoue.

Indexation par lot (OpenAI + Gemini + Voyage) :

-   Désactivé par défaut. Définissez `agents.defaults.memorySearch.remote.batch.enabled = true` pour l'activer pour l'indexation de grands corpus (OpenAI, Gemini et Voyage).
-   Le comportement par défaut attend la fin du lot ; ajustez `remote.batch.wait`, `remote.batch.pollIntervalMs` et `remote.batch.timeoutMinutes` si nécessaire.
-   Définissez `remote.batch.concurrency` pour contrôler combien de travaux par lot nous soumettons en parallèle (par défaut : 2).
-   Le mode lot s'applique lorsque `memorySearch.provider = "openai"` ou `"gemini"` et utilise la clé API correspondante.
-   Les travaux par lot Gemini utilisent l'endpoint batch d'embeddings asynchrones et nécessitent la disponibilité de l'API Batch Gemini.

Pourquoi le lot OpenAI est rapide + économique :

-   Pour les réindexations importantes, OpenAI est généralement l'option la plus rapide que nous supportons car nous pouvons soumettre de nombreuses requêtes d'embedding dans un seul travail par lot et laisser OpenAI les traiter de manière asynchrone.
-   OpenAI propose des tarifs réduits pour les charges de travail Batch API, donc les exécutions d'indexation importantes sont généralement moins chères que l'envoi des mêmes requêtes de manière synchrone.
-   Voir la documentation et les tarifs de l'API Batch OpenAI pour plus de détails :
    -   [https://platform.openai.com/docs/api-reference/batch](https://platform.openai.com/docs/api-reference/batch)
    -   [https://platform.openai.com/pricing](https://platform.openai.com/pricing)

Exemple de configuration :

```
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      fallback: "openai",
      remote: {
        batch: { enabled: true, concurrency: 2 }
      },
      sync: { watch: true }
    }
  }
}
```

Outils :

-   `memory_search` — retourne des extraits avec fichier + plages de lignes.
-   `memory_get` — lit le contenu d'un fichier de mémoire par chemin.

Mode local :

-   Définissez `agents.defaults.memorySearch.provider = "local"`.
-   Fournissez `agents.defaults.memorySearch.local.modelPath` (GGUF ou URI `hf:`).
-   Optionnel : définissez `agents.defaults.memorySearch.fallback = "none"` pour éviter le repli distant.

### Comment fonctionnent les outils de mémoire

-   `memory_search` recherche sémantiquement des morceaux Markdown (~400 tokens cible, chevauchement de 80 tokens) depuis `MEMORY.md` + `memory/**/*.md`. Il retourne le texte de l'extrait (limité à ~700 caractères), le chemin du fichier, la plage de lignes, le score, le fournisseur/modèle, et si nous sommes passés des embeddings locaux → distants. Aucune charge utile complète de fichier n'est retournée.
-   `memory_get` lit un fichier Markdown de mémoire spécifique (relatif à l'espace de travail), optionnellement à partir d'une ligne de départ et pour N lignes. Les chemins en dehors de `MEMORY.md` / `memory/` sont rejetés.
-   Les deux outils sont activés uniquement lorsque `memorySearch.enabled` est résolu à vrai pour l'agent.

### Ce qui est indexé (et quand)

-   Type de fichier : Markdown uniquement (`MEMORY.md`, `memory/**/*.md`).
-   Stockage de l'index : SQLite par agent à `~/.openclaw/memory/.sqlite` (configurable via `agents.defaults.memorySearch.store.path`, supporte le jeton `{agentId}`).
-   Fraîcheur : un observateur sur `MEMORY.md` + `memory/` marque l'index comme sale (anti-rebond 1,5s). La synchronisation est planifiée au démarrage de la session, à la recherche, ou à un intervalle et s'exécute de manière asynchrone. Les transcriptions de session utilisent des seuils delta pour déclencher une synchronisation en arrière-plan.
-   Déclencheurs de réindexation : l'index stocke l'**empreinte du fournisseur/modèle d'embedding + du point de terminaison + des paramètres de découpage**. Si l'un de ceux-ci change, OpenClaw réinitialise et réindexe automatiquement l'ensemble du stock.

### Recherche hybride (BM25 + vectorielle)

Lorsqu'elle est activée, OpenClaw combine :

-   **Similarité vectorielle** (correspondance sémantique, la formulation peut différer)
-   **Pertinence des mots-clés BM25** (tokens exacts comme les ID, variables d'environnement, symboles de code)

Si la recherche en texte intégral n'est pas disponible sur votre plateforme, OpenClaw revient à la recherche vectorielle uniquement.

#### Pourquoi hybride ?

La recherche vectorielle est excellente pour « cela signifie la même chose » :

-   « hôte de la passerelle Mac Studio » vs « la machine qui exécute la passerelle »
-   « anti-rebond des mises à jour de fichiers » vs « éviter l'indexation à chaque écriture »

Mais elle peut être faible sur les tokens exacts à fort signal :

-   ID (`a828e60`, `b3b9895a…`)
-   symboles de code (`memorySearch.query.hybrid`)
-   chaînes d'erreur (« sqlite-vec unavailable »)

BM25 (texte intégral) est l'opposé : fort sur les tokens exacts, plus faible sur les paraphrases. La recherche hybride est un compromis pragmatique : **utilisez les deux signaux de récupération** pour obtenir de bons résultats à la fois pour les requêtes en « langage naturel » et les requêtes « aiguille dans une botte de foin ».

#### Comment nous fusionnons les résultats (la conception actuelle)

Ébauche d'implémentation :

1.  Récupérez un pool de candidats des deux côtés :

-   **Vectoriel** : top `maxResults * candidateMultiplier` par similarité cosinus.
-   **BM25** : top `maxResults * candidateMultiplier` par rang BM25 FTS5 (plus bas est meilleur).

2.  Convertissez le rang BM25 en un score de type 0..1 :

-   `textScore = 1 / (1 + max(0, bm25Rank))`

3.  Unissez les candidats par identifiant de morceau et calculez un score pondéré :

-   `finalScore = vectorWeight * vectorScore + textWeight * textScore`

Notes :

-   `vectorWeight` + `textWeight` est normalisé à 1.0 dans la résolution de configuration, donc les poids se comportent comme des pourcentages.
-   Si les embeddings ne sont pas disponibles (ou si le fournisseur retourne un vecteur nul), nous exécutons toujours BM25 et retournons les correspondances de mots-clés.
-   Si FTS5 ne peut pas être créé, nous conservons la recherche vectorielle uniquement (pas d'échec dur).

Ce n'est pas « parfait selon la théorie de la RI », mais c'est simple, rapide et tend à améliorer le rappel/précision sur des notes réelles. Si nous voulons devenir plus sophistiqués plus tard, les étapes suivantes courantes sont la Fusion de Rang Réciproque (RRF) ou la normalisation des scores (min/max ou z-score) avant le mélange.

#### Pipeline de post-traitement

Après avoir fusionné les scores vectoriels et textuels, deux étapes de post-traitement optionnelles affinent la liste des résultats avant qu'elle n'atteigne l'agent :

```bash
Vectoriel + Mots-clés → Fusion pondérée → Décroissance temporelle → Tri → MMR → Top-K Résultats
```

Les deux étapes sont **désactivées par défaut** et peuvent être activées indépendamment.

#### Reclassement MMR (diversité)

Lorsque la recherche hybride retourne des résultats, plusieurs morceaux peuvent contenir un contenu similaire ou se chevaucher. Par exemple, une recherche pour « configuration réseau domestique » pourrait retourner cinq extraits presque identiques provenant de différentes notes quotidiennes qui mentionnent toutes la même configuration de routeur. **MMR (Maximal Marginal Relevance)** reclasse les résultats pour équilibrer pertinence et diversité, garantissant que les premiers résultats couvrent différents aspects de la requête au lieu de répéter la même information. Comment cela fonctionne :

1.  Les résultats sont notés par leur pertinence d'origine (score pondéré vectoriel + BM25).
2.  MMR sélectionne itérativement les résultats qui maximisent : `λ × relevance − (1−λ) × max_similarity_to_selected`.
3.  La similarité entre les résultats est mesurée en utilisant la similarité textuelle de Jaccard sur le contenu tokenisé.

Le paramètre `lambda` contrôle le compromis :

-   `lambda = 1.0` → pertinence pure (aucune pénalité de diversité)
-   `lambda = 0.0` → diversité maximale (ignore la pertinence)
-   Par défaut : `0.7` (équilibré, léger biais de pertinence)

**Exemple — requête : « configuration réseau domestique »** Étant donné ces fichiers de mémoire :

```bash
memory/2026-02-10.md  → "Routeur Omada configuré, VLAN 10 défini pour les appareils IoT"
memory/2026-02-08.md  → "Routeur Omada configuré, IoT déplacé vers VLAN 10"
memory/2026-02-05.md  → "DNS AdGuard configuré sur 192.168.10.2"
memory/network.md     → "Routeur : Omada ER605, AdGuard : 192.168.10.2, VLAN 10 : IoT"
```

 Sans MMR — top 3 résultats :

```bash
1. memory/2026-02-10.md  (score : 0.92)  ← routeur + VLAN
2. memory/2026-02-08.md  (score : 0.89)  ← routeur + VLAN (quasi-duplicat !)
3. memory/network.md     (score : 0.85)  ← document de référence
```

 Avec MMR (λ=0.7) — top 3 résultats :

```bash
1. memory/2026-02-10.md  (score : 0.92)  ← routeur + VLAN
2. memory/network.md     (score : 0.85)  ← document de référence (divers !)
3. memory/2026-02-05.md  (score : 0.78)  ← DNS AdGuard (divers !)
```

Le quasi-duplicat du 8 février disparaît, et l'agent obtient trois informations distinctes. **Quand l'activer :** Si vous remarquez que `memory_search` retourne des extraits redondants ou quasi-duplicats, surtout avec des notes quotidiennes qui répètent souvent des informations similaires d'un jour à l'autre.

#### Décroissance temporelle (boost de récence)

Les agents avec des notes quotidiennes accumulent des centaines de fichiers datés au fil du temps. Sans décroissance, une note bien formulée datant de six mois peut surpasser la mise à jour d'hier sur le même sujet. **La décroissance temporelle** applique un multiplicateur exponentiel aux scores en fonction de l'âge de chaque résultat, de sorte que les souvenirs récents se classent naturellement plus haut tandis que les anciens s'estompent :

```bash
decayedScore = score × e^(-λ × ageInDays)
```

où `λ = ln(2) / halfLifeDays`. Avec la demi-vie par défaut de 30 jours :

-   Notes d'aujourd'hui : **100%** du score original
-   Il y a 7 jours : **~84%**
-   Il y a 30 jours : **50%**
-   Il y a 90 jours : **12.5%**
-   Il y a 180 jours : **~1.6%**

**Les fichiers pérennes ne sont jamais décroissants :**

-   `MEMORY.md` (fichier de mémoire racine)
-   Fichiers non datés dans `memory/` (par exemple, `memory/projects.md`, `memory/network.md`)
-   Ceux-ci contiennent des informations de référence durables qui doivent toujours être classées normalement.

**Fichiers quotidiens datés** (`memory/AAAA-MM-JJ.md`) utilisent la date extraite du nom de fichier. D'autres sources (par exemple, transcriptions de session) utilisent le temps de modification du fichier (`mtime`). **Exemple — requête : « quel est l'horaire de travail de Rod ? »** Étant donné ces fichiers de mémoire (aujourd'hui est le 10 février) :

```bash
memory/2025-09-15.md  → "Rod travaille lun-ven, standup à 10h, pairing à 14h"  (148 jours)
memory/2026-02-10.md  → "Rod a un standup à 14:15, 1:1 avec Zeb à 14:45"    (aujourd'hui)
memory/2026-02-03.md  → "Rod a rejoint une nouvelle équipe, standup déplacé à 14:15" (7 jours)
```

 Sans décroissance :

```bash
1. memory/2025-09-15.md  (score : 0.91)  ← meilleure correspondance sémantique, mais obsolète !
2. memory/2026-02-10.md  (score : 0.82)
3. memory/2026-02-03.md  (score : 0.80)
```

 Avec décroissance (halfLife=30) :

```bash
1. memory/2026-02-10.md  (score : 0.82 × 1.00 = 0.82)  ← aujourd'hui, pas de décroissance
2. memory/2026-02-03.md  (score : 0.80 × 0.85 = 0.68)  ← 7 jours, légère décroissance
3. memory/2025-09-15.md  (score : 0.91 × 0.03 = 0.03)  ← 148 jours, presque disparu
```

La note obsolète de septembre tombe en bas malgré la meilleure correspondance sémantique brute. **Quand l'activer :** Si votre agent a des mois de notes quotidiennes et que vous constatez que des informations anciennes et obsolètes surpassent le contexte récent. Une demi-vie de 30 jours fonctionne bien pour les flux de travail riches en notes quotidiennes ; augmentez-la (par exemple, 90 jours) si vous référencez fréquemment des notes plus anciennes.

#### Configuration

Les deux fonctionnalités sont configurées sous `memorySearch.query