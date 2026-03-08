title: "Guide d'installation et de configuration du navigateur géré OpenClaw"
description: "Apprenez à configurer et utiliser le navigateur géré OpenClaw pour une automatisation d'agent sûre et isolée. Inclut les commandes de démarrage rapide, la gestion des profils et les bonnes pratiques de sécurité."
keywords: ["navigateur openclaw", "navigateur géré", "automatisation navigateur", "configuration cdp", "profils navigateur", "automatisation agent", "intégration browserless", "contrôle navigateur distant"]
---

  Navigateur

  
# Navigateur (géré par OpenClaw)

OpenClaw peut exécuter un **profil Chrome/Brave/Edge/Chromium dédié** que l'agent contrôle. Il est isolé de votre navigateur personnel et est géré via un petit service de contrôle local à l'intérieur de la Gateway (boucle locale uniquement). Vue pour débutant :

-   Considérez-le comme un **navigateur séparé, réservé à l'agent**.
-   Le profil `openclaw` **ne touche pas** à votre profil de navigateur personnel.
-   L'agent peut **ouvrir des onglets, lire des pages, cliquer et taper** dans un couloir sécurisé.
-   Le profil par défaut `chrome` utilise le **navigateur Chromium par défaut du système** via le relais d'extension ; passez à `openclaw` pour le navigateur géré isolé.

## Ce que vous obtenez

-   Un profil de navigateur séparé nommé **openclaw** (accent orange par défaut).
-   Contrôle déterministe des onglets (lister/ouvrir/focus/fermer).
-   Actions de l'agent (cliquer/taper/glisser/sélectionner), instantanés, captures d'écran, PDF.
-   Prise en charge optionnelle de plusieurs profils (`openclaw`, `work`, `remote`, …).

Ce navigateur **n'est pas** votre navigateur quotidien. C'est une surface sûre et isolée pour l'automatisation et la vérification par l'agent.

## Démarrage rapide

```bash
openclaw browser --browser-profile openclaw status
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

Si vous obtenez "Browser disabled", activez-le dans la configuration (voir ci-dessous) et redémarrez la Gateway.

## Profils : openclaw vs chrome

-   `openclaw` : navigateur géré, isolé (aucune extension requise).
-   `chrome` : relais d'extension vers votre **navigateur système** (nécessite que l'extension OpenClaw soit attachée à un onglet).

Définissez `browser.defaultProfile: "openclaw"` si vous voulez le mode géré par défaut.

## Configuration

Les paramètres du navigateur se trouvent dans `~/.openclaw/openclaw.json`.

```json
{
  browser: {
    enabled: true, // défaut : true
    ssrfPolicy: {
      dangerouslyAllowPrivateNetwork: true, // mode réseau de confiance par défaut
      // allowPrivateNetwork: true, // alias hérité
      // hostnameAllowlist: ["*.example.com", "example.com"],
      // allowedHostnames: ["localhost"],
    },
    // cdpUrl: "http://127.0.0.1:18792", // remplacement hérité pour un seul profil
    remoteCdpTimeoutMs: 1500, // timeout HTTP CDP distant (ms)
    remoteCdpHandshakeTimeoutMs: 3000, // timeout de poignée de main WebSocket CDP distant (ms)
    defaultProfile: "chrome",
    color: "#FF4500",
    headless: false,
    noSandbox: false,
    attachOnly: false,
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" },
    },
  },
}
```

Notes :

-   Le service de contrôle du navigateur se lie en boucle locale sur un port dérivé de `gateway.port` (défaut : `18791`, qui est gateway + 2). Le relais utilise le port suivant (`18792`).
-   Si vous remplacez le port de la Gateway (`gateway.port` ou `OPENCLAW_GATEWAY_PORT`), les ports dérivés du navigateur se décalent pour rester dans la même "famille".
-   `cdpUrl` utilise par défaut le port du relais s'il n'est pas défini.
-   `remoteCdpTimeoutMs` s'applique aux vérifications d'accessibilité CDP distantes (non boucle locale).
-   `remoteCdpHandshakeTimeoutMs` s'applique aux vérifications d'accessibilité WebSocket CDP distantes.
-   La navigation/ouverture d'onglet du navigateur est protégée contre les SSRF avant la navigation et vérifiée au mieux sur l'URL `http(s)` finale après la navigation.
-   `browser.ssrfPolicy.dangerouslyAllowPrivateNetwork` est par défaut `true` (modèle réseau de confiance). Définissez-le à `false` pour une navigation strictement publique.
-   `browser.ssrfPolicy.allowPrivateNetwork` reste pris en charge comme alias hérité pour la compatibilité.
-   `attachOnly: true` signifie "ne jamais lancer un navigateur local ; seulement s'attacher s'il est déjà en cours d'exécution."
-   `color` + `color` par profil teintent l'interface utilisateur du navigateur pour que vous puissiez voir quel profil est actif.
-   Le profil par défaut est `openclaw` (navigateur autonome géré par OpenClaw). Utilisez `defaultProfile: "chrome"` pour opter pour le relais d'extension Chrome.
-   Ordre de détection automatique : navigateur par défaut du système s'il est basé sur Chromium ; sinon Chrome → Brave → Edge → Chromium → Chrome Canary.
-   Les profils locaux `openclaw` attribuent automatiquement `cdpPort`/`cdpUrl` — ne définissez ceux-ci que pour le CDP distant.

## Utiliser Brave (ou un autre navigateur basé sur Chromium)

Si votre **navigateur par défaut système** est basé sur Chromium (Chrome/Brave/Edge/etc), OpenClaw l'utilise automatiquement. Définissez `browser.executablePath` pour remplacer la détection automatique : exemple CLI :

```bash
openclaw config set browser.executablePath "/usr/bin/google-chrome"
```

```
// macOS
{
  browser: {
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser"
  }
}

// Windows
{
  browser: {
    executablePath: "C:\\Program Files\\BraveSoftware\\Brave-Browser\\Application\\brave.exe"
  }
}

// Linux
{
  browser: {
    executablePath: "/usr/bin/brave-browser"
  }
}
```

## Contrôle local vs distant

-   **Contrôle local (défaut) :** la Gateway démarre le service de contrôle en boucle locale et peut lancer un navigateur local.
-   **Contrôle distant (hôte nœud) :** exécutez un hôte nœud sur la machine qui a le navigateur ; la Gateway proxy les actions du navigateur vers lui.
-   **CDP distant :** définissez `browser.profiles..cdpUrl` (ou `browser.cdpUrl`) pour s'attacher à un navigateur distant basé sur Chromium. Dans ce cas, OpenClaw ne lancera pas de navigateur local.

Les URL CDP distantes peuvent inclure une authentification :

-   Tokens de requête (par exemple, `https://provider.example?token=`)
-   Authentification HTTP Basic (par exemple, `https://user:pass@provider.example`)

OpenClaw préserve l'authentification lors de l'appel des endpoints `/json/*` et lors de la connexion au WebSocket CDP. Préférez les variables d'environnement ou les gestionnaires de secrets pour les tokens plutôt que de les commettre dans les fichiers de configuration.

## Proxy navigateur nœud (défaut sans configuration)

Si vous exécutez un **hôte nœud** sur la machine qui a votre navigateur, OpenClaw peut acheminer automatiquement les appels d'outils de navigateur vers ce nœud sans configuration supplémentaire. C'est le chemin par défaut pour les gateways distantes. Notes :

-   L'hôte nœud expose son serveur de contrôle de navigateur local via une **commande proxy**.
-   Les profils proviennent de la propre configuration `browser.profiles` du nœud (identique au local).
-   Désactivez si vous n'en voulez pas :
    -   Sur le nœud : `nodeHost.browserProxy.enabled=false`
    -   Sur la gateway : `gateway.nodes.browser.mode="off"`

## Browserless (CDP distant hébergé)

[Browserless](https://browserless.io) est un service Chromium hébergé qui expose des endpoints CDP via HTTPS. Vous pouvez pointer un profil de navigateur OpenClaw vers un endpoint de région Browserless et vous authentifier avec votre clé API. Exemple :

```json
{
  browser: {
    enabled: true,
    defaultProfile: "browserless",
    remoteCdpTimeoutMs: 2000,
    remoteCdpHandshakeTimeoutMs: 4000,
    profiles: {
      browserless: {
        cdpUrl: "https://production-sfo.browserless.io?token=<BROWSERLESS_API_KEY>",
        color: "#00AA00",
      },
    },
  },
}
```

Notes :

-   Remplacez `<BROWSERLESS_API_KEY>` par votre véritable token Browserless.
-   Choisissez l'endpoint de région qui correspond à votre compte Browserless (voir leur documentation).

## Sécurité

Idées clés :

-   Le contrôle du navigateur est en boucle locale uniquement ; l'accès passe par l'authentification de la Gateway ou l'appariement de nœuds.
-   Si le contrôle du navigateur est activé et qu'aucune authentification n'est configurée, OpenClaw génère automatiquement `gateway.auth.token` au démarrage et le persiste dans la configuration.
-   Gardez la Gateway et tout hôte nœud sur un réseau privé (Tailscale) ; évitez l'exposition publique.
-   Traitez les URL/tokens CDP distants comme des secrets ; préférez les variables d'environnement ou un gestionnaire de secrets.

Conseils pour le CDP distant :

-   Préférez les endpoints HTTPS et les tokens à courte durée de vie lorsque c'est possible.
-   Évitez d'intégrer des tokens de longue durée directement dans les fichiers de configuration.

## Profils (multi-navigateurs)

OpenClaw prend en charge plusieurs profils nommés (configurations de routage). Les profils peuvent être :

-   **Géré par openclaw** : une instance de navigateur basée sur Chromium dédiée avec son propre répertoire de données utilisateur + port CDP
-   **Distant** : une URL CDP explicite (navigateur basé sur Chromium exécuté ailleurs)
-   **Relais d'extension** : vos onglets Chrome existants via le relais local + l'extension Chrome

Par défaut :

-   Le profil `openclaw` est créé automatiquement s'il est manquant.
-   Le profil `chrome` est intégré pour le relais d'extension Chrome (pointe par défaut vers `http://127.0.0.1:18792`).
-   Les ports CDP locaux sont alloués par défaut à partir de **18800–18899**.
-   Supprimer un profil déplace son répertoire de données local vers la Corbeille.

Tous les endpoints de contrôle acceptent `?profile=` ; la CLI utilise `--browser-profile`.

## Relais d'extension Chrome (utilisez votre Chrome existant)

OpenClaw peut également piloter **vos onglets Chrome existants** (pas d'instance Chrome "openclaw" séparée) via un relais CDP local + une extension Chrome. Guide complet : [Extension Chrome](./chrome-extension.md) Flux :

-   La Gateway s'exécute localement (même machine) ou un hôte nœud s'exécute sur la machine du navigateur.
-   Un **serveur de relais** local écoute sur une `cdpUrl` en boucle locale (défaut : `http://127.0.0.1:18792`).
-   Vous cliquez sur l'icône de l'extension **OpenClaw Browser Relay** sur un onglet pour vous attacher (elle ne s'attache pas automatiquement).
-   L'agent contrôle cet onglet via l'outil `browser` normal, en sélectionnant le bon profil.

Si la Gateway s'exécute ailleurs, exécutez un hôte nœud sur la machine du navigateur pour que la Gateway puisse proxy les actions du navigateur.

### Sessions sandboxées

Si la session de l'agent est sandboxée, l'outil `browser` peut par défaut utiliser `target="sandbox"` (navigateur sandbox). La prise de contrôle par le relais d'extension Chrome nécessite un contrôle du navigateur hôte, donc soit :

-   exécutez la session non sandboxée, soit
-   définissez `agents.defaults.sandbox.browser.allowHostControl: true` et utilisez `target="host"` lors de l'appel de l'outil.

### Installation

1.  Chargez l'extension (dev/unpacked) :

```bash
openclaw browser extension install
```

-   Chrome → `chrome://extensions` → activez "Mode développeur"
-   "Charger l'extension non empaquetée" → sélectionnez le répertoire affiché par `openclaw browser extension path`
-   Épinglez l'extension, puis cliquez dessus sur l'onglet que vous voulez contrôler (le badge affiche `ON`).

2.  Utilisez-la :

-   CLI : `openclaw browser --browser-profile chrome tabs`
-   Outil agent : `browser` avec `profile="chrome"`

Optionnel : si vous voulez un nom ou un port de relais différent, créez votre propre profil :

```bash
openclaw browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

Notes :

-   Ce mode repose sur Playwright-on-CDP pour la plupart des opérations (captures d'écran/instantanés/actions).
-   Détachez-vous en cliquant à nouveau sur l'icône de l'extension.

## Garanties d'isolation

-   **Répertoire de données utilisateur dédié** : ne touche jamais à votre profil de navigateur personnel.
-   **Ports dédiés** : évite `9222` pour prévenir les collisions avec les flux de travail de développement.
-   **Contrôle d'onglet déterministe** : cible les onglets par `targetId`, pas par "dernier onglet".

## Sélection du navigateur

Lors d'un lancement local, OpenClaw choisit le premier disponible :

1.  Chrome
2.  Brave
3.  Edge
4.  Chromium
5.  Chrome Canary

Vous pouvez remplacer avec `browser.executablePath`. Plateformes :

-   macOS : vérifie `/Applications` et `~/Applications`.
-   Linux : cherche `google-chrome`, `brave`, `microsoft-edge`, `chromium`, etc.
-   Windows : vérifie les emplacements d'installation courants.

## API de contrôle (optionnelle)

Pour les intégrations locales uniquement, la Gateway expose une petite API HTTP en boucle locale :

-   Statut/démarrage/arrêt : `GET /`, `POST /start`, `POST /stop`
-   Onglets : `GET /tabs`, `POST /tabs/open`, `POST /tabs/focus`, `DELETE /tabs/:targetId`
-   Instantané/capture d'écran : `GET /snapshot`, `POST /screenshot`
-   Actions : `POST /navigate`, `POST /act`
-   Hooks : `POST /hooks/file-chooser`, `POST /hooks/dialog`
-   Téléchargements : `POST /download`, `POST /wait/download`
-   Débogage : `GET /console`, `POST /pdf`
-   Débogage : `GET /errors`, `GET /requests`, `POST /trace/start`, `POST /trace/stop`, `POST /highlight`
-   Réseau : `POST /response/body`
-   État : `GET /cookies`, `POST /cookies/set`, `POST /cookies/clear`
-   État : `GET /storage/:kind`, `POST /storage/:kind/set`, `POST /storage/:kind/clear`
-   Paramètres : `POST /set/offline`, `POST /set/headers`, `POST /set/credentials`, `POST /set/geolocation`, `POST /set/media`, `POST /set/timezone`, `POST /set/locale`, `POST /set/device`

Tous les endpoints acceptent `?profile=`. Si l'authentification de la gateway est configurée, les routes HTTP du navigateur la requièrent également :

-   `Authorization: Bearer `
-   `x-openclaw-password: ` ou authentification HTTP Basic avec ce mot de passe

### Exigence Playwright

Certaines fonctionnalités (navigation/action/instantané IA/instantané de rôle, captures d'écran d'élément, PDF) nécessitent Playwright. Si Playwright n'est pas installé, ces endpoints renvoient une erreur 501 claire. Les instantanés ARIA et les captures d'écran basiques fonctionnent toujours pour Chrome géré par openclaw. Pour le pilote de relais d'extension Chrome, les instantanés ARIA et les captures d'écran nécessitent Playwright. Si vous voyez `Playwright is not available in this gateway build`, installez le package Playwright complet (pas `playwright-core`) et redémarrez la gateway, ou réinstallez OpenClaw avec le support navigateur.

#### Installation Playwright Docker

Si votre Gateway s'exécute dans Docker, évitez `npx playwright` (conflits de remplacement npm). Utilisez plutôt la CLI fournie :

```bash
docker compose run --rm openclaw-cli \
  node /app/node_modules/playwright-core/cli.js install chromium
```

Pour persister les téléchargements de navigateurs, définissez `PLAYWRIGHT_BROWSERS_PATH` (par exemple, `/home/node/.cache/ms-playwright`) et assurez-vous que `/home/node` est persisté via `OPENCLAW_HOME_VOLUME` ou un montage bind. Voir [Docker](../install/docker.md).

## Fonctionnement (interne)

Flux de haut niveau :

-   Un petit **serveur de contrôle** accepte les requêtes HTTP.
-   Il se connecte aux navigateurs basés sur Chromium (Chrome/Brave/Edge/Chromium) via **CDP**.
-   Pour les actions avancées (cliquer/taper/instantané/PDF), il utilise **Playwright** par-dessus CDP.
-   Lorsque Playwright est manquant, seules les opérations non-Playwright sont disponibles.

Cette conception maintient l'agent sur une interface stable et déterministe tout en vous permettant d'échanger des navigateurs et profils locaux/distants.

## Référence rapide CLI

Toutes les commandes acceptent `--browser-profile ` pour cibler un profil spécifique. Toutes les commandes acceptent également `--json` pour une sortie lisible par machine (charges utiles stables). Bases :

-   `openclaw browser status`
-   `openclaw browser start`
-   `openclaw browser stop`
-   `openclaw browser tabs`
-   `openclaw browser tab`
-   `openclaw browser tab new`
-   `openclaw browser tab select 2`
-   `openclaw browser tab close 2`
-   `openclaw browser open https://example.com`
-   `openclaw browser focus abcd1234`
-   `openclaw browser close abcd1234`

Inspection :

-   `openclaw browser screenshot`
-   `openclaw browser screenshot --full-page`
-   `openclaw browser screenshot --ref 12`
-   `openclaw browser screenshot --ref e12`
-   `openclaw browser snapshot`
-   `openclaw browser snapshot --format aria --limit 200`
-   `openclaw browser snapshot --interactive --compact --depth 6`
-   `openclaw browser snapshot --efficient`
-   `openclaw browser snapshot --labels`
-   `openclaw browser snapshot --selector "#main" --interactive`
-   `openclaw browser snapshot --frame "iframe#main" --interactive`
-   `openclaw browser console --level error`
-   `openclaw browser errors --clear`
-   `openclaw browser requests --filter api --clear`
-   `openclaw browser pdf`
-   `openclaw browser responsebody "**/api" --max-chars 5000`

Actions :

-   `openclaw browser navigate https://example.com`
-   `openclaw browser resize 1280 720`
-   `openclaw browser click 12 --double`
-   `openclaw browser click e12 --double`
-   `openclaw browser type 23 "hello" --submit`
-   `openclaw browser press Enter`
-   `openclaw browser hover 44`
-   `openclaw browser scrollintoview e12`
-   `openclaw browser drag 10 11`
-   `openclaw browser select 9 OptionA OptionB`
-   `openclaw browser download e12 report.pdf`
-   `openclaw browser waitfordownload report.pdf`
-   `openclaw browser upload /tmp/openclaw/uploads/file.pdf`
-   `openclaw browser fill --fields '[{"ref":"1","type":"text","value":"Ada"}]'`
-   `openclaw browser dialog --accept`
-   `openclaw browser wait --text "Done"`
-   `openclaw browser wait "#main" --url "**/dash" --load networkidle --fn "window.ready===true"`
-   `openclaw browser evaluate --fn '(el) => el.textContent' --ref 7`
-   `openclaw browser highlight e12`
-   `openclaw browser trace start`
-   `openclaw browser trace stop`

État :

-   `openclaw browser cookies`
-   `openclaw browser cookies set session abc123 --url "https://example.com"`
-   `openclaw browser cookies clear`
-   `openclaw browser storage local get`
-   `openclaw browser storage local set theme dark`
-   `openclaw browser storage session clear`
-   `openclaw browser set offline on`
-   `openclaw browser set headers --headers-json '{"X-Debug":"1"}'`
-   `openclaw browser set credentials user pass`
-   `openclaw browser set credentials --clear`
-   `openclaw browser set geo 37.7749 -122.4194 --origin "https://example.com"`
-   `openclaw browser set geo --clear`
-   `openclaw browser set media dark`
-   `openclaw browser set timezone America/New_York`
-   `openclaw browser set locale en-US`
-   `openclaw browser set device "iPhone 14"`

Notes :

-   `upload` et `dialog` sont des appels **d'armement** ; exécutez-les avant le clic/appui qui déclenche le sélecteur de fichier/boîte de dialogue.
-   Les chemins de sortie des téléchargements et traces sont contraints aux racines temporaires d'OpenClaw :
    -   traces : `/tmp/openclaw` (secours : `${os.tmpdir()}/openclaw`)
    -   téléchargements : `/tmp/openclaw/downloads` (secours : `${os.tmpdir()}/openclaw/downloads`)
-   Les chemins de téléversement sont contraints à une racine temporaire de téléversements OpenClaw :
    -   téléversements : `/tmp/openclaw/uploads` (secours : `${os.tmpdir()}/openclaw/uploads`)
-   `upload` peut également définir directement les champs de fichier via `--input-ref` ou `--element`.
-   `snapshot` :
    -   `--format ai` (défaut lorsque Playwright est installé) : renvoie un instantané IA avec des références numériques (`aria-ref=""`).
    -   `--format aria` : renvoie l'arbre d'accessibilité (pas de références ; inspection uniquement).
    -   `--efficient` (ou `--mode efficient`) : préréglage d'instantané de rôle compact (interactif + compact + profondeur + maxChars inférieur).
    -   Défaut de configuration (outil/CLI uniquement) : définissez `browser.snapshotDefaults.mode: "efficient"` pour utiliser des instantanés efficaces lorsque l'appelant ne passe pas de mode (voir [Configuration de la Gateway](../gateway/configuration.md#browser-openclaw-managed-browser)).
    -   Les options d'instantané de rôle (`--interactive`, `--compact`, `--depth`, `--selector`) forcent un instantané basé sur les rôles avec des références comme `ref=e12`.
    -   `--frame ""` limite les instantanés de rôle à un iframe (s'associe avec des références de rôle comme `e12`).
    -   `--interactive` produit une liste plate et facile à choisir d'éléments interactifs (meilleur pour piloter des actions).
    -   `--labels` ajoute une capture d'écran de la fenêtre d'affichage uniquement avec des étiquettes de référence superposées (affiche `MEDIA:`).
-   `click`/`type`/etc nécessitent une `ref` provenant de `snapshot` (soit numérique `12` soit référence de rôle `e12`). Les sélecteurs CSS ne sont intentionnellement pas pris en charge pour les actions.

## Instantanés et références

OpenClaw prend en charge deux styles "d'instantané" :

-   **Instantané IA (références numériques)** : `openclaw browser snapshot` (défaut ; `--format ai`)
    -   Sortie : un instantané texte qui inclut des références numériques.
    -   Actions : `openclaw browser click 12`, `openclaw browser type 23 "hello"`.
    -   En interne, la référence est résolue via `aria-ref` de Playwright.
-   **Instantané de rôle (références de rôle comme `e12`)** : `openclaw browser snapshot --interactive` (ou `--compact`, `--depth`, `--selector`, `--frame`)
    -   Sortie : une liste/arbre basé sur les rôles avec `[ref=e12]` (et optionnellement `[nth=1]`).
    -   Actions : `openclaw browser click e12`, `openclaw browser highlight e12`.
    -   En interne, la référence est résolue via `getByRole(...)` (plus `nth()` pour les doublons).
    -   Ajoutez `--labels` pour inclure une capture d'écran de la fenêtre d'affichage avec des étiquettes `e12` superposées.

Comportement des références :

-   Les références **ne sont pas stables entre les navigations** ; si quelque chose échoue, réexécutez `snapshot` et utilisez une nouvelle référence.
-   Si l'instantané de rôle a été pris avec `--frame`, les références de rôle sont limitées à cet iframe jusqu'au prochain instantané de rôle.

## Améliorations d'attente

Vous pouvez attendre plus que juste du temps/du texte :

-   Attendre une URL (les globs sont pris en charge par Playwright) :
    -   `openclaw browser wait --url "**/dash"`
-   Attendre un état de chargement :
    -   `openclaw browser wait --load networkidle`
-   Attendre un prédicat JS :
    -   `openclaw browser wait --fn "window.ready===true"`
-   Attendre qu'un sélecteur devienne visible :
    -   `openclaw browser wait "#main"`

Ils peuvent être combinés :

```bash
openclaw browser wait "#main" \
  --url "**/dash" \
  --load networkidle \
  --fn "window.ready===true" \
  --timeout-ms 15000
```

## Flux de travail de débogage

Lorsqu'une action échoue (par exemple "not visible", "strict mode violation", "covered") :

1.  `openclaw browser snapshot --interactive`
2.  Utilisez `click ` / `type ` (préférez les références de rôle en mode interactif)
3.  Si cela échoue toujours : `openclaw browser highlight ` pour voir ce que Playwright cible
4.  Si la page se comporte bizarrement :
    -   `openclaw browser errors --clear`
    -   `openclaw browser requests --filter api --clear`
5.  Pour un débogage approfondi : enregistrez une trace :
    -   `openclaw browser trace start`
    -   reproduisez le problème
    -   `openclaw browser trace stop` (affiche `TRACE:`)

## Sortie JSON

`--json` est destiné au script et aux outils structurés. Exemples :

```bash
openclaw browser status --json
openclaw browser snapshot --interactive --json
openclaw browser requests --filter api --json
openclaw browser cookies --json
```

Les instantanés de rôle en JSON incluent `refs` plus un petit bloc `stats` (lignes/caractères/références/interactif) pour que les outils puissent raisonner sur la taille et la densité de la charge utile.

## État et paramètres d'environnement

Ceux-ci sont utiles pour les flux de travail "faire se comporter le site comme X" :

-   Cookies : `cookies`, `cookies set`, `cookies clear`
-   Stockage : `storage local|session get|set|clear`
-   Hors ligne : `set offline on|off`
-   En-têtes : `set headers --headers-json '{"X-Debug":"1"}'` (l'hérité `set headers --json '{"X-Debug":"1"}'` reste pris en charge)
-   Authentification HTTP basic : `set credentials user pass` (ou `--clear`)
-   Géolocalisation : `set geo   --origin "https://example.com"` (ou `--clear`)
-   Média : `set media dark|light|no-preference|none`
-   Fuseau horaire / locale : `set timezone ...`, `set locale ...`
-   Appareil / fenêtre d'affichage :
    -   `set device "iPhone 14"` (préréglages d'appareil Playwright)
    -   `set viewport 1280 720`

## Sécurité et confidentialité

-   Le profil de navigateur openclaw peut contenir des sessions connectées ; traitez-le comme sensible.
-   `browser act kind=evaluate` / `openclaw browser evaluate` et `wait --fn` exécutent du JavaScript arbitraire dans le contexte de la page. L'injection de prompt peut orienter cela. Désactivez-le avec `browser.evaluateEnabled=false` si vous n'en avez pas besoin.
-   Pour les connexions et les notes anti-bot (X/Twitter, etc.), voir [Connexion navigateur + publication X/Twitter](./browser-login.md).
-   Gardez la Gateway/l'hôte nœud privé (boucle locale ou tailnet uniquement).
-   Les endpoints CDP distants sont puissants ; tunnelisez-les et protégez-les.

Exemple de mode strict (bloque les destinations privées/internes par défaut) :

```json
{
  browser: {
    ssrfPolicy: {
      dangerouslyAllowPrivateNetwork: false,
      hostnameAllowlist: ["*.example.com", "example.com"],
      allowedHostnames: ["localhost"], // autorisation exacte optionnelle
    },
  },
}
```

## Dépannage

Pour les problèmes spécifiques à Linux (en particulier Chromium snap), voir [Dépannage navigateur Linux](./browser-linux-troubleshooting.md).

## Outils de l'agent + fonctionnement du contrôle

L'agent obtient **un outil** pour l'automatisation du navigateur :

-   `browser` — statut/démarrage/arrêt/onglets/ouverture/focus/fermeture/instantané/capture d'écran/navigation/action

Comment cela se mappe :

-   `browser snapshot` renvoie un arbre d'interface utilisateur stable (IA ou ARIA).
-   `browser act` utilise les IDs `ref` de l'instantané pour cliquer/taper/glisser/sélectionner.
-   `browser screenshot` capture des pixels (page entière ou élément).
-   `browser` accepte :
    -   `profile` pour choisir un profil de navigateur nommé (openclaw, chrome, ou CDP distant).
    -   `target` (`sandbox` | `host` | `node`) pour sélectionner où se trouve le navigateur.
    -   Dans les sessions sandboxées, `target: "host"` nécessite `agents.defaults.sandbox.browser.allowHostControl=true`.
    -   Si `target` est omis : les sessions sandboxées utilisent par défaut `sandbox`, les sessions non sandboxées utilisent par défaut `host`.
    -   Si un nœud capable de navigateur est connecté, l