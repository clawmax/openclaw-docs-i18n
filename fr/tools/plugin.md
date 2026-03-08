title: "Guide des plugins de compétences OpenClaw Installation Configuration et SDK"
description: "Apprenez à installer, configurer et développer des plugins pour OpenClaw. Étendez les fonctionnalités avec des plugins officiels, des routes HTTP et des aides d'exécution."
keywords: ["plugins openclaw", "plugins de compétences", "installation de plugin", "sdk de plugin", "gateway rpc", "outils d'agent", "manifeste de plugin", "aides d'exécution"]
---

  Compétences

  
# Plugins

## Démarrage rapide (nouveau avec les plugins ?)

Un plugin est simplement **un petit module de code** qui étend OpenClaw avec des fonctionnalités supplémentaires (commandes, outils et Gateway RPC). La plupart du temps, vous utiliserez des plugins lorsque vous voudrez une fonctionnalité qui n'est pas encore intégrée au cœur d'OpenClaw (ou que vous voudrez garder des fonctionnalités optionnelles hors de votre installation principale). Voie rapide :

1.  Voir ce qui est déjà chargé :

```bash
openclaw plugins list
```

2.  Installer un plugin officiel (exemple : Appel vocal) :

```bash
openclaw plugins install @openclaw/voice-call
```

Les spécifications npm sont **uniquement pour le registre** (nom du package + optionnellement une **version exacte** ou un **dist-tag**). Les spécifications Git/URL/fichier et les plages semver sont rejetées. Les spécifications nues et `@latest` restent sur la voie stable. Si npm résout l'une d'entre elles en une préversion, OpenClaw s'arrête et vous demande d'accepter explicitement avec un tag de préversion tel que `@beta`/`@rc` ou une version de préversion exacte.

3.  Redémarrer la Gateway, puis configurer sous `plugins.entries..config`.

Voir [Appel vocal](../plugins/voice-call.md) pour un exemple concret de plugin. Vous cherchez des listes tierces ? Voir [Plugins communautaires](../plugins/community.md).

## Plugins disponibles (officiels)

-   Microsoft Teams est uniquement disponible via plugin depuis la version 2026.1.15 ; installez `@openclaw/msteams` si vous utilisez Teams.
-   Mémoire (Core) — plugin de recherche en mémoire intégré (activé par défaut via `plugins.slots.memory`)
-   Mémoire (LanceDB) — plugin de mémoire à long terme intégré (rappel/capture automatique ; définir `plugins.slots.memory = "memory-lancedb"`)
-   [Appel vocal](../plugins/voice-call.md) — `@openclaw/voice-call`
-   [Zalo Personnel](../plugins/zalouser.md) — `@openclaw/zalouser`
-   [Matrix](../channels/matrix.md) — `@openclaw/matrix`
-   [Nostr](../channels/nostr.md) — `@openclaw/nostr`
-   [Zalo](../channels/zalo.md) — `@openclaw/zalo`
-   [Microsoft Teams](../channels/msteams.md) — `@openclaw/msteams`
-   Google Antigravity OAuth (authentification fournisseur) — intégré sous le nom `google-antigravity-auth` (désactivé par défaut)
-   Gemini CLI OAuth (authentification fournisseur) — intégré sous le nom `google-gemini-cli-auth` (désactivé par défaut)
-   Qwen OAuth (authentification fournisseur) — intégré sous le nom `qwen-portal-auth` (désactivé par défaut)
-   Copilot Proxy (authentification fournisseur) — pont proxy Copilot VS Code local ; distinct de la connexion d'appareil intégrée `github-copilot` (intégré, désactivé par défaut)

Les plugins OpenClaw sont des **modules TypeScript** chargés à l'exécution via jiti. **La validation de configuration n'exécute pas le code du plugin** ; elle utilise à la place le manifeste du plugin et JSON Schema. Voir [Manifeste de plugin](../plugins/manifest.md). Les plugins peuvent enregistrer :

-   Méthodes Gateway RPC
-   Routes HTTP Gateway
-   Outils d'agent
-   Commandes CLI
-   Services en arrière-plan
-   Moteurs de contexte
-   Validation de configuration optionnelle
-   **Compétences** (en listant des répertoires `skills` dans le manifeste du plugin)
-   **Commandes de réponse automatique** (s'exécutent sans invoquer l'agent IA)

Les plugins s'exécutent **dans le même processus** que la Gateway, traitez-les donc comme du code de confiance. Guide de création d'outils : [Outils d'agent de plugin](../plugins/agent-tools.md).

## Aides d'exécution

Les plugins peuvent accéder à des aides du cœur sélectionnées via `api.runtime`. Pour la TTS téléphonique :

```typescript
const result = await api.runtime.tts.textToSpeechTelephony({
  text: "Hello from OpenClaw",
  cfg: api.config,
});
```

Notes :

-   Utilise la configuration `messages.tts` du cœur (OpenAI ou ElevenLabs).
-   Renvoie un tampon audio PCM + fréquence d'échantillonnage. Les plugins doivent rééchantillonner/encoder pour les fournisseurs.
-   Edge TTS n'est pas pris en charge pour la téléphonie.

Pour la STT/transcription, les plugins peuvent appeler :

```typescript
const { text } = await api.runtime.stt.transcribeAudioFile({
  filePath: "/tmp/inbound-audio.ogg",
  cfg: api.config,
  // Optionnel lorsque le type MIME ne peut pas être déduit de manière fiable :
  mime: "audio/ogg",
});
```

Notes :

-   Utilise la configuration audio de compréhension multimédia du cœur (`tools.media.audio`) et l'ordre de repli des fournisseurs.
-   Renvoie `{ text: undefined }` lorsqu'aucune transcription n'est produite (par exemple entrée ignorée/non prise en charge).

## Routes HTTP Gateway

Les plugins peuvent exposer des points de terminaison HTTP avec `api.registerHttpRoute(...)`.

```
api.registerHttpRoute({
  path: "/acme/webhook",
  auth: "plugin",
  match: "exact",
  handler: async (_req, res) => {
    res.statusCode = 200;
    res.end("ok");
    return true;
  },
});
```

Champs de la route :

-   `path` : chemin de la route sous le serveur HTTP de la gateway.
-   `auth` : requis. Utilisez `"gateway"` pour exiger l'authentification normale de la gateway, ou `"plugin"` pour l'authentification/gestion de webhook par le plugin.
-   `match` : optionnel. `"exact"` (par défaut) ou `"prefix"`.
-   `replaceExisting` : optionnel. Permet au même plugin de remplacer son propre enregistrement de route existant.
-   `handler` : renvoie `true` lorsque la route a traité la requête.

Notes :

-   `api.registerHttpHandler(...)` est obsolète. Utilisez `api.registerHttpRoute(...)`.
-   Les routes de plugin doivent déclarer explicitement `auth`.
-   Les conflits `path + match` exacts sont rejetés sauf si `replaceExisting: true`, et un plugin ne peut pas remplacer la route d'un autre plugin.
-   Les routes qui se chevauchent avec des niveaux `auth` différents sont rejetées. Gardez les chaînes de repli `exact`/`prefix` uniquement au même niveau d'authentification.

## Chemins d'importation du SDK de plugin

Utilisez les sous-chemins du SDK au lieu de l'import monolithique `openclaw/plugin-sdk` lors de la création de plugins :

-   `openclaw/plugin-sdk/core` pour les API de plugin génériques, les types d'authentification de fournisseur et les aides partagées.
-   `openclaw/plugin-sdk/compat` pour le code de plugin intégré/interne nécessitant des aides d'exécution partagées plus larges que `core`.
-   `openclaw/plugin-sdk/telegram` pour les plugins de canal Telegram.
-   `openclaw/plugin-sdk/discord` pour les plugins de canal Discord.
-   `openclaw/plugin-sdk/slack` pour les plugins de canal Slack.
-   `openclaw/plugin-sdk/signal` pour les plugins de canal Signal.
-   `openclaw/plugin-sdk/imessage` pour les plugins de canal iMessage.
-   `openclaw/plugin-sdk/whatsapp` pour les plugins de canal WhatsApp.
-   `openclaw/plugin-sdk/line` pour les plugins de canal LINE.
-   `openclaw/plugin-sdk/msteams` pour la surface du plugin Microsoft Teams intégré.
-   Les sous-chemins spécifiques aux extensions intégrées sont également disponibles : `openclaw/plugin-sdk/acpx`, `openclaw/plugin-sdk/bluebubbles`, `openclaw/plugin-sdk/copilot-proxy`, `openclaw/plugin-sdk/device-pair`, `openclaw/plugin-sdk/diagnostics-otel`, `openclaw/plugin-sdk/diffs`, `openclaw/plugin-sdk/feishu`, `openclaw/plugin-sdk/google-gemini-cli-auth`, `openclaw/plugin-sdk/googlechat`, `openclaw/plugin-sdk/irc`, `openclaw/plugin-sdk/llm-task`, `openclaw/plugin-sdk/lobster`, `openclaw/plugin-sdk/matrix`, `openclaw/plugin-sdk/mattermost`, `openclaw/plugin-sdk/memory-core`, `openclaw/plugin-sdk/memory-lancedb`, `openclaw/plugin-sdk/minimax-portal-auth`, `openclaw/plugin-sdk/nextcloud-talk`, `openclaw/plugin-sdk/nostr`, `openclaw/plugin-sdk/open-prose`, `openclaw/plugin-sdk/phone-control`, `openclaw/plugin-sdk/qwen-portal-auth`, `openclaw/plugin-sdk/synology-chat`, `openclaw/plugin-sdk/talk-voice`, `openclaw/plugin-sdk/test-utils`, `openclaw/plugin-sdk/thread-ownership`, `openclaw/plugin-sdk/tlon`, `openclaw/plugin-sdk/twitch`, `openclaw/plugin-sdk/voice-call`, `openclaw/plugin-sdk/zalo`, et `openclaw/plugin-sdk/zalouser`.

Note de compatibilité :

-   `openclaw/plugin-sdk` reste pris en charge pour les plugins externes existants.
-   Les nouveaux plugins intégrés et ceux migrés doivent utiliser les sous-chemins spécifiques aux canaux ou extensions ; utilisez `core` pour les surfaces génériques et `compat` uniquement lorsque des aides partagées plus larges sont requises.

## Inspection de canal en lecture seule

Si votre plugin enregistre un canal, préférez implémenter `plugin.config.inspectAccount(cfg, accountId)` aux côtés de `resolveAccount(...)`. Pourquoi :

-   `resolveAccount(...)` est le chemin d'exécution. Il est autorisé à supposer que les identifiants sont entièrement matérialisés et peut échouer rapidement lorsque les secrets requis sont manquants.
-   Les chemins de commande en lecture seule tels que `openclaw status`, `openclaw status --all`, `openclaw channels status`, `openclaw channels resolve`, et les flux de réparation doctor/config ne devraient pas avoir besoin de matérialiser les identifiants d'exécution juste pour décrire la configuration.

Comportement recommandé pour `inspectAccount(...)` :

-   Renvoyer uniquement l'état descriptif du compte.
-   Préserver `enabled` et `configured`.
-   Inclure les champs de source/statut des identifiants lorsque pertinent, tels que :
    -   `tokenSource`, `tokenStatus`
    -   `botTokenSource`, `botTokenStatus`
    -   `appTokenSource`, `appTokenStatus`
    -   `signingSecretSource`, `signingSecretStatus`
-   Vous n'avez pas besoin de renvoyer les valeurs brutes des jetons juste pour signaler la disponibilité en lecture seule. Renvoyer `tokenStatus: "available"` (et le champ source correspondant) suffit pour les commandes de type status.
-   Utilisez `configured_unavailable` lorsqu'un identifiant est configuré via SecretRef mais indisponible dans le chemin de commande actuel.

Cela permet aux commandes en lecture seule de signaler "configuré mais indisponible dans ce chemin de commande" au lieu de planter ou de signaler incorrectement le compte comme non configuré. Note de performance :

-   La découverte de plugins et les métadonnées du manifeste utilisent de petits caches en processus pour réduire le travail de démarrage/rechargement par rafales.
-   Définissez `OPENCLAW_DISABLE_PLUGIN_DISCOVERY_CACHE=1` ou `OPENCLAW_DISABLE_PLUGIN_MANIFEST_CACHE=1` pour désactiver ces caches.
-   Ajustez les fenêtres de cache avec `OPENCLAW_PLUGIN_DISCOVERY_CACHE_MS` et `OPENCLAW_PLUGIN_MANIFEST_CACHE_MS`.

## Découverte & précédence

OpenClaw scanne, dans l'ordre :

1.  Chemins de configuration

-   `plugins.load.paths` (fichier ou répertoire)

2.  Extensions d'espace de travail

-   `/.openclaw/extensions/*.ts`
-   `/.openclaw/extensions/*/index.ts`

3.  Extensions globales

-   `~/.openclaw/extensions/*.ts`
-   `~/.openclaw/extensions/*/index.ts`

4.  Extensions intégrées (livrées avec OpenClaw, principalement désactivées par défaut)

-   `/extensions/*`

La plupart des plugins intégrés doivent être activés explicitement via `plugins.entries..enabled` ou `openclaw plugins enable `. Exceptions de plugins intégrés activés par défaut :

-   `device-pair`
-   `phone-control`
-   `talk-voice`
-   plugin de slot de mémoire actif (slot par défaut : `memory-core`)

Les plugins installés sont activés par défaut, mais peuvent être désactivés de la même manière. Notes de durcissement :

-   Si `plugins.allow` est vide et que des plugins non intégrés sont détectables, OpenClaw enregistre un avertissement de démarrage avec les ids et sources des plugins.
-   Les chemins candidats sont vérifiés pour la sécurité avant l'admission à la découverte. OpenClaw bloque les candidats lorsque :
    -   l'entrée d'extension se résout en dehors de la racine du plugin (y compris les échappements de lien symbolique/traversal de chemin),
    -   le chemin racine/source du plugin est accessible en écriture pour tous,
    -   la propriété du chemin est suspecte pour les plugins non intégrés (le propriétaire POSIX n'est ni l'uid actuel ni root).
-   Les plugins non intégrés chargés sans provenance d'installation/chemin de chargement émettent un avertissement pour que vous puissiez figer la confiance (`plugins.allow`) ou le suivi d'installation (`plugins.installs`).

Chaque plugin doit inclure un fichier `openclaw.plugin.json` dans sa racine. Si un chemin pointe vers un fichier, la racine du plugin est le répertoire du fichier et doit contenir le manifeste. Si plusieurs plugins se résolvent vers le même id, la première correspondance dans l'ordre ci-dessus l'emporte et les copies de précédence inférieure sont ignorées.

### Packs de packages

Un répertoire de plugin peut inclure un `package.json` avec `openclaw.extensions` :

```json
{
  "name": "my-pack",
  "openclaw": {
    "extensions": ["./src/safety.ts", "./src/tools.ts"]
  }
}
```

Chaque entrée devient un plugin. Si le pack liste plusieurs extensions, l'id du plugin devient `name/`. Si votre plugin importe des dépendances npm, installez-les dans ce répertoire pour que `node_modules` soit disponible (`npm install` / `pnpm install`). Garde-fou de sécurité : chaque entrée `openclaw.extensions` doit rester à l'intérieur du répertoire du plugin après résolution des liens symboliques. Les entrées qui s'échappent du répertoire du package sont rejetées. Note de sécurité : `openclaw plugins install` installe les dépendances du plugin avec `npm install --ignore-scripts` (pas de scripts de cycle de vie). Gardez les arbres de dépendances des plugins "purs JS/TS" et évitez les packages qui nécessitent des builds `postinstall`.

### Métadonnées du catalogue de canaux

Les plugins de canal peuvent publier des métadonnées d'intégration via `openclaw.channel` et des conseils d'installation via `openclaw.install`. Cela garde les données du catalogue principal sans dépendances. Exemple :

```json
{
  "name": "@openclaw/nextcloud-talk",
  "openclaw": {
    "extensions": ["./index.ts"],
    "channel": {
      "id": "nextcloud-talk",
      "label": "Nextcloud Talk",
      "selectionLabel": "Nextcloud Talk (auto-hébergé)",
      "docsPath": "/channels/nextcloud-talk",
      "docsLabel": "nextcloud-talk",
      "blurb": "Chat auto-hébergé via bots webhook Nextcloud Talk.",
      "order": 65,
      "aliases": ["nc-talk", "nc"]
    },
    "install": {
      "npmSpec": "@openclaw/nextcloud-talk",
      "localPath": "extensions/nextcloud-talk",
      "defaultChoice": "npm"
    }
  }
}
```

OpenClaw peut également fusionner **des catalogues de canaux externes** (par exemple, une exportation de registre MPM). Déposez un fichier JSON à l'un des emplacements suivants :

-   `~/.openclaw/mpm/plugins.json`
-   `~/.openclaw/mpm/catalog.json`
-   `~/.openclaw/plugins/catalog.json`

Ou pointez `OPENCLAW_PLUGIN_CATALOG_PATHS` (ou `OPENCLAW_MPM_CATALOG_PATHS`) vers un ou plusieurs fichiers JSON (séparés par virgule/point-virgule/`PATH`). Chaque fichier doit contenir `{ "entries": [ { "name": "@scope/pkg", "openclaw": { "channel": {...}, "install": {...} } } ] }`.

## IDs de plugin

IDs de plugin par défaut :

-   Packs de packages : `name` du `package.json`
-   Fichier autonome : nom de base du fichier (`~/.../voice-call.ts` → `voice-call`)

Si un plugin exporte `id`, OpenClaw l'utilise mais avertit lorsqu'il ne correspond pas à l'id configuré.

## Configuration

```json
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    deny: ["untrusted-plugin"],
    load: { paths: ["~/Projects/oss/voice-call-extension"] },
    entries: {
      "voice-call": { enabled: true, config: { provider: "twilio" } },
    },
  },
}
```

Champs :

-   `enabled` : interrupteur principal (par défaut : true)
-   `allow` : liste d'autorisation (optionnelle)
-   `deny` : liste de refus (optionnelle ; le refus l'emporte)
-   `load.paths` : fichiers/répertoires de plugin supplémentaires
-   `slots` : sélecteurs de slots exclusifs tels que `memory` et `contextEngine`
-   `entries.` : interrupteurs par plugin + configuration

Les changements de configuration **nécessitent un redémarrage de la gateway**. Règles de validation (strictes) :

-   Les ids de plugin inconnus dans `entries`, `allow`, `deny`, ou `slots` sont des **erreurs**.
-   Les clés `channels.` inconnues sont des **erreurs** sauf si un manifeste de plugin déclare l'id du canal.
-   La configuration du plugin est validée en utilisant le JSON Schema intégré dans `openclaw.plugin.json` (`configSchema`).
-   Si un plugin est désactivé, sa configuration est préservée et un **avertissement** est émis.

## Slots de plugin (catégories exclusives)

Certaines catégories de plugins sont **exclusives** (un seul actif à la fois). Utilisez `plugins.slots` pour sélectionner quel plugin possède le slot :

```json
{
  plugins: {
    slots: {
      memory: "memory-core", // ou "none" pour désactiver les plugins de mémoire
      contextEngine: "legacy", // ou un id de plugin tel que "lossless-claw"
    },
  },
}
```

Slots exclusifs pris en charge :

-   `memory` : plugin de mémoire actif (`"none"` désactive les plugins de mémoire)
-   `contextEngine` : plugin de moteur de contexte actif (`"legacy"` est la valeur intégrée par défaut)

Si plusieurs plugins déclarent `kind: "memory"` ou `kind: "context-engine"`, seul le plugin sélectionné se charge pour ce slot. Les autres sont désactivés avec des diagnostics.

### Plugins de moteur de contexte

Les plugins de moteur de contexte possèdent l'orchestration du contexte de session pour l'ingestion, l'assemblage et la compaction. Enregistrez-les depuis votre plugin avec `api.registerContextEngine(id, factory)`, puis sélectionnez le moteur actif avec `plugins.slots.contextEngine`. Utilisez ceci lorsque votre plugin doit remplacer ou étendre le pipeline de contexte par défaut plutôt que simplement ajouter une recherche en mémoire ou des hooks.

## Interface de contrôle (schéma + étiquettes)

L'interface de contrôle utilise `config.schema` (JSON Schema + `uiHints`) pour afficher de meilleurs formulaires. OpenClaw enrichit `uiHints` à l'exécution en fonction des plugins détectés :

-   Ajoute des étiquettes par plugin pour `plugins.entries.` / `.enabled` / `.config`
-   Fusionne les conseils de champs de configuration optionnels fournis par le plugin sous : `plugins.entries..config.`

Si vous voulez que les champs de configuration de votre plugin affichent de bonnes étiquettes/placeholders (et marquent les secrets comme sensibles), fournissez `uiHints` à côté de votre JSON Schema dans le manifeste du plugin. Exemple :

```json
{
  "id": "my-plugin",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {
      "apiKey": { "type": "string" },
      "region": { "type": "string" }
    }
  },
  "uiHints": {
    "apiKey": { "label": "Clé API", "sensitive": true },
    "region": { "label": "Région", "placeholder": "us-east-1" }
  }
}
```

## CLI

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins install <path>                 # copie un fichier/répertoire local dans ~/.openclaw/extensions/<id>
openclaw plugins install ./extensions/voice-call # chemin relatif ok
openclaw plugins install ./plugin.tgz           # installe depuis une archive tar locale
openclaw plugins install ./plugin.zip           # installe depuis une archive zip locale
openclaw plugins install -l ./extensions/voice-call # lien (pas de copie) pour le développement
openclaw plugins install @openclaw/voice-call # installe depuis npm
openclaw plugins install @openclaw/voice-call --pin # stocke le nom@version exact résolu
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
```

`plugins update` ne fonctionne que pour les installations npm suivies sous `plugins.installs`. Si les métadonnées d'intégrité stockées changent entre les mises à jour, OpenClaw avertit et demande confirmation (utilisez le `--yes` global pour contourner les invites). Les plugins peuvent également enregistrer leurs propres commandes de premier niveau (exemple : `openclaw voicecall`).

## API de plugin (aperçu)

Les plugins exportent soit :

-   Une fonction : `(api) => { ... }`
-   Un objet : `{ id, name, configSchema, register(api) { ... } }`

Les plugins de moteur de contexte peuvent également enregistrer un gestionnaire de contexte détenu par l'exécution :

```bash
export default function (api) {
  api.registerContextEngine("lossless-claw", () => ({
    info: { id: "lossless-claw", name: "Lossless Claw", ownsCompaction: true },
    async ingest() {
      return { ingested: true };
    },
    async assemble({ messages }) {
      return { messages, estimatedTokens: 0 };
    },
    async compact() {
      return { ok: true, compacted: false };
    },
  }));
}
```

Puis activez-le dans la configuration :

```json
{
  plugins: {
    slots: {
      contextEngine: "lossless-claw",
    },
  },
}
```

## Hooks de plugin

Les plugins peuvent enregistrer des hooks à l'exécution. Cela permet à un plugin d'embarquer une automatisation pilotée par événements sans installation séparée d'un pack de hooks.

### Exemple

```bash
export default function register(api) {
  api.registerHook(
    "command:new",
    async () => {
      // Logique du hook ici.
    },
    {
      name: "my-plugin.command-new",
      description: "S'exécute lorsque /new est invoqué",
    },
  );
}
```

Notes :

-   Enregistrez les hooks explicitement via `api.registerHook(...)`.
-   Les règles d'éligibilité des hooks s'appliquent toujours (exigences OS/binaires/env/config).
-   Les hooks gérés par le plugin apparaissent dans `openclaw hooks list` avec `plugin:`.
-   Vous ne pouvez pas activer/désactiver les hooks gérés par le plugin via `openclaw hooks` ; activez/désactivez le plugin à la place.

### Hooks de cycle de vie de l'agent (api.on)

Pour les hooks de cycle de vie d'exécution typés, utilisez `api.on(...)` :

```bash
export default function register(api) {
  api.on(
    "before_prompt_build",
    (event, ctx) => {
      return {
        prependSystemContext: "Suivez le guide de style de l'entreprise.",
      };
    },
    { priority: 10 },
  );
}
```

Hooks importants pour la construction de prompt :

-   `before_model_resolve` : s'exécute avant le chargement de la session (`messages` ne sont pas disponibles). Utilisez ceci pour remplacer de manière déterministe `modelOverride` ou `providerOverride`.
-   `before_prompt_build` : s'exécute après le chargement de la session (`messages` sont disponibles). Utilisez ceci pour façonner l'entrée du prompt.
-   `before_agent_start` : hook de compatibilité hérité. Préférez les deux hooks explicites ci-dessus.

Politique de hook appliquée par le cœur :

-   Les opérateurs peuvent désactiver les hooks de mutation de prompt par plugin via `plugins.entries..hooks.allowPromptInjection: false`.
-   Lorsqu'il est désactivé, OpenClaw bloque `before_prompt_build` et ignore les champs de mutation de prompt renvoyés par le hook hérité `before_agent_start` tout en préservant les `modelOverride` et `providerOverride` hérités.

Champs de résultat de `before_prompt_build` :

-   `prependContext` : ajoute du texte au prompt utilisateur pour cette exécution. Idéal pour le contenu spécifique au tour ou dynamique.
-   `systemPrompt` : remplacement complet du prompt système.
-   `prependSystemContext` : ajoute du texte au prompt système actuel.
-   `appendSystemContext` : ajoute du texte au prompt système actuel.

Ordre de construction du prompt dans l'exécution embarquée :

1.  Appliquer `prependContext` au prompt utilisateur.
2.  Appliquer le remplacement `systemPrompt` lorsqu'il est fourni.
3.  Appliquer `prependSystemContext + prompt système actuel + appendSystemContext`.

Notes de fusion et de précédence :

-   Les gestionnaires de hooks s'exécutent par priorité (plus élevée d'abord).
-   Pour les champs de contexte fusionnés, les valeurs sont concaténées dans l'ordre d'exécution.
-   Les valeurs de `before_prompt_build` sont appliquées avant les valeurs de repli du hook hérité `before_agent_start`.

Conseils de migration :

-   Déplacez les instructions statiques de `prependContext` vers `prependSystemContext` (ou `appendSystemContext`) pour que les fournisseurs puissent mettre en cache le contenu stable du préfixe système.
-   Gardez `prependContext` pour le contexte dynamique par tour qui doit rester lié au message utilisateur.

## Plugins de fournisseur (authentification de modèle)

Les plugins peuvent enregistrer des flux **d'authentification de fournisseur de modèle** pour que les utilisateurs puissent exécuter la configuration OAuth ou de clé API dans OpenClaw (pas besoin de scripts externes). Enregistrez un fournisseur via `api.registerProvider(...)`. Chaque fournisseur expose une ou plusieurs méthodes d'authentification (OAuth, clé API, code d'appareil, etc.). Ces méthodes alimentent :

-   `openclaw models auth login --provider  [--method ]`

Exemple :

```
api.registerProvider({
  id: "acme",
  label: "AcmeAI",
  auth: [
    {
      id: "oauth",
      label: "OAuth",
      kind: "oauth",
      run: async (ctx) => {
        // Exécute le flux OAuth et renvoie les profils d'authentification.
        return {
          profiles: [
            {
              profileId: "acme:default",
              credential: {
                type: "oauth",
                provider: "acme",
                access: "...",
                refresh: "...",
                expires: Date.now() + 3600 * 1000,
              },
            },
          ],
          defaultModel: "acme/opus-1",
        };
      },
    },
  ],
});
```

Notes :

-   `run` reçoit un `ProviderAuthContext` avec des aides `prompter`, `runtime`, `openUrl`, et `oauth.createVpsAwareHandlers`.
-   Renvoyez `configPatch` lorsque vous devez ajouter des modèles par défaut ou une configuration de fournisseur.
-   Renvoyez `defaultModel` pour que `--set-default` puisse mettre à jour les valeurs par défaut de l'agent.

### Enregistrer un canal de messagerie

Les plugins peuvent enregistrer **des plugins de canal** qui se comportent comme des canaux intégrés (WhatsApp, Telegram, etc.). La configuration du canal se trouve sous `channels.` et est validée par votre code de plugin de canal.

```typescript
const myChannel = {
  id: "acmechat",
  meta: {
    id: "acmechat",
    label: "AcmeChat",
    selectionLabel: "AcmeChat (API)",
    docsPath: "/channels/acmechat",
    blurb: "plugin de canal de démonstration.",
    aliases: ["acme"],
  },
  capabilities: { chatTypes: ["direct"] },
  config: {
    listAccountIds: (cfg) => Object.keys(cfg.channels?.acmechat?.accounts ?? {}),
    resolveAccount: (cfg, accountId) =>
      cfg.channels?.acmechat?.accounts?.[accountId ?? "default"] ?? {
        accountId,
      },
  },
  outbound: {
    deliveryMode: "direct",
    sendText: async () => ({ ok: true }),
  },
};

export default function (api) {
  api.registerChannel({ plugin: myChannel });
}
```

Notes :

-   Placez la configuration sous `channels.` (pas `plugins.entries`).
-   `meta.label` est utilisé pour les étiquettes dans les listes CLI/UI.
-   `meta.aliases` ajoute des ids alternatifs pour la normalisation et les entrées CLI.
-   `meta.preferOver` liste les ids de canal à ignorer pour l'activation automatique lorsque les deux sont configurés.
-   `meta.detailLabel` et `meta.systemImage` permettent aux interfaces d'afficher des étiquettes/icônes de canal plus riches.

### Hooks d'intégration de canal

Les plugins de canal peuvent définir des hooks d'intégration optionnels sur `plugin.onboarding` :

-   `configure(ctx)` est le flux de configuration de base.
-   `configureInteractive(ctx)` peut entièrement posséder la configuration interactive pour les états configurés et non configurés.
-   `configureWhenConfigured(ctx)` peut remplacer le comportement uniquement pour les canaux déjà configurés.

Précédence des hooks dans l'assistant :

1.  `configureInteractive` (si présent)
2.  `configureWhenConfigured` (uniquement lorsque le statut du canal est déjà configuré)
3.  repli sur `configure`

Détails du contexte :

-   `configureInteractive` et `configureWhenConfigured` reçoivent :
    -   `configured` (`true` ou `false`)
    -   `label` (nom du canal côté utilisateur utilisé par les invites)
    -   plus les champs partagés config/runtime/prompter/options
-   Renvoyer `"skip"` laisse la