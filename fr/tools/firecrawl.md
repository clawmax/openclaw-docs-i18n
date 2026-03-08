title: "Configurer Firecrawl pour l'extraction de contenu web dans OpenClaw"
description: "Apprenez à configurer et utiliser Firecrawl comme extracteur web de secours dans OpenClaw pour scraper des sites riches en JavaScript avec contournement de bots et mise en cache."
keywords: ["firecrawl", "scraping web", "extraction de contenu", "outils openclaw", "contournement de bot", "configuration api", "récupération web", "mode proxy"]
---

  Outils intégrés

  
# Firecrawl

OpenClaw peut utiliser **Firecrawl** comme extracteur de secours pour `web_fetch`. C'est un service hébergé d'extraction de contenu qui prend en charge le contournement de bots et la mise en cache, ce qui aide pour les sites riches en JavaScript ou les pages qui bloquent les requêtes HTTP simples.

## Obtenir une clé API

1.  Créez un compte Firecrawl et générez une clé API.
2.  Stockez-la dans la configuration ou définissez `FIRECRAWL_API_KEY` dans l'environnement de la passerelle.

## Configurer Firecrawl

```json
{
  tools: {
    web: {
      fetch: {
        firecrawl: {
          apiKey: "FIRECRAWL_API_KEY_HERE",
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 172800000,
          timeoutSeconds: 60,
        },
      },
    },
  },
}
```

Notes :

-   `firecrawl.enabled` est activé par défaut lorsqu'une clé API est présente.
-   `maxAgeMs` contrôle l'âge maximum des résultats en cache (ms). La valeur par défaut est de 2 jours.

## Furtivité / contournement de bots

Firecrawl expose un paramètre de **mode proxy** pour le contournement de bots (`basic`, `stealth`, ou `auto`). OpenClaw utilise toujours `proxy: "auto"` plus `storeInCache: true` pour les requêtes Firecrawl. Si le proxy est omis, Firecrawl utilise par défaut `auto`. `auto` réessaie avec des proxies furtifs si une tentative basique échoue, ce qui peut utiliser plus de crédits qu'un scraping en mode basique uniquement.

## Comment web\_fetch utilise Firecrawl

Ordre d'extraction de `web_fetch` :

1.  Readability (local)
2.  Firecrawl (si configuré)
3.  Nettoyage HTML basique (dernier recours)

Voir [Outils Web](./web.md) pour la configuration complète des outils web.

[Approbations Exec](./exec-approvals.md)[Tâche LLM](./llm-task.md)

---