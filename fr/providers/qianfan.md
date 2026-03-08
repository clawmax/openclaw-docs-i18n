

  Fournisseurs

  
# Qianfan

Qianfan est la plateforme MaaS de Baidu, elle fournit une **API unifiée** qui achemine les requêtes vers de nombreux modèles derrière un seul point de terminaison et une seule clé API. Elle est compatible avec OpenAI, donc la plupart des SDK OpenAI fonctionnent en changeant l'URL de base.

## Prérequis

1.  Un compte Baidu Cloud avec accès à l'API Qianfan
2.  Une clé API depuis la console Qianfan
3.  OpenClaw installé sur votre système

## Obtenir votre clé API

1.  Visitez la [Console Qianfan](https://console.bce.baidu.com/qianfan/ais/console/apiKey)
2.  Créez une nouvelle application ou sélectionnez-en une existante
3.  Générez une clé API (format : `bce-v3/ALTAK-...`)
4.  Copiez la clé API pour l'utiliser avec OpenClaw

## Configuration du CLI

```bash
openclaw onboard --auth-choice qianfan-api-key
```

## Documentation associée

-   [Configuration d'OpenClaw](../gateway/configuration.md)
-   [Fournisseurs de modèles](../concepts/model-providers.md)
-   [Configuration d'un Agent](../concepts/agent.md)
-   [Documentation de l'API Qianfan](https://cloud.baidu.com/doc/qianfan-api/s/3m7of64lb)

[OpenRouter](./openrouter.md)[Qwen](./qwen.md)

---