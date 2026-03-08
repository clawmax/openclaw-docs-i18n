

  Sécurité et sandboxing

  
# Sandboxing

OpenClaw peut exécuter **des outils à l'intérieur de conteneurs Docker** pour réduire le rayon d'impact. C'est **optionnel** et contrôlé par la configuration (`agents.defaults.sandbox` ou `agents.list[].sandbox`). Si le sandboxing est désactivé, les outils s'exécutent sur l'hôte. La passerelle reste sur l'hôte ; l'exécution des outils s'effectue dans un sandbox isolé lorsqu'il est activé. Ce n'est pas une limite de sécurité parfaite, mais cela limite matériellement l'accès au système de fichiers et aux processus lorsque le modèle fait quelque chose de stupide.

## Ce qui est sandboxé

-   L'exécution des outils (`exec`, `read`, `write`, `edit`, `apply_patch`, `process`, etc.).
-   Navigateur sandboxé optionnel (`agents.defaults.sandbox.browser`).
    -   Par défaut, le navigateur sandboxé démarre automatiquement (garantit que CDP est accessible) lorsque l'outil navigateur en a besoin. Configurable via `agents.defaults.sandbox.browser.autoStart` et `agents.defaults.sandbox.browser.autoStartTimeoutMs`.
    -   Par défaut, les conteneurs du navigateur sandboxé utilisent un réseau Docker dédié (`openclaw-sandbox-browser`) au lieu du réseau global `bridge`. Configurable avec `agents.defaults.sandbox.browser.network`.
    -   L'option `agents.defaults.sandbox.browser.cdpSourceRange` restreint l'ingress CDP entre le conteneur et la passerelle avec une liste d'autorisation CIDR (par exemple `172.21.0.1/32`).
    -   L'accès observateur noVNC est protégé par mot de passe par défaut ; OpenClaw émet une URL à jeton de courte durée qui sert une page de démarrage locale et ouvre noVNC avec le mot de passe dans le fragment d'URL (pas dans les logs de requête/en-tête).
    -   `agents.defaults.sandbox.browser.allowHostControl` permet aux sessions sandboxées de cibler explicitement le navigateur de l'hôte.
    -   Des listes d'autorisation optionnelles contrôlent `target: "custom"` : `allowedControlUrls`, `allowedControlHosts`, `allowedControlPorts`.

Non sandboxé :

-   Le processus de la passerelle lui-même.
-   Tout outil explicitement autorisé à s'exécuter sur l'hôte (par ex. `tools.elevated`).
    -   **L'exécution élevée (`elevated exec`) s'exécute sur l'hôte et contourne le sandboxing.**
    -   Si le sandboxing est désactivé, `tools.elevated` ne change pas l'exécution (déjà sur l'hôte). Voir [Mode Élevé](../tools/elevated.md).

## Modes

`agents.defaults.sandbox.mode` contrôle **quand** le sandboxing est utilisé :

-   `"off"` : pas de sandboxing.
-   `"non-main"` : sandbox uniquement les sessions **non principales** (par défaut si vous voulez des discussions normales sur l'hôte).
-   `"all"` : chaque session s'exécute dans un sandbox. Note : `"non-main"` est basé sur `session.mainKey` (par défaut `"main"`), pas sur l'identifiant de l'agent. Les sessions de groupe/canal utilisent leurs propres clés, elles sont donc considérées comme non principales et seront sandboxées.

## Portée

`agents.defaults.sandbox.scope` contrôle **combien de conteneurs** sont créés :

-   `"session"` (par défaut) : un conteneur par session.
-   `"agent"` : un conteneur par agent.
-   `"shared"` : un conteneur partagé par toutes les sessions sandboxées.

## Accès à l'espace de travail

`agents.defaults.sandbox.workspaceAccess` contrôle **ce que le sandbox peut voir** :

-   `"none"` (par défaut) : les outils voient un espace de travail sandbox sous `~/.openclaw/sandboxes`.
-   `"ro"` : monte l'espace de travail de l'agent en lecture seule sur `/agent` (désactive `write`/`edit`/`apply_patch`).
-   `"rw"` : monte l'espace de travail de l'agent en lecture/écriture sur `/workspace`.

Les médias entrants sont copiés dans l'espace de travail sandbox actif (`media/inbound/*`). Note sur les compétences : l'outil `read` est ancré à la racine du sandbox. Avec `workspaceAccess: "none"`, OpenClaw reflète les compétences éligibles dans l'espace de travail sandbox (`.../skills`) pour qu'elles puissent être lues. Avec `"rw"`, les compétences de l'espace de travail sont lisibles depuis `/workspace/skills`.

## Montages bind personnalisés

`agents.defaults.sandbox.docker.binds` monte des répertoires hôtes supplémentaires dans le conteneur. Format : `hôte:conteneur:mode` (par ex., `"/home/user/source:/source:rw"`). Les montages globaux et par agent sont **fusionnés** (non remplacés). Sous `scope: "shared"`, les montages par agent sont ignorés. `agents.defaults.sandbox.browser.binds` monte des répertoires hôtes supplémentaires uniquement dans le conteneur du **navigateur sandboxé**.

-   Lorsqu'il est défini (y compris `[]`), il remplace `agents.defaults.sandbox.docker.binds` pour le conteneur du navigateur.
-   Lorsqu'il est omis, le conteneur du navigateur utilise `agents.defaults.sandbox.docker.binds` (rétrocompatible).

Exemple (source en lecture seule + un répertoire de données supplémentaire) :

```json
{
  agents: {
    defaults: {
      sandbox: {
        docker: {
          binds: ["/home/user/source:/source:ro", "/var/data/myapp:/data:ro"],
        },
      },
    },
    list: [
      {
        id: "build",
        sandbox: {
          docker: {
            binds: ["/mnt/cache:/cache:rw"],
          },
        },
      },
    ],
  },
}
```

Notes de sécurité :

-   Les montages bind contournent le système de fichiers du sandbox : ils exposent les chemins hôtes avec le mode que vous définissez (`:ro` ou `:rw`).
-   OpenClaw bloque les sources de montage dangereuses (par exemple : `docker.sock`, `/etc`, `/proc`, `/sys`, `/dev`, et les montages parents qui les exposeraient).
-   Les montages sensibles (secrets, clés SSH, identifiants de service) doivent être en `:ro` sauf absolue nécessité.
-   Combinez avec `workspaceAccess: "ro"` si vous n'avez besoin que d'un accès en lecture à l'espace de travail ; les modes de montage restent indépendants.
-   Voir [Sandbox vs Politique d'outils vs Élevé](./sandbox-vs-tool-policy-vs-elevated.md) pour comprendre comment les montages interagissent avec la politique d'outils et l'exécution élevée.

## Images + configuration

Image par défaut : `openclaw-sandbox:bookworm-slim` Construisez-la une fois :

```
scripts/sandbox-setup.sh
```

Note : l'image par défaut **n'inclut pas** Node. Si une compétence nécessite Node (ou d'autres runtimes), soit créez une image personnalisée, soit installez via `sandbox.docker.setupCommand` (nécessite un accès réseau sortant + racine accessible en écriture + utilisateur root). Si vous voulez une image sandbox plus fonctionnelle avec des outils courants (par exemple `curl`, `jq`, `nodejs`, `python3`, `git`), construisez :

```
scripts/sandbox-common-setup.sh
```

Puis définissez `agents.defaults.sandbox.docker.image` sur `openclaw-sandbox-common:bookworm-slim`. Image du navigateur sandboxé :

```
scripts/sandbox-browser-setup.sh
```

Par défaut, les conteneurs sandbox s'exécutent **sans réseau**. Remplacez avec `agents.defaults.sandbox.docker.network`. L'image de navigateur sandboxé fournie applique également des paramètres de démarrage Chromium conservateurs pour les charges de travail conteneurisées. Les paramètres par défaut actuels du conteneur incluent :

-   `--remote-debugging-address=127.0.0.1`
-   `--remote-debugging-port=<dérivé de OPENCLAW_BROWSER_CDP_PORT>`
-   `--user-data-dir=${HOME}/.chrome`
-   `--no-first-run`
-   `--no-default-browser-check`
-   `--disable-3d-apis`
-   `--disable-gpu`
-   `--disable-dev-shm-usage`
-   `--disable-background-networking`
-   `--disable-extensions`
-   `--disable-features=TranslateUI`
-   `--disable-breakpad`
-   `--disable-crash-reporter`
-   `--disable-software-rasterizer`
-   `--no-zygote`
-   `--metrics-recording-only`
-   `--renderer-process-limit=2`
-   `--no-sandbox` et `--disable-setuid-sandbox` lorsque `noSandbox` est activé.
-   Les trois drapeaux de durcissement graphique (`--disable-3d-apis`, `--disable-software-rasterizer`, `--disable-gpu`) sont optionnels et utiles lorsque les conteneurs manquent de support GPU. Définissez `OPENCLAW_BROWSER_DISABLE_GRAPHICS_FLAGS=0` si votre charge de travail nécessite WebGL ou d'autres fonctionnalités 3D/navigateur.
-   `--disable-extensions` est activé par défaut et peut être désactivé avec `OPENCLAW_BROWSER_DISABLE_EXTENSIONS=0` pour les flux dépendant des extensions.
-   `--renderer-process-limit=2` est contrôlé par `OPENCLAW_BROWSER_RENDERER_PROCESS_LIMIT=`, où `0` garde la valeur par défaut de Chromium.

Si vous avez besoin d'un profil d'exécution différent, utilisez une image de navigateur personnalisée et fournissez votre propre point d'entrée. Pour les profils Chromium locaux (non conteneurisés), utilisez `browser.extraArgs` pour ajouter des drapeaux de démarrage supplémentaires. Paramètres de sécurité par défaut :

-   `network: "host"` est bloqué.
-   `network: "container:"` est bloqué par défaut (risque de contournement par jointure d'espace de noms).
-   Contournement d'urgence : `agents.defaults.sandbox.docker.dangerouslyAllowContainerNamespaceJoin: true`.

Les installations Docker et la passerelle conteneurisée se trouvent ici : [Docker](../install/docker.md) Pour les déploiements de passerelle Docker, `docker-setup.sh` peut amorcer la configuration du sandbox. Définissez `OPENCLAW_SANDBOX=1` (ou `true`/`yes`/`on`) pour activer ce chemin. Vous pouvez remplacer l'emplacement du socket avec `OPENCLAW_DOCKER_SOCKET`. Configuration complète et référence d'environnement : [Docker](../install/docker.md#enable-agent-sandbox-for-docker-gateway-opt-in).

## setupCommand (configuration unique du conteneur)

`setupCommand` s'exécute **une fois** après la création du conteneur sandbox (pas à chaque exécution). Il s'exécute à l'intérieur du conteneur via `sh -lc`. Chemins :

-   Global : `agents.defaults.sandbox.docker.setupCommand`
-   Par agent : `agents.list[].sandbox.docker.setupCommand`

Pièges courants :

-   Par défaut, `docker.network` est `"none"` (pas de sortie), donc les installations de paquets échoueront.
-   `docker.network: "container:"` nécessite `dangerouslyAllowContainerNamespaceJoin: true` et est réservé aux situations d'urgence.
-   `readOnlyRoot: true` empêche les écritures ; définissez `readOnlyRoot: false` ou créez une image personnalisée.
-   `user` doit être root pour les installations de paquets (omettez `user` ou définissez `user: "0:0"`).
-   L'exécution sandbox **n'hérite pas** de `process.env` de l'hôte. Utilisez `agents.defaults.sandbox.docker.env` (ou une image personnalisée) pour les clés API des compétences.

## Politique d'outils + échappatoires

Les politiques d'autorisation/interdiction d'outils s'appliquent toujours avant les règles de sandbox. Si un outil est interdit globalement ou par agent, le sandboxing ne le réactive pas. `tools.elevated` est une échappatoire explicite qui exécute `exec` sur l'hôte. Les directives `/exec` ne s'appliquent qu'aux expéditeurs autorisés et persistent par session ; pour désactiver strictement `exec`, utilisez la politique de refus d'outils (voir [Sandbox vs Politique d'outils vs Élevé](./sandbox-vs-tool-policy-vs-elevated.md)). Débogage :

-   Utilisez `openclaw sandbox explain` pour inspecter le mode sandbox effectif, la politique d'outils et les clés de configuration de correction.
-   Voir [Sandbox vs Politique d'outils vs Élevé](./sandbox-vs-tool-policy-vs-elevated.md) pour le modèle mental "pourquoi est-ce bloqué ?". Gardez-le verrouillé.

## Surcharges multi-agents

Chaque agent peut remplacer le sandbox et les outils : `agents.list[].sandbox` et `agents.list[].tools` (plus `agents.list[].tools.sandbox.tools` pour la politique d'outils sandbox). Voir [Sandbox & Outils Multi-Agents](../tools/multi-agent-sandbox-tools.md) pour la priorité.

## Exemple minimal d'activation

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none",
      },
    },
  },
}
```

## Documentation associée

-   [Configuration du Sandbox](./configuration.md#agentsdefaults-sandbox)
-   [Sandbox & Outils Multi-Agents](../tools/multi-agent-sandbox-tools.md)
-   [Sécurité](./security.md)

[Sécurité](./security.md)[Sandbox vs Politique d'outils vs Élevé](./sandbox-vs-tool-policy-vs-elevated.md)