

  Extensions

  
# Plugin Appel vocal

Appels vocaux pour OpenClaw via un plugin. Prend en charge les notifications sortantes et les conversations multi-tours avec des politiques d'appels entrants. Fournisseurs actuels :

-   `twilio` (Programmable Voice + Media Streams)
-   `telnyx` (Call Control v2)
-   `plivo` (Voice API + transfert XML + GetInput speech)
-   `mock` (dev/sans réseau)

Modèle mental rapide :

-   Installer le plugin
-   Redémarrer la Gateway
-   Configurer sous `plugins.entries.voice-call.config`
-   Utiliser `openclaw voicecall ...` ou l'outil `voice_call`

## Où il s'exécute (local vs distant)

Le plugin Appel vocal s'exécute **dans le processus de la Gateway**. Si vous utilisez une Gateway distante, installez/configurez le plugin sur la **machine exécutant la Gateway**, puis redémarrez la Gateway pour le charger.

## Installation

### Option A : installer depuis npm (recommandé)

```bash
openclaw plugins install @openclaw/voice-call
```

Redémarrez la Gateway ensuite.

### Option B : installer depuis un dossier local (dev, sans copie)

```bash
openclaw plugins install ./extensions/voice-call
cd ./extensions/voice-call && pnpm install
```

Redémarrez la Gateway ensuite.

## Configuration

Définissez la configuration sous `plugins.entries.voice-call.config` :

```json
{
  plugins: {
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio", // ou "telnyx" | "plivo" | "mock"
          fromNumber: "+15550001234",
          toNumber: "+15550005678",

          twilio: {
            accountSid: "ACxxxxxxxx",
            authToken: "...",
          },

          telnyx: {
            apiKey: "...",
            connectionId: "...",
            // Clé publique du webhook Telnyx depuis le Telnyx Mission Control Portal
            // (Chaîne Base64 ; peut aussi être définie via TELNYX_PUBLIC_KEY).
            publicKey: "...",
          },

          plivo: {
            authId: "MAxxxxxxxxxxxxxxxxxxxx",
            authToken: "...",
          },

          // Serveur webhook
          serve: {
            port: 3334,
            path: "/voice/webhook",
          },

          // Sécurité webhook (recommandé pour les tunnels/proxies)
          webhookSecurity: {
            allowedHosts: ["voice.example.com"],
            trustedProxyIPs: ["100.64.0.1"],
          },

          // Exposition publique (choisir une)
          // publicUrl: "https://example.ngrok.app/voice/webhook",
          // tunnel: { provider: "ngrok" },
          // tailscale: { mode: "funnel", path: "/voice/webhook" }

          outbound: {
            defaultMode: "notify", // notify | conversation
          },

          streaming: {
            enabled: true,
            streamPath: "/voice/stream",
            preStartTimeoutMs: 5000,
            maxPendingConnections: 32,
            maxPendingConnectionsPerIp: 4,
            maxConnections: 128,
          },
        },
      },
    },
  },
}
```

Notes :

-   Twilio/Telnyx nécessitent une URL de webhook **publiquement accessible**.
-   Plivo nécessite une URL de webhook **publiquement accessible**.
-   `mock` est un fournisseur de développement local (pas d'appels réseau).
-   Telnyx nécessite `telnyx.publicKey` (ou `TELNYX_PUBLIC_KEY`) sauf si `skipSignatureVerification` est vrai.
-   `skipSignatureVerification` est uniquement pour les tests locaux.
-   Si vous utilisez le niveau gratuit de ngrok, définissez `publicUrl` sur l'URL ngrok exacte ; la vérification de signature est toujours appliquée.
-   `tunnel.allowNgrokFreeTierLoopbackBypass: true` autorise les webhooks Twilio avec des signatures invalides **uniquement** lorsque `tunnel.provider="ngrok"` et `serve.bind` est en boucle locale (agent local ngrok). À utiliser uniquement pour le développement local.
-   Les URL du niveau gratuit de ngrok peuvent changer ou ajouter un comportement intermédiaire ; si `publicUrl` dérive, les signatures Twilio échoueront. Pour la production, préférez un domaine stable ou un funnel Tailscale.
-   Sécurité du streaming par défaut :
    -   `streaming.preStartTimeoutMs` ferme les sockets qui n'envoient jamais une trame `start` valide.
    -   `streaming.maxPendingConnections` limite le total des sockets non authentifiés avant démarrage.
    -   `streaming.maxPendingConnectionsPerIp` limite les sockets non authentifiés avant démarrage par IP source.
    -   `streaming.maxConnections` limite le total des sockets de flux média ouverts (en attente + actifs).

## Nettoyeur d'appels obsolètes

Utilisez `staleCallReaperSeconds` pour terminer les appels qui ne reçoivent jamais de webhook terminal (par exemple, les appels en mode notification qui ne se terminent jamais). La valeur par défaut est `0` (désactivé). Plages recommandées :

-   **Production :** `120`–`300` secondes pour les flux de type notification.
-   Gardez cette valeur **supérieure à `maxDurationSeconds`** pour que les appels normaux puissent se terminer. Un bon point de départ est `maxDurationSeconds + 30–60` secondes.

Exemple :

```json
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          maxDurationSeconds: 300,
          staleCallReaperSeconds: 360,
        },
      },
    },
  },
}
```

## Sécurité Webhook

Lorsqu'un proxy ou un tunnel se trouve devant la Gateway, le plugin reconstruit l'URL publique pour la vérification de signature. Ces options contrôlent quels en-têtes transmis sont fiables. `webhookSecurity.allowedHosts` autorise les hôtes provenant des en-têtes de transmission. `webhookSecurity.trustForwardingHeaders` fait confiance aux en-têtes transmis sans liste d'autorisation. `webhookSecurity.trustedProxyIPs` ne fait confiance aux en-têtes transmis que lorsque l'IP distante de la requête correspond à la liste. La protection contre la relecture des webhooks est activée pour Twilio et Plivo. Les requêtes de webhook valides relues sont acquittées mais ignorées pour les effets secondaires. Les tours de conversation Twilio incluent un jeton par tour dans les rappels ``, donc les rappels de parole obsolètes/relus ne peuvent pas satisfaire un tour de transcription plus récent en attente. Exemple avec un hôte public stable :

```json
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          publicUrl: "https://voice.example.com/voice/webhook",
          webhookSecurity: {
            allowedHosts: ["voice.example.com"],
          },
        },
      },
    },
  },
}
```

## TTS pour les appels

Voice Call utilise la configuration principale `messages.tts` (OpenAI ou ElevenLabs) pour la synthèse vocale en streaming sur les appels. Vous pouvez la remplacer dans la configuration du plugin avec la **même structure** — elle fusionne en profondeur avec `messages.tts`.

```json
{
  tts: {
    provider: "elevenlabs",
    elevenlabs: {
      voiceId: "pMsXgVXv3BLzUgSXRplE",
      modelId: "eleven_multilingual_v2",
    },
  },
}
```

Notes :

-   **Edge TTS est ignoré pour les appels vocaux** (l'audio téléphonique nécessite du PCM ; la sortie Edge n'est pas fiable).
-   Le TTS principal est utilisé lorsque le streaming média Twilio est activé ; sinon, les appels reviennent aux voix natives du fournisseur.

### Plus d'exemples

Utiliser uniquement le TTS principal (pas de remplacement) :

```json
{
  messages: {
    tts: {
      provider: "openai",
      openai: { voice: "alloy" },
    },
  },
}
```

Remplacer par ElevenLabs uniquement pour les appels (garder la valeur par défaut principale ailleurs) :

```json
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          tts: {
            provider: "elevenlabs",
            elevenlabs: {
              apiKey: "elevenlabs_key",
              voiceId: "pMsXgVXv3BLzUgSXRplE",
              modelId: "eleven_multilingual_v2",
            },
          },
        },
      },
    },
  },
}
```

Remplacer uniquement le modèle OpenAI pour les appels (exemple de fusion en profondeur) :

```json
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          tts: {
            openai: {
              model: "gpt-4o-mini-tts",
              voice: "marin",
            },
          },
        },
      },
    },
  },
}
```

## Appels entrants

La politique d'appels entrants est par défaut `disabled`. Pour activer les appels entrants, définissez :

```json
{
  inboundPolicy: "allowlist",
  allowFrom: ["+15550001234"],
  inboundGreeting: "Bonjour ! Comment puis-je vous aider ?",
}
```

Les réponses automatiques utilisent le système de l'agent. Ajustez avec :

-   `responseModel`
-   `responseSystemPrompt`
-   `responseTimeoutMs`

## CLI

```bash
openclaw voicecall call --to "+15555550123" --message "Bonjour depuis OpenClaw"
openclaw voicecall continue --call-id <id> --message "Des questions ?"
openclaw voicecall speak --call-id <id> --message "Un instant"
openclaw voicecall end --call-id <id>
openclaw voicecall status --call-id <id>
openclaw voicecall tail
openclaw voicecall expose --mode funnel
```

## Outil Agent

Nom de l'outil : `voice_call` Actions :

-   `initiate_call` (message, to?, mode?)
-   `continue_call` (callId, message)
-   `speak_to_user` (callId, message)
-   `end_call` (callId)
-   `get_status` (callId)

Ce dépôt inclut une documentation de compétence correspondante dans `skills/voice-call/SKILL.md`.

## Gateway RPC

-   `voicecall.initiate` (`to?`, `message`, `mode?`)
-   `voicecall.continue` (`callId`, `message`)
-   `voicecall.speak` (`callId`, `message`)
-   `voicecall.end` (`callId`)
-   `voicecall.status` (`callId`)

[Plugins communautaires](./community.md)[Plugin Zalo Personnel](./zalouser.md)