

  Médias et appareils

  
# Compréhension multimédia

OpenClaw peut **résumer les médias entrants** (image/audio/vidéo) avant l'exécution du pipeline de réponse. Il détecte automatiquement quand des outils locaux ou des clés de fournisseur sont disponibles, et peut être désactivé ou personnalisé. Si la compréhension est désactivée, les modèles reçoivent toujours les fichiers/URLs originaux comme d'habitude.

## Objectifs

-   Optionnel : pré-digérer les médias entrants en texte court pour un routage plus rapide et une meilleure analyse des commandes.
-   Préserver la livraison des médias originaux au modèle (toujours).
-   Prendre en charge les **API de fournisseurs** et les **solutions de repli CLI**.
-   Permettre plusieurs modèles avec un repli ordonné (erreur/taille/timeout).

## Comportement de haut niveau

1.  Collecter les pièces jointes entrantes (`MediaPaths`, `MediaUrls`, `MediaTypes`).
2.  Pour chaque capacité activée (image/audio/vidéo), sélectionner les pièces jointes selon la politique (par défaut : **la première**).
3.  Choisir la première entrée de modèle éligible (taille + capacité + authentification).
4.  Si un modèle échoue ou si le média est trop volumineux, **revenir à l'entrée suivante**.
5.  En cas de succès :
    -   Le `Body` devient un bloc `[Image]`, `[Audio]`, ou `[Video]`.
    -   L'audio définit `{{Transcript}}` ; l'analyse des commandes utilise le texte de la légende lorsqu'il est présent, sinon la transcription.
    -   Les légendes sont préservées sous la forme `User text:` à l'intérieur du bloc.

Si la compréhension échoue ou est désactivée, **le flux de réponse continue** avec le corps original + les pièces jointes.

## Aperçu de la configuration

`tools.media` prend en charge des **modèles partagés** ainsi que des remplacements par capacité :

-   `tools.media.models` : liste de modèles partagés (utiliser `capabilities` pour filtrer).
-   `tools.media.image` / `tools.media.audio` / `tools.media.video` :
    -   valeurs par défaut (`prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`)
    -   remplacements de fournisseur (`baseUrl`, `headers`, `providerOptions`)
    -   options audio Deepgram via `tools.media.audio.providerOptions.deepgram`
    -   contrôles d'écho de transcription audio (`echoTranscript`, par défaut `false` ; `echoFormat`)
    -   liste **`models` par capacité** facultative (prioritaire avant les modèles partagés)
    -   politique `attachments` (`mode`, `maxAttachments`, `prefer`)
    -   `scope` (filtrage facultatif par canal/chatType/clé de session)
-   `tools.media.concurrency` : nombre maximum d'exécutions concurrentes par capacité (par défaut **2**).

```json
{
  tools: {
    media: {
      models: [
        /* shared list */
      ],
      image: {
        /* optional overrides */
      },
      audio: {
        /* optional overrides */
        echoTranscript: true,
        echoFormat: '📝 "{transcript}"',
      },
      video: {
        /* optional overrides */
      },
    },
  },
}
```

### Entrées de modèle

Chaque entrée `models[]` peut être **fournisseur** ou **CLI** :

```json
{
  type: "provider", // default if omitted
  provider: "openai",
  model: "gpt-5.2",
  prompt: "Describe the image in <= 500 chars.",
  maxChars: 500,
  maxBytes: 10485760,
  timeoutSeconds: 60,
  capabilities: ["image"], // optional, used for multi‑modal entries
  profile: "vision-profile",
  preferredProfile: "vision-fallback",
}
```

```json
{
  type: "cli",
  command: "gemini",
  args: [
    "-m",
    "gemini-3-flash",
    "--allowed-tools",
    "read_file",
    "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
  ],
  maxChars: 500,
  maxBytes: 52428800,
  timeoutSeconds: 120,
  capabilities: ["video", "image"],
}
```

Les modèles CLI peuvent également utiliser :

-   `{{MediaDir}}` (répertoire contenant le fichier multimédia)
-   `{{OutputDir}}` (répertoire temporaire créé pour cette exécution)
-   `{{OutputBase}}` (chemin de base du fichier temporaire, sans extension)

## Valeurs par défaut et limites

Valeurs par défaut recommandées :

-   `maxChars` : **500** pour image/vidéo (court, adapté aux commandes)
-   `maxChars` : **non défini** pour l'audio (transcription complète sauf si vous définissez une limite)
-   `maxBytes` :
    -   image : **10 Mo**
    -   audio : **20 Mo**
    -   vidéo : **50 Mo**

Règles :

-   Si le média dépasse `maxBytes`, ce modèle est ignoré et le **modèle suivant est essayé**.
-   Les fichiers audio de moins de **1024 octets** sont traités comme vides/corrompus et ignorés avant la transcription par le fournisseur/CLI.
-   Si le modèle renvoie plus de `maxChars`, la sortie est tronquée.
-   `prompt` est par défaut un simple "Décrivez le ." plus la directive `maxChars` (image/vidéo uniquement).
-   Si `.enabled: true` mais qu'aucun modèle n'est configuré, OpenClaw essaie le **modèle de réponse actif** lorsque son fournisseur prend en charge la capacité.

### Détection automatique de la compréhension multimédia (par défaut)

Si `tools.media..enabled` n'est **pas** défini sur `false` et que vous n'avez pas configuré de modèles, OpenClaw détecte automatiquement dans cet ordre et **s'arrête à la première option fonctionnelle** :

1.  **CLIs locaux** (audio uniquement ; si installés)
    -   `sherpa-onnx-offline` (nécessite `SHERPA_ONNX_MODEL_DIR` avec encodeur/décodeur/joiner/tokens)
    -   `whisper-cli` (`whisper-cpp` ; utilise `WHISPER_CPP_MODEL` ou le modèle tiny intégré)
    -   `whisper` (CLI Python ; télécharge les modèles automatiquement)
2.  **CLI Gemini** (`gemini`) utilisant `read_many_files`
3.  **Clés de fournisseur**
    -   Audio : OpenAI → Groq → Deepgram → Google
    -   Image : OpenAI → Anthropic → Google → MiniMax
    -   Vidéo : Google

Pour désactiver la détection automatique, définissez :

```json
{
  tools: {
    media: {
      audio: {
        enabled: false,
      },
    },
  },
}
```

Note : La détection binaire est faite au mieux sur macOS/Linux/Windows ; assurez-vous que le CLI est dans le `PATH` (nous développons `~`), ou définissez un modèle CLI explicite avec un chemin de commande complet.

### Prise en charge de l'environnement proxy (modèles fournisseur)

Lorsque la compréhension multimédia basée sur un fournisseur pour l'**audio** et la **vidéo** est activée, OpenClaw respecte les variables d'environnement de proxy sortant standard pour les appels HTTP des fournisseurs :

-   `HTTPS_PROXY`
-   `HTTP_PROXY`
-   `https_proxy`
-   `http_proxy`

Si aucune variable d'environnement de proxy n'est définie, la compréhension multimédia utilise une sortie directe. Si la valeur du proxy est mal formée, OpenClaw enregistre un avertissement et revient à une récupération directe.

## Capacités (optionnel)

Si vous définissez `capabilities`, l'entrée ne s'exécute que pour ces types de média. Pour les listes partagées, OpenClaw peut déduire les valeurs par défaut :

-   `openai`, `anthropic`, `minimax` : **image**
-   `google` (API Gemini) : **image + audio + vidéo**
-   `groq` : **audio**
-   `deepgram` : **audio**

Pour les entrées CLI, **définissez `capabilities` explicitement** pour éviter les correspondances inattendues. Si vous omettez `capabilities`, l'entrée est éligible pour la liste dans laquelle elle apparaît.

## Matrice de support des fournisseurs (intégrations OpenClaw)

| Capacité | Intégration fournisseur | Notes |
| --- | --- | --- |
| Image | OpenAI / Anthropic / Google / autres via `pi-ai` | Tout modèle capable de traiter les images dans le registre fonctionne. |
| Audio | OpenAI, Groq, Deepgram, Google, Mistral | Transcription par le fournisseur (Whisper/Deepgram/Gemini/Voxtral). |
| Vidéo | Google (API Gemini) | Compréhension vidéo par le fournisseur. |

## Guide de sélection des modèles

-   Préférez le modèle de dernière génération le plus performant disponible pour chaque capacité multimédia lorsque la qualité et la sécurité sont importantes.
-   Pour les agents avec outils gérant des entrées non fiables, évitez les modèles multimédias plus anciens/plus faibles.
-   Gardez au moins une solution de repli par capacité pour la disponibilité (modèle de qualité + modèle plus rapide/économique).
-   Les solutions de repli CLI (`whisper-cli`, `whisper`, `gemini`) sont utiles lorsque les API des fournisseurs sont indisponibles.
-   Note `parakeet-mlx` : avec `--output-dir`, OpenClaw lit `<output-dir>/<media-basename>.txt` lorsque le format de sortie est `txt` (ou non spécifié) ; les formats non-`txt` reviennent à stdout.

## Politique de pièces jointes

`attachments` par capacité contrôle quelles pièces jointes sont traitées :

-   `mode` : `first` (par défaut) ou `all`
-   `maxAttachments` : limiter le nombre traité (par défaut **1**)
-   `prefer` : `first`, `last`, `path`, `url`

Lorsque `mode: "all"`, les sorties sont étiquetées `[Image 1/2]`, `[Audio 2/2]`, etc.

## Exemples de configuration

### 1) Liste de modèles partagés + remplacements

```json
{
  tools: {
    media: {
      models: [
        { provider: "openai", model: "gpt-5.2", capabilities: ["image"] },
        {
          provider: "google",
          model: "gemini-3-flash-preview",
          capabilities: ["image", "audio", "video"],
        },
        {
          type: "cli",
          command: "gemini",
          args: [
            "-m",
            "gemini-3-flash",
            "--allowed-tools",
            "read_file",
            "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
          ],
          capabilities: ["image", "video"],
        },
      ],
      audio: {
        attachments: { mode: "all", maxAttachments: 2 },
      },
      video: {
        maxChars: 500,
      },
    },
  },
}
```

### 2) Audio + Vidéo uniquement (image désactivée)

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
          },
        ],
      },
      video: {
        enabled: true,
        maxChars: 500,
        models: [
          { provider: "google", model: "gemini-3-flash-preview" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
            ],
          },
        ],
      },
    },
  },
}
```

### 3) Compréhension d'image optionnelle

```json
{
  tools: {
    media: {
      image: {
        enabled: true,
        maxBytes: 10485760,
        maxChars: 500,
        models: [
          { provider: "openai", model: "gpt-5.2" },
          { provider: "anthropic", model: "claude-opus-4-6" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
            ],
          },
        ],
      },
    },
  },
}
```

### 4) Entrée multimodale unique (capacités explicites)

```json
{
  tools: {
    media: {
      image: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
      audio: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
      video: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
    },
  },
}
```

## Sortie d'état

Lorsque la compréhension multimédia s'exécute, `/status` inclut une courte ligne de résumé :

```
📎 Media: image ok (openai/gpt-5.2) · audio skipped (maxBytes)
```

Cela montre les résultats par capacité et le fournisseur/modèle choisi le cas échéant.

## Notes

-   La compréhension est **au mieux**. Les erreurs ne bloquent pas les réponses.
-   Les pièces jointes sont toujours transmises aux modèles même lorsque la compréhension est désactivée.
-   Utilisez `scope` pour limiter où la compréhension s'exécute (par ex. uniquement les messages privés).

## Documents connexes

-   [Configuration](../gateway/configuration.md)
-   [Support des images et médias](./images.md)

[Dépannage des nœuds](./troubleshooting.md)[Support des images et médias](./images.md)

---