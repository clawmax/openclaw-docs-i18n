

  Médias et appareils

  
# Synthèse vocale

OpenClaw peut convertir les réponses sortantes en audio en utilisant ElevenLabs, OpenAI ou Edge TTS. Cela fonctionne partout où OpenClaw peut envoyer de l'audio ; Telegram reçoit une bulle de note vocale ronde.

## Services pris en charge

-   **ElevenLabs** (fournisseur principal ou de secours)
-   **OpenAI** (fournisseur principal ou de secours ; également utilisé pour les résumés)
-   **Edge TTS** (fournisseur principal ou de secours ; utilise `node-edge-tts`, par défaut lorsqu'aucune clé API n'est présente)

### Notes sur Edge TTS

Edge TTS utilise le service de synthèse vocale neuronale en ligne de Microsoft Edge via la bibliothèque `node-edge-tts`. C'est un service hébergé (pas local), utilise les points de terminaison de Microsoft et ne nécessite pas de clé API. `node-edge-tts` expose les options de configuration de la parole et les formats de sortie, mais toutes les options ne sont pas prises en charge par le service Edge. citeturn2search0 Comme Edge TTS est un service web public sans contrat de niveau de service (SLA) ou quota publié, considérez-le comme un service au mieux. Si vous avez besoin de limites et d'un support garantis, utilisez OpenAI ou ElevenLabs. L'API REST Speech de Microsoft documente une limite de 10 minutes d'audio par requête ; Edge TTS ne publie pas de limites, donc supposez des limites similaires ou inférieures. citeturn0search3

## Clés optionnelles

Si vous souhaitez utiliser OpenAI ou ElevenLabs :

-   `ELEVENLABS_API_KEY` (ou `XI_API_KEY`)
-   `OPENAI_API_KEY`

Edge TTS ne nécessite **pas** de clé API. Si aucune clé API n'est trouvée, OpenClaw utilise par défaut Edge TTS (sauf s'il est désactivé via `messages.tts.edge.enabled=false`). Si plusieurs fournisseurs sont configurés, le fournisseur sélectionné est utilisé en premier et les autres sont des options de secours. Le résumé automatique utilise le `summaryModel` configuré (ou `agents.defaults.model.primary`), donc ce fournisseur doit également être authentifié si vous activez les résumés.

## Liens des services

-   [Guide OpenAI Text-to-Speech](https://platform.openai.com/docs/guides/text-to-speech)
-   [Référence de l'API Audio OpenAI](https://platform.openai.com/docs/api-reference/audio)
-   [ElevenLabs Text to Speech](https://elevenlabs.io/docs/api-reference/text-to-speech)
-   [Authentification ElevenLabs](https://elevenlabs.io/docs/api-reference/authentication)
-   [node-edge-tts](https://github.com/SchneeHertz/node-edge-tts)
-   [Formats de sortie Microsoft Speech](https://learn.microsoft.com/azure/ai-services/speech-service/rest-text-to-speech#audio-outputs)

## Est-il activé par défaut ?

Non. Le TTS automatique est **désactivé** par défaut. Activez-le dans la configuration avec `messages.tts.auto` ou par session avec `/tts always` (alias : `/tts on`). Edge TTS **est** activé par défaut une fois le TTS activé, et est utilisé automatiquement lorsqu'aucune clé API OpenAI ou ElevenLabs n'est disponible.

## Configuration

La configuration TTS se trouve sous `messages.tts` dans `openclaw.json`. Le schéma complet est dans [Configuration de la passerelle](./gateway/configuration.md).

### Configuration minimale (activation + fournisseur)

```json
{
  messages: {
    tts: {
      auto: "always",
      provider: "elevenlabs",
    },
  },
}
```

### OpenAI principal avec ElevenLabs en secours

```json
{
  messages: {
    tts: {
      auto: "always",
      provider: "openai",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: {
        enabled: true,
      },
      openai: {
        apiKey: "openai_api_key",
        baseUrl: "https://api.openai.com/v1",
        model: "gpt-4o-mini-tts",
        voice: "alloy",
      },
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api.elevenlabs.io",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        languageCode: "en",
        voiceSettings: {
          stability: 0.5,
          similarityBoost: 0.75,
          style: 0.0,
          useSpeakerBoost: true,
          speed: 1.0,
        },
      },
    },
  },
}
```

### Edge TTS principal (pas de clé API)

```json
{
  messages: {
    tts: {
      auto: "always",
      provider: "edge",
      edge: {
        enabled: true,
        voice: "en-US-MichelleNeural",
        lang: "en-US",
        outputFormat: "audio-24khz-48kbitrate-mono-mp3",
        rate: "+10%",
        pitch: "-5%",
      },
    },
  },
}
```

### Désactiver Edge TTS

```json
{
  messages: {
    tts: {
      edge: {
        enabled: false,
      },
    },
  },
}
```

### Limites personnalisées + chemin des préférences

```json
{
  messages: {
    tts: {
      auto: "always",
      maxTextLength: 4000,
      timeoutMs: 30000,
      prefsPath: "~/.openclaw/settings/tts.json",
    },
  },
}
```

### Répondre uniquement avec de l'audio après une note vocale entrante

```json
{
  messages: {
    tts: {
      auto: "inbound",
    },
  },
}
```

### Désactiver le résumé automatique pour les longues réponses

```json
{
  messages: {
    tts: {
      auto: "always",
    },
  },
}
```

Puis exécutez :

```bash
/tts summary off
```

### Notes sur les champs

-   `auto` : mode TTS automatique (`off`, `always`, `inbound`, `tagged`).
    -   `inbound` n'envoie de l'audio qu'après une note vocale entrante.
    -   `tagged` n'envoie de l'audio que lorsque la réponse inclut des balises `[[tts]]`.
-   `enabled` : interrupteur hérité (le docteur migre ceci vers `auto`).
-   `mode` : `"final"` (par défaut) ou `"all"` (inclut les réponses d'outils/blocs).
-   `provider` : `"elevenlabs"`, `"openai"`, ou `"edge"` (le secours est automatique).
-   Si `provider` est **non défini**, OpenClaw préfère `openai` (si clé), puis `elevenlabs` (si clé), sinon `edge`.
-   `summaryModel` : modèle économique optionnel pour le résumé automatique ; par défaut `agents.defaults.model.primary`.
    -   Accepte `provider/model` ou un alias de modèle configuré.
-   `modelOverrides` : permet au modèle d'émettre des directives TTS (activé par défaut).
    -   `allowProvider` est par défaut `false` (le changement de fournisseur est optionnel).
-   `maxTextLength` : limite stricte pour l'entrée TTS (caractères). `/tts audio` échoue si elle est dépassée.
-   `timeoutMs` : délai d'expiration de la requête (ms).
-   `prefsPath` : remplace le chemin JSON des préférences locales (fournisseur/limite/résumé).
-   Les valeurs `apiKey` utilisent les variables d'environnement en secours (`ELEVENLABS_API_KEY`/`XI_API_KEY`, `OPENAI_API_KEY`).
-   `elevenlabs.baseUrl` : remplace l'URL de base de l'API ElevenLabs.
-   `openai.baseUrl` : remplace le point de terminaison TTS OpenAI.
    -   Ordre de résolution : `messages.tts.openai.baseUrl` -> `OPENAI_TTS_BASE_URL` -> `https://api.openai.com/v1`
    -   Les valeurs non par défaut sont traitées comme des points de terminaison TTS compatibles OpenAI, donc les noms de modèles et de voix personnalisés sont acceptés.
-   `elevenlabs.voiceSettings` :
    -   `stability`, `similarityBoost`, `style` : `0..1`
    -   `useSpeakerBoost` : `true|false`
    -   `speed` : `0.5..2.0` (1.0 = normal)
-   `elevenlabs.applyTextNormalization` : `auto|on|off`
-   `elevenlabs.languageCode` : code ISO 639-1 à 2 lettres (ex. `en`, `de`)
-   `elevenlabs.seed` : entier `0..4294967295` (déterminisme au mieux)
-   `edge.enabled` : autorise l'utilisation d'Edge TTS (par défaut `true` ; pas de clé API).
-   `edge.voice` : nom de la voix neuronale Edge (ex. `en-US-MichelleNeural`).
-   `edge.lang` : code de langue (ex. `en-US`).
-   `edge.outputFormat` : format de sortie Edge (ex. `audio-24khz-48kbitrate-mono-mp3`).
    -   Voir les formats de sortie Microsoft Speech pour les valeurs valides ; tous les formats ne sont pas pris en charge par Edge.
-   `edge.rate` / `edge.pitch` / `edge.volume` : chaînes de pourcentage (ex. `+10%`, `-5%`).
-   `edge.saveSubtitles` : écrit des sous-titres JSON à côté du fichier audio.
-   `edge.proxy` : URL du proxy pour les requêtes Edge TTS.
-   `edge.timeoutMs` : remplacement du délai d'expiration de la requête (ms).

## Remplacements pilotés par le modèle (activés par défaut)

Par défaut, le modèle **peut** émettre des directives TTS pour une seule réponse. Lorsque `messages.tts.auto` est `tagged`, ces directives sont nécessaires pour déclencher l'audio. Lorsqu'elles sont activées, le modèle peut émettre des directives `[[tts:...]]` pour remplacer la voix pour une seule réponse, plus un bloc optionnel `[[tts:text]]...[[/tts:text]]` pour fournir des balises expressives (rires, indications de chant, etc.) qui ne doivent apparaître que dans l'audio. Les directives `provider=...` sont ignorées sauf si `modelOverrides.allowProvider: true`. Exemple de charge utile de réponse :

```
Voilà.

[[tts:voiceId=pMsXgVXv3BLzUgSXRplE model=eleven_v3 speed=1.1]]
[[tts:text]](rit) Relis la chanson une fois de plus.[[/tts:text]]
```

Clés de directives disponibles (lorsqu'elles sont activées) :

-   `provider` (`openai` | `elevenlabs` | `edge`, nécessite `allowProvider: true`)
-   `voice` (voix OpenAI) ou `voiceId` (ElevenLabs)
-   `model` (modèle TTS OpenAI ou identifiant de modèle ElevenLabs)
-   `stability`, `similarityBoost`, `style`, `speed`, `useSpeakerBoost`
-   `applyTextNormalization` (`auto|on|off`)
-   `languageCode` (ISO 639-1)
-   `seed`

Désactiver tous les remplacements par le modèle :

```json
{
  messages: {
    tts: {
      modelOverrides: {
        enabled: false,
      },
    },
  },
}
```

Liste d'autorisation optionnelle (activer le changement de fournisseur tout en gardant les autres paramètres configurables) :

```json
{
  messages: {
    tts: {
      modelOverrides: {
        enabled: true,
        allowProvider: true,
        allowSeed: false,
      },
    },
  },
}
```

## Préférences par utilisateur

Les commandes slash écrivent les remplacements locaux dans `prefsPath` (par défaut : `~/.openclaw/settings/tts.json`, remplacez avec `OPENCLAW_TTS_PREFS` ou `messages.tts.prefsPath`). Champs stockés :

-   `enabled`
-   `provider`
-   `maxLength` (seuil de résumé ; par défaut 1500 caractères)
-   `summarize` (par défaut `true`)

Ceux-ci remplacent `messages.tts.*` pour cet hôte.

## Formats de sortie (fixes)

-   **Telegram** : Note vocale Opus (`opus_48000_64` depuis ElevenLabs, `opus` depuis OpenAI).
    -   48 kHz / 64 kbps est un bon compromis pour les notes vocales et requis pour la bulle ronde.
-   **Autres canaux** : MP3 (`mp3_44100_128` depuis ElevenLabs, `mp3` depuis OpenAI).
    -   44,1 kHz / 128 kbps est l'équilibre par défaut pour la clarté de la parole.
-   **Edge TTS** : utilise `edge.outputFormat` (par défaut `audio-24khz-48kbitrate-mono-mp3`).
    -   `node-edge-tts` accepte un `outputFormat`, mais tous les formats ne sont pas disponibles depuis le service Edge. citeturn2search0
    -   Les valeurs de format de sortie suivent les formats de sortie Microsoft Speech (y compris Ogg/WebM Opus). citeturn1search0
    -   `sendVoice` de Telegram accepte OGG/MP3/M4A ; utilisez OpenAI/ElevenLabs si vous avez besoin de notes vocales Opus garanties. citeturn1search1
    -   Si le format de sortie Edge configuré échoue, OpenClaw réessaie avec MP3.

Les formats OpenAI/ElevenLabs sont fixes ; Telegram attend Opus pour l'UX des notes vocales.

## Comportement du TTS automatique

Lorsqu'il est activé, OpenClaw :

-   ignore le TTS si la réponse contient déjà un média ou une directive `MEDIA:`.
-   ignore les réponses très courtes (< 10 caractères).
-   résume les longues réponses lorsqu'activé en utilisant `agents.defaults.model.primary` (ou `summaryModel`).
-   attache l'audio généré à la réponse.

Si la réponse dépasse `maxLength` et que le résumé est désactivé (ou qu'aucune clé API n'est disponible pour le modèle de résumé), l'audio est ignoré et la réponse texte normale est envoyée.

## Diagramme de flux

```
Réponse -> TTS activé ?
  non  -> envoyer le texte
  oui  -> a un média / MEDIA: / court ?
          oui -> envoyer le texte
          non  -> longueur > limite ?
                   non  -> TTS -> attacher l'audio
                   oui  -> résumé activé ?
                            non  -> envoyer le texte
                            oui  -> résumer (summaryModel ou agents.defaults.model.primary)
                                      -> TTS -> attacher l'audio
```

## Utilisation des commandes slash

Il y a une seule commande : `/tts`. Voir [Commandes slash](./tools/slash-commands.md) pour les détails d'activation. Note Discord : `/tts` est une commande intégrée de Discord, donc OpenClaw enregistre `/voice` comme commande native là-bas. Le texte `/tts ...` fonctionne toujours.

```bash
/tts off
/tts always
/tts inbound
/tts tagged
/tts status
/tts provider openai
/tts limit 2000
/tts summary off
/tts audio Hello from OpenClaw
```

Notes :

-   Les commandes nécessitent un expéditeur autorisé (les règles de liste d'autorisation/propriétaire s'appliquent toujours).
-   `commands.text` ou l'enregistrement de commande native doit être activé.
-   `off|always|inbound|tagged` sont des bascules par session (`/tts on` est un alias pour `/tts always`).
-   `limit` et `summary` sont stockés dans les préférences locales, pas dans la configuration principale.
-   `/tts audio` génère une réponse audio ponctuelle (n'active pas le TTS).

## Outil d'agent

L'outil `tts` convertit le texte en parole et renvoie un chemin `MEDIA:`. Lorsque le résultat est compatible avec Telegram, l'outil inclut `[[audio_as_voice]]` pour que Telegram envoie une bulle vocale.

## RPC de la passerelle

Méthodes de la passerelle :

-   `tts.status`
-   `tts.enable`
-   `tts.disable`
-   `tts.convert`
-   `tts.setProvider`
-   `tts.providers`

[Location Command](./nodes/location-command.md)

---