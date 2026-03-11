

  Fournisseurs

  
# Deepgram

Deepgram est une API de reconnaissance vocale. Dans OpenClaw, il est utilisé pour la **transcription des messages audio/notes vocaux entrants** via `tools.media.audio`. Lorsqu'il est activé, OpenClaw téléverse le fichier audio vers Deepgram et injecte la transcription dans le pipeline de réponse (`{{Transcript}}` + bloc `[Audio]`). Ce n'est **pas en streaming** ; il utilise le point de terminaison de transcription pour fichiers pré-enregistrés. Site web : [https://deepgram.com](https://deepgram.com)  
Documentation : [https://developers.deepgram.com](https://developers.deepgram.com)

## Démarrage rapide

1.  Définissez votre clé API :

```
DEEPGRAM_API_KEY=dg_...
```

2.  Activez le fournisseur :

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "deepgram", model: "nova-3" }],
      },
    },
  },
}
```

## Options

-   `model` : identifiant du modèle Deepgram (par défaut : `nova-3`)
-   `language` : indication de langue (optionnelle)
-   `tools.media.audio.providerOptions.deepgram.detect_language` : activer la détection de langue (optionnelle)
-   `tools.media.audio.providerOptions.deepgram.punctuate` : activer la ponctuation (optionnelle)
-   `tools.media.audio.providerOptions.deepgram.smart_format` : activer le formatage intelligent (optionnelle)

Exemple avec langue :

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "deepgram", model: "nova-3", language: "en" }],
      },
    },
  },
}
```

Exemple avec options Deepgram :

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        providerOptions: {
          deepgram: {
            detect_language: true,
            punctuate: true,
            smart_format: true,
          },
        },
        models: [{ provider: "deepgram", model: "nova-3" }],
      },
    },
  },
}
```

## Notes

-   L'authentification suit l'ordre standard des fournisseurs ; `DEEPGRAM_API_KEY` est le chemin le plus simple.
-   Remplacez les points de terminaison ou les en-têtes avec `tools.media.audio.baseUrl` et `tools.media.audio.headers` lors de l'utilisation d'un proxy.
-   La sortie suit les mêmes règles audio que les autres fournisseurs (limites de taille, délais d'attente, injection de transcription).

[Proxy API Claude Max](./claude-max-api-proxy.md)[GitHub Copilot](./github-copilot.md)