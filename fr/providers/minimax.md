

  Fournisseurs

  
# MiniMax

MiniMax est une entreprise d'IA qui développe la famille de modèles **M2/M2.5**. La version actuelle axée sur le codage est **MiniMax M2.5** (23 décembre 2025), conçue pour des tâches complexes du monde réel. Source : [Note de version MiniMax M2.5](https://www.minimax.io/news/minimax-m25)

## Aperçu du modèle (M2.5)

MiniMax met en avant ces améliorations dans M2.5 :

-   Un **codage multilingue** plus performant (Rust, Java, Go, C++, Kotlin, Objective-C, TS/JS).
-   Une meilleure qualité de **développement web/app** et de sortie esthétique (y compris mobile natif).
-   Une meilleure gestion des **instructions composites** pour les flux de travail de type bureautique, s'appuyant sur une réflexion entrelacée et une exécution intégrée des contraintes.
-   Des **réponses plus concises** avec une utilisation réduite de tokens et des boucles d'itération plus rapides.
-   Une meilleure compatibilité avec les **frameworks d'outils/agents** et la gestion du contexte (Claude Code, Droid/Factory AI, Cline, Kilo Code, Roo Code, BlackBox).
-   Des sorties de **dialogue et de rédaction technique** de plus haute qualité.

## MiniMax M2.5 vs MiniMax M2.5 Highspeed

-   **Vitesse :** `MiniMax-M2.5-highspeed` est le niveau rapide officiel dans la documentation MiniMax.
-   **Coût :** La tarification MiniMax indique le même coût en entrée et un coût en sortie plus élevé pour le highspeed.
-   **Compatibilité :** OpenClaw accepte toujours les configurations héritées `MiniMax-M2.5-Lightning`, mais préférez `MiniMax-M2.5-highspeed` pour une nouvelle installation.

## Choisir une configuration

### OAuth MiniMax (Plan de Codage) — recommandé

**Idéal pour :** une configuration rapide avec le Plan de Codage MiniMax via OAuth, sans clé API requise. Activez le plugin OAuth intégré et authentifiez-vous :

```bash
openclaw plugins enable minimax-portal-auth  # à ignorer si déjà chargé.
openclaw gateway restart  # redémarrez si la passerelle est déjà en cours d'exécution
openclaw onboard --auth-choice minimax-portal
```

Il vous sera demandé de sélectionner un point de terminaison :

-   **Global** - Utilisateurs internationaux (`api.minimax.io`)
-   **CN** - Utilisateurs en Chine (`api.minimaxi.com`)

Voir le [README du plugin OAuth MiniMax](https://github.com/openclaw/openclaw/tree/main/extensions/minimax-portal-auth) pour plus de détails.

### MiniMax M2.5 (Clé API)

**Idéal pour :** MiniMax hébergé avec une API compatible Anthropic. Configurez via la CLI :

-   Exécutez `openclaw configure`
-   Sélectionnez **Model/auth**
-   Choisissez **MiniMax M2.5**

```json
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "minimax/MiniMax-M2.5" } } },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.5",
            name: "MiniMax M2.5",
            reasoning: true,
            input: ["text"],
            cost: { input: 0.3, output: 1.2, cacheRead: 0.03, cacheWrite: 0.12 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
          {
            id: "MiniMax-M2.5-highspeed",
            name: "MiniMax M2.5 Highspeed",
            reasoning: true,
            input: ["text"],
            cost: { input: 0.3, output: 1.2, cacheRead: 0.03, cacheWrite: 0.12 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

### MiniMax M2.5 en secours (exemple)

**Idéal pour :** garder votre modèle de dernière génération le plus puissant comme principal, avec basculement vers MiniMax M2.5. L'exemple ci-dessous utilise Opus comme principal concret ; remplacez-le par votre modèle principal de dernière génération préféré.

```json
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "primary" },
        "minimax/MiniMax-M2.5": { alias: "minimax" },
      },
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["minimax/MiniMax-M2.5"],
      },
    },
  },
}
```

### Optionnel : Local via LM Studio (manuel)

**Idéal pour :** l'inférence locale avec LM Studio. Nous avons observé de très bons résultats avec MiniMax M2.5 sur du matériel puissant (par ex. un ordinateur de bureau/serveur) en utilisant le serveur local de LM Studio. Configurez manuellement via `openclaw.json` :

```json
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.5-gs32" },
      models: { "lmstudio/minimax-m2.5-gs32": { alias: "Minimax" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.5-gs32",
            name: "MiniMax M2.5 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## Configurer via openclaw configure

Utilisez l'assistant de configuration interactif pour configurer MiniMax sans éditer de JSON :

1.  Exécutez `openclaw configure`.
2.  Sélectionnez **Model/auth**.
3.  Choisissez **MiniMax M2.5**.
4.  Sélectionnez votre modèle par défaut lorsque vous y êtes invité.

## Options de configuration

-   `models.providers.minimax.baseUrl` : préférez `https://api.minimax.io/anthropic` (compatible Anthropic) ; `https://api.minimax.io/v1` est optionnel pour les payloads compatibles OpenAI.
-   `models.providers.minimax.api` : préférez `anthropic-messages` ; `openai-completions` est optionnel pour les payloads compatibles OpenAI.
-   `models.providers.minimax.apiKey` : clé API MiniMax (`MINIMAX_API_KEY`).
-   `models.providers.minimax.models` : définissez `id`, `name`, `reasoning`, `contextWindow`, `maxTokens`, `cost`.
-   `agents.defaults.models` : alias des modèles que vous souhaitez dans la liste autorisée.
-   `models.mode` : gardez `merge` si vous souhaitez ajouter MiniMax aux modèles intégrés.

## Notes

-   Les références de modèle sont `minimax/`.
-   Identifiants de modèle recommandés : `MiniMax-M2.5` et `MiniMax-M2.5-highspeed`.
-   API d'utilisation du Plan de Codage : `https://api.minimaxi.com/v1/api/openplatform/coding_plan/remains` (nécessite une clé de plan de codage).
-   Mettez à jour les valeurs de tarification dans `models.json` si vous avez besoin d'un suivi précis des coûts.
-   Lien de parrainage pour le Plan de Codage MiniMax (10% de réduction) : [https://platform.minimax.io/subscribe/coding-plan?code=DbXJTRClnb&source=link](https://platform.minimax.io/subscribe/coding-plan?code=DbXJTRClnb&source=link)
-   Voir [/concepts/model-providers](../concepts/model-providers.md) pour les règles des fournisseurs.
-   Utilisez `openclaw models list` et `openclaw models set minimax/MiniMax-M2.5` pour changer.

## Dépannage

### “Modèle inconnu : minimax/MiniMax-M2.5”

Cela signifie généralement que le **fournisseur MiniMax n'est pas configuré** (aucune entrée de fournisseur et aucun profil d'authentification/clé d'environnement MiniMax trouvé). Un correctif pour cette détection est dans **2026.1.12** (non publié au moment de la rédaction). Corrigez en :

-   Mettant à niveau vers **2026.1.12** (ou en exécutant depuis la source `main`), puis en redémarrant la passerelle.
-   Exécutant `openclaw configure` et en sélectionnant **MiniMax M2.5**, ou
-   Ajoutant le bloc `models.providers.minimax` manuellement, ou
-   Définissant `MINIMAX_API_KEY` (ou un profil d'authentification MiniMax) pour que le fournisseur puisse être injecté.

Assurez-vous que l'identifiant du modèle est **sensible à la casse** :

-   `minimax/MiniMax-M2.5`
-   `minimax/MiniMax-M2.5-highspeed`
-   `minimax/MiniMax-M2.5-Lightning` (hérité)

Puis vérifiez à nouveau avec :

```bash
openclaw models list
```

[Modèles GLM](./glm.md)[Moonshot AI](./moonshot.md)