

  Outils intégrés

  
# Brave Search

OpenClaw prend en charge Brave Search comme fournisseur de recherche web pour `web_search`.

## Obtenir une clé API

1.  Créez un compte pour l'API Brave Search sur [https://brave.com/search/api/](https://brave.com/search/api/)
2.  Dans le tableau de bord, choisissez le plan **Data for Search** et générez une clé API.
3.  Stockez la clé dans la configuration (recommandé) ou définissez `BRAVE_API_KEY` dans l'environnement de la Gateway.

## Exemple de configuration

```json
{
  tools: {
    web: {
      search: {
        provider: "brave",
        apiKey: "BRAVE_API_KEY_HERE",
        maxResults: 5,
        timeoutSeconds: 30,
      },
    },
  },
}
```

## Paramètres de l'outil

| Paramètre | Description |
| --- | --- |
| `query` | Requête de recherche (obligatoire) |
| `count` | Nombre de résultats à retourner (1-10, par défaut : 5) |
| `country` | Code pays ISO à 2 lettres (ex. "US", "DE") |
| `language` | Code langue ISO 639-1 pour les résultats de recherche (ex. "en", "de", "fr") |
| `ui_lang` | Code langue ISO pour les éléments de l'interface utilisateur |
| `freshness` | Filtre temporel : `day` (24h), `week`, `month`, ou `year` |
| `date_after` | Uniquement les résultats publiés après cette date (AAAA-MM-JJ) |
| `date_before` | Uniquement les résultats publiés avant cette date (AAAA-MM-JJ) |

**Exemples :**

```javascript
// Recherche spécifique à un pays et une langue
await web_search({
  query: "énergie renouvelable",
  country: "DE",
  language: "de",
});

// Résultats récents (semaine passée)
await web_search({
  query: "actualités IA",
  freshness: "week",
});

// Recherche par plage de dates
await web_search({
  query: "développements IA",
  date_after: "2024-01-01",
  date_before: "2024-06-30",
});
```

## Notes

-   Le plan Data for AI **n'est pas** compatible avec `web_search`.
-   Brave propose des plans payants ; consultez le portail de l'API Brave pour connaître les limites actuelles.
-   Les Conditions d'utilisation de Brave incluent des restrictions sur certaines utilisations liées à l'IA des Résultats de Recherche. Consultez les Conditions d'utilisation de Brave et confirmez que votre utilisation prévue est conforme. Pour les questions juridiques, consultez votre conseiller.
-   Les résultats sont mis en cache pendant 15 minutes par défaut (configurable via `cacheTtlMinutes`).

Voir [Outils web](./tools/web.md) pour la configuration complète de web\_search.

[Outil apply\_patch](./tools/apply-patch.md)[Perplexity Sonar](./perplexity.md)

---