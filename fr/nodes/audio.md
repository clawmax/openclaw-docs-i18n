title: "Configuration du traitement audio et des notes vocales pour OpenClaw AI"
description: "Apprenez à configurer OpenClaw AI pour transcrire l'audio et les notes vocales en utilisant des fournisseurs comme OpenAI, Deepgram ou des CLI locaux. Configurez la détection automatique, les solutions de repli et la détection des mentions pour les groupes."
keywords: ["transcription audio", "notes vocales", "configuration openclaw", "whisper cli", "deepgram", "audio openai", "détection de mentions", "traitement des médias"]
---

  Médias et appareils

  
# Audio et notes vocales

## Ce qui fonctionne

-   **Compréhension des médias (audio)** : Si la compréhension audio est activée (ou détectée automatiquement), OpenClaw :
    1.  Localise la première pièce jointe audio (chemin local ou URL) et la télécharge si nécessaire.
    2.  Applique `maxBytes` avant d'envoyer à chaque entrée de modèle.
    3.  Exécute la première entrée de modèle éligible dans l'ordre (fournisseur ou CLI).
    4.  En cas d'échec ou de saut (taille/délai d'attente), elle essaie l'entrée suivante.
    5.  En cas de succès, elle remplace `Body` par un bloc `[Audio]` et définit `{{Transcript}}`.
-   **Analyse des commandes** : Lorsque la transcription réussit, `CommandBody`/`RawBody` sont définis sur la transcription pour que les commandes slash fonctionnent toujours.
-   **Journalisation détaillée** : En mode `--verbose`, nous enregistrons quand la transcription s'exécute et quand elle remplace le corps.

## Détection automatique (par défaut)

Si vous **ne configurez pas de modèles** et que `tools.media.audio.enabled` n'est **pas** défini sur `false`, OpenClaw détecte automatiquement dans cet ordre et s'arrête à la première option fonctionnelle :

1.  **CLI locaux** (s'ils sont installés)
    -   `sherpa-onnx-offline` (nécessite `SHERPA_ONNX_MODEL_DIR` avec encodeur/décodeur/joiner/tokens)
    -   `whisper-cli` (de `whisper-cpp` ; utilise `WHISPER_CPP_MODEL` ou le modèle tiny inclus)
    -   `whisper` (CLI Python ; télécharge les modèles automatiquement)
2.  **CLI Gemini** (`gemini`) utilisant `read_many_files`
3.  **Clés de fournisseur** (OpenAI → Groq → Deepgram → Google)

Pour désactiver la détection automatique, définissez `tools.media.audio.enabled: false`. Pour personnaliser, définissez `tools.media.audio.models`. Remarque : La détection des binaires est faite au mieux sur macOS/Linux/Windows ; assurez-vous que le CLI est dans le `PATH` (nous développons `~`), ou définissez un modèle CLI explicite avec un chemin de commande complet.

## Exemples de configuration

### Fournisseur + repli CLI (OpenAI + Whisper CLI)

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        maxBytes: 20971520,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
            timeoutSeconds: 45,
          },
        ],
      },
    },
  },
}
```

### Fournisseur uniquement avec restriction par portée

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        scope: {
          default: "allow",
          rules: [{ action: "deny", match: { chatType: "group" } }],
        },
        models: [{ provider: "openai", model: "gpt-4o-mini-transcribe" }],
      },
    },
  },
}
```

### Fournisseur uniquement (Deepgram)

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

### Fournisseur uniquement (Mistral Voxtral)

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

### Renvoyer la transcription dans le chat (opt-in)

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        echoTranscript: true, // par défaut est false
        echoFormat: '📝 "{transcript}"', // optionnel, supporte {transcript}
        models: [{ provider: "openai", model: "gpt-4o-mini-transcribe" }],
      },
    },
  },
}
```

## Notes et limites

-   L'authentification auprès des fournisseurs suit l'ordre d'authentification standard des modèles (profils d'authentification, variables d'environnement, `models.providers.*.apiKey`).
-   Deepgram récupère `DEEPGRAM_API_KEY` lorsque `provider: "deepgram"` est utilisé.
-   Détails de configuration Deepgram : [Deepgram (transcription audio)](../providers/deepgram.md).
-   Détails de configuration Mistral : [Mistral](../providers/mistral.md).
-   Les fournisseurs audio peuvent remplacer `baseUrl`, `headers` et `providerOptions` via `tools.media.audio`.
-   La limite de taille par défaut est de 20 Mo (`tools.media.audio.maxBytes`). Les fichiers audio trop volumineux sont ignorés pour ce modèle et l'entrée suivante est essayée.
-   Les petits fichiers audio vides de moins de 1024 octets sont ignorés avant la transcription par le fournisseur/CLI.
-   La valeur par défaut de `maxChars` pour l'audio est **non définie** (transcription complète). Définissez `tools.media.audio.maxChars` ou `maxChars` par entrée pour tronquer la sortie.
-   La valeur par défaut automatique d'OpenAI est `gpt-4o-mini-transcribe` ; définissez `model: "gpt-4o-transcribe"` pour une précision supérieure.
-   Utilisez `tools.media.audio.attachments` pour traiter plusieurs notes vocales (`mode: "all"` + `maxAttachments`).
-   La transcription est disponible pour les modèles sous la forme `{{Transcript}}`.
-   `tools.media.audio.echoTranscript` est désactivé par défaut ; activez-le pour envoyer une confirmation de transcription dans le chat d'origine avant le traitement par l'agent.
-   `tools.media.audio.echoFormat` personnalise le texte de l'écho (espace réservé : `{transcript}`).
-   La sortie standard du CLI est limitée (5 Mo) ; gardez la sortie CLI concise.

### Prise en charge de l'environnement proxy

La transcription audio basée sur un fournisseur respecte les variables d'environnement proxy sortantes standard :

-   `HTTPS_PROXY`
-   `HTTP_PROXY`
-   `https_proxy`
-   `http_proxy`

Si aucune variable d'environnement proxy n'est définie, la sortie directe est utilisée. Si la configuration du proxy est mal formée, OpenClaw enregistre un avertissement et revient à une récupération directe.

## Détection des mentions dans les groupes

Lorsque `requireMention: true` est défini pour un chat de groupe, OpenClaw transcrit maintenant l'audio **avant** de vérifier les mentions. Cela permet de traiter les notes vocales même lorsqu'elles contiennent des mentions. **Fonctionnement :**

1.  Si un message vocal n'a pas de corps texte et que le groupe nécessite des mentions, OpenClaw effectue une transcription "préliminaire".
2.  La transcription est vérifiée pour les modèles de mentions (par exemple, `@BotName`, déclencheurs d'emoji).
3.  Si une mention est trouvée, le message passe par le pipeline complet de réponse.
4.  La transcription est utilisée pour la détection des mentions afin que les notes vocales puissent passer la barrière des mentions.

**Comportement de repli :**

-   Si la transcription échoue pendant la phase préliminaire (délai d'attente, erreur API, etc.), le message est traité en fonction de la détection de mentions basée uniquement sur le texte.
-   Cela garantit que les messages mixtes (texte + audio) ne sont jamais incorrectement ignorés.

**Désactivation par groupe/sujet Telegram :**

-   Définissez `channels.telegram.groups..disableAudioPreflight: true` pour ignorer les vérifications de mentions par transcription préliminaire pour ce groupe.
-   Définissez `channels.telegram.groups..topics..disableAudioPreflight` pour remplacer par sujet (`true` pour ignorer, `false` pour forcer l'activation).
-   La valeur par défaut est `false` (préliminaire activée lorsque les conditions de mention sont remplies).

**Exemple :** Un utilisateur envoie une note vocale disant "Hé @Claude, quel temps fait-il ?" dans un groupe Telegram avec `requireMention: true`. La note vocale est transcrite, la mention est détectée et l'agent répond.

## Pièges

-   Les règles de portée utilisent le principe du premier match gagnant. `chatType` est normalisé en `direct`, `group` ou `room`.
-   Assurez-vous que votre CLI se termine avec le code 0 et imprime du texte brut ; le JSON doit être traité via `jq -r .text`.
-   Pour `parakeet-mlx`, si vous passez `--output-dir`, OpenClaw lit `<output-dir>/<media-basename>.txt` lorsque `--output-format` est `txt` (ou omis) ; les formats de sortie non-`txt` reviennent à l'analyse de la sortie standard.
-   Gardez des délais d'attente raisonnables (`timeoutSeconds`, par défaut 60s) pour éviter de bloquer la file d'attente des réponses.
-   La transcription préliminaire ne traite que la **première** pièce jointe audio pour la détection des mentions. Les fichiers audio supplémentaires sont traités pendant la phase principale de compréhension des médias.

[Prise en charge des images et médias](./images.md)[Capture d'écran](./camera.md)