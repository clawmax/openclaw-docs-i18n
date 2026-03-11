

  Fournisseurs

  
# Mistral

OpenClaw prend en charge Mistral à la fois pour le routage des modèles texte/image (`mistral/...`) et pour la transcription audio via Voxtral dans la compréhension multimédia. Mistral peut également être utilisé pour les embeddings de mémoire (`memorySearch.provider = "mistral"`).

## Configuration CLI

```bash
openclaw onboard --auth-choice mistral-api-key
# ou non-interactif
openclaw onboard --mistral-api-key "$MISTRAL_API_KEY"
```

## Extrait de configuration (fournisseur LLM)

```json
{
  env: { MISTRAL_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "mistral/mistral-large-latest" } } },
}
```

## Extrait de configuration (transcription audio avec Voxtral)

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "mistral", model: "voxtral-mini-latest" }],
      },
    },
  },
}
```

## Notes

-   L'authentification Mistral utilise `MISTRAL_API_KEY`.
-   L'URL de base du fournisseur est par défaut `https://api.mistral.ai/v1`.
-   Le modèle par défaut lors de l'intégration est `mistral/mistral-large-latest`.
-   Le modèle audio par défaut pour Mistral dans la compréhension multimédia est `voxtral-mini-latest`.
-   Le chemin de transcription multimédia utilise `/v1/audio/transcriptions`.
-   Le chemin des embeddings de mémoire utilise `/v1/embeddings` (modèle par défaut : `mistral-embed`).

[Moonshot AI](./moonshot.md)[NVIDIA](./nvidia.md)

---