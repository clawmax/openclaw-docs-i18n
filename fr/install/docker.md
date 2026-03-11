

  Autres méthodes d'installation

  
# Docker

Docker est **optionnel**. Utilisez-le uniquement si vous voulez une passerelle conteneurisée ou pour valider le flux Docker.

## Docker est-il fait pour moi ?

-   **Oui** : vous voulez un environnement de passerelle isolé et jetable ou exécuter OpenClaw sur un hôte sans installations locales.
-   **Non** : vous exécutez sur votre propre machine et voulez juste la boucle de développement la plus rapide. Utilisez plutôt le flux d'installation normal.
-   **Note sur le bac à sable** : le bac à sable des agents utilise aussi Docker, mais il **ne** nécessite **pas** que la passerelle complète s'exécute dans Docker. Voir [Bac à sable](../gateway/sandboxing.md).

Ce guide couvre :

-   Passerelle conteneurisée (OpenClaw complet dans Docker)
-   Bac à sable par session pour les agents (passerelle hôte + outils d'agent isolés par Docker)

Détails sur le bac à sable : [Bac à sable](../gateway/sandboxing.md)

## Prérequis

-   Docker Desktop (ou Docker Engine) + Docker Compose v2
-   Au moins 2 Go de RAM pour la construction de l'image (`pnpm install` peut être tué par manque de mémoire sur des hôtes de 1 Go avec le code de sortie 137)
-   Suffisamment d'espace disque pour les images + logs
-   Si vous exécutez sur un VPS/hôte public, consultez [Renforcement de la sécurité pour l'exposition réseau](../gateway/security.md#04-network-exposure-bind--port--firewall), en particulier la politique de pare-feu Docker `DOCKER-USER`.

## Passerelle conteneurisée (Docker Compose)

### Démarrage rapide (recommandé)

> **ℹ️** Les valeurs par défaut Docker ici supposent des modes de liaison (`lan`/`loopback`), pas des alias d'hôte. Utilisez les valeurs de mode de liaison dans `gateway.bind` (par exemple `lan` ou `loopback`), pas des alias d'hôte comme `0.0.0.0` ou `localhost`.

 Depuis la racine du dépôt :

```
./docker-setup.sh
```

Ce script :

-   construit l'image de la passerelle localement (ou tire une image distante si `OPENCLAW_IMAGE` est défini)
-   exécute l'assistant de configuration initiale
-   affiche des conseils optionnels pour la configuration des fournisseurs
-   démarre la passerelle via Docker Compose
-   génère un jeton de passerelle et l'écrit dans `.env`

Variables d'environnement optionnelles :

-   `OPENCLAW_IMAGE` — utiliser une image distante au lieu de construire localement (par ex. `ghcr.io/openclaw/openclaw:latest`)
-   `OPENCLAW_DOCKER_APT_PACKAGES` — installer des paquets apt supplémentaires pendant la construction
-   `OPENCLAW_EXTENSIONS` — pré-installer les dépendances des extensions au moment de la construction (noms d'extensions séparés par des espaces, par ex. `diagnostics-otel matrix`)
-   `OPENCLAW_EXTRA_MOUNTS` — ajouter des montages de liaison hôte supplémentaires
-   `OPENCLAW_HOME_VOLUME` — persister `/home/node` dans un volume nommé
-   `OPENCLAW_SANDBOX` — activer l'amorçage du bac à sable Docker de la passerelle. Seules les valeurs explicitement vraies l'activent : `1`, `true`, `yes`, `on`
-   `OPENCLAW_INSTALL_DOCKER_CLI` — argument de construction transmis pour les constructions d'image locales (`1` installe Docker CLI dans l'image). `docker-setup.sh` définit ceci automatiquement quand `OPENCLAW_SANDBOX=1` pour les constructions locales.
-   `OPENCLAW_DOCKER_SOCKET` — remplacer le chemin du socket Docker (par défaut : chemin `DOCKER_HOST=unix://...`, sinon `/var/run/docker.sock`)
-   `OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1` — solution de secours : autoriser les cibles `ws://` de réseau privé de confiance pour les chemins client CLI/configuration initiale (par défaut uniquement en boucle locale)
-   `OPENCLAW_BROWSER_DISABLE_GRAPHICS_FLAGS=0` — désactiver les drapeaux de durcissement du navigateur en conteneur `--disable-3d-apis`, `--disable-software-rasterizer`, `--disable-gpu` quand vous avez besoin de compatibilité WebGL/3D.
-   `OPENCLAW_BROWSER_DISABLE_EXTENSIONS=0` — garder les extensions activées quand les flux navigateur les nécessitent (par défaut, les extensions sont désactivées dans le navigateur du bac à sable).
-   `OPENCLAW_BROWSER_RENDERER_PROCESS_LIMIT=` — définir la limite de processus de rendu Chromium ; définir à `0` pour ignorer le drapeau et utiliser le comportement par défaut de Chromium.

Une fois terminé :

-   Ouvrez `http://127.0.0.1:18789/` dans votre navigateur.
-   Collez le jeton dans l'interface de contrôle (Paramètres → jeton).
-   Besoin de l'URL à nouveau ? Exécutez `docker compose run --rm openclaw-cli dashboard --no-open`.

### Activer le bac à sable des agents pour la passerelle Docker (opt-in)

`docker-setup.sh` peut aussi amorcer `agents.defaults.sandbox.*` pour les déploiements Docker. Activez avec :

```bash
export OPENCLAW_SANDBOX=1
./docker-setup.sh
```

Chemin de socket personnalisé (par exemple Docker rootless) :

```bash
export OPENCLAW_SANDBOX=1
export OPENCLAW_DOCKER_SOCKET=/run/user/1000/docker.sock
./docker-setup.sh
```

Notes :

-   Le script monte `docker.sock` seulement après que les prérequis du bac à sable sont validés.
-   Si la configuration du bac à sable ne peut pas être terminée, le script réinitialise `agents.defaults.sandbox.mode` à `off` pour éviter une configuration de bac à sable obsolète/cassée lors des ré-exécutions.
-   Si `Dockerfile.sandbox` est manquant, le script affiche un avertissement et continue ; construisez `openclaw-sandbox:bookworm-slim` avec `scripts/sandbox-setup.sh` si nécessaire.
-   Pour les valeurs `OPENCLAW_IMAGE` non locales, l'image doit déjà contenir le support Docker CLI pour l'exécution du bac à sable.

### Automatisation/CI (non interactif, pas de bruit TTY)

Pour les scripts et l'intégration continue, désactivez l'allocation de pseudo-TTY de Compose avec `-T` :

```bash
docker compose run -T --rm openclaw-cli gateway probe
docker compose run -T --rm openclaw-cli devices list --json
```

Si votre automatisation n'exporte pas de variables de session Claude, les laisser non définies se résout maintenant à des valeurs vides par défaut dans `docker-compose.yml` pour éviter les avertissements répétés "variable non définie".

### Note de sécurité réseau partagé (CLI + passerelle)

`openclaw-cli` utilise `network_mode: "service:openclaw-gateway"` pour que les commandes CLI puissent atteindre de manière fiable la passerelle via `127.0.0.1` dans Docker. Traitez ceci comme une limite de confiance partagée : la liaison en boucle locale n'est pas une isolation entre ces deux conteneurs. Si vous avez besoin d'une séparation plus forte, exécutez les commandes depuis un chemin réseau conteneur/hôte séparé au lieu du service groupé `openclaw-cli`. Pour réduire l'impact si le processus CLI est compromis, la configuration compose supprime `NET_RAW`/`NET_ADMIN` et active `no-new-privileges` sur `openclaw-cli`. Il écrit la configuration/l'espace de travail sur l'hôte :

-   `~/.openclaw/`
-   `~/.openclaw/workspace`

Exécution sur un VPS ? Voir [Hetzner (VPS Docker)](./hetzner.md).

### Utiliser une image distante (ignorer la construction locale)

Les images pré-construites officielles sont publiées à :

-   [Package GitHub Container Registry](https://github.com/openclaw/openclaw/pkgs/container/openclaw)

Utilisez le nom d'image `ghcr.io/openclaw/openclaw` (pas les images Docker Hub de nom similaire). Étiquettes courantes :

-   `main` — dernière construction depuis `main`
-   `` — constructions d'étiquette de version (par exemple `2026.2.26`)
-   `latest` — dernière étiquette de version stable

### Métadonnées de l'image de base

L'image Docker principale utilise actuellement :

-   `node:22-bookworm`

L'image docker publie maintenant des annotations d'image de base OCI (sha256 est un exemple) :

-   `org.opencontainers.image.base.name=docker.io/library/node:22-bookworm`
-   `org.opencontainers.image.base.digest=sha256:6d735b4d33660225271fda0a412802746658c3a1b975507b2803ed299609760a`
-   `org.opencontainers.image.source=https://github.com/openclaw/openclaw`
-   `org.opencontainers.image.url=https://openclaw.ai`
-   `org.opencontainers.image.documentation=https://docs.openclaw.ai/install/docker`
-   `org.opencontainers.image.licenses=MIT`
-   `org.opencontainers.image.title=OpenClaw`
-   `org.opencontainers.image.description=OpenClaw gateway and CLI runtime container image`
-   `org.opencontainers.image.revision=<git-sha>`
-   `org.opencontainers.image.version=<tag-or-main>`
-   `org.opencontainers.image.created=`

Référence : [Annotations d'image OCI](https://github.com/opencontainers/image-spec/blob/main/annotations.md) Contexte de version : l'historique étiqueté de ce dépôt utilise déjà Bookworm dans `v2026.2.22` et les étiquettes 2026 antérieures (par exemple `v2026.2.21`, `v2026.2.9`). Par défaut, le script de configuration construit l'image depuis la source. Pour tirer une image pré-construite à la place, définissez `OPENCLAW_IMAGE` avant d'exécuter le script :

```bash
export OPENCLAW_IMAGE="ghcr.io/openclaw/openclaw:latest"
./docker-setup.sh
```

Le script détecte que `OPENCLAW_IMAGE` n'est pas la valeur par défaut `openclaw:local` et exécute `docker pull` au lieu de `docker build`. Tout le reste (configuration initiale, démarrage de la passerelle, génération de jeton) fonctionne de la même manière. `docker-setup.sh` s'exécute toujours depuis la racine du dépôt car il utilise le `docker-compose.yml` local et les fichiers d'aide. `OPENCLAW_IMAGE` évite le temps de construction d'image locale ; il ne remplace pas le flux de travail compose/configuration.

### Aides Shell (optionnel)

Pour une gestion Docker quotidienne plus facile, installez `ClawDock` :

```bash
mkdir -p ~/.clawdock && curl -sL https://raw.githubusercontent.com/openclaw/openclaw/main/scripts/shell-helpers/clawdock-helpers.sh -o ~/.clawdock/clawdock-helpers.sh
```

**Ajoutez à votre configuration shell (zsh) :**

```bash
echo 'source ~/.clawdock/clawdock-helpers.sh' >> ~/.zshrc && source ~/.zshrc
```

Utilisez ensuite `clawdock-start`, `clawdock-stop`, `clawdock-dashboard`, etc. Exécutez `clawdock-help` pour toutes les commandes. Voir le [README de l'aide `ClawDock`](https://github.com/openclaw/openclaw/blob/main/scripts/shell-helpers/README.md) pour les détails.

### Flux manuel (compose)

```bash
docker build -t openclaw:local -f Dockerfile .
docker compose run --rm openclaw-cli onboard
docker compose up -d openclaw-gateway
```

Note : exécutez `docker compose ...` depuis la racine du dépôt. Si vous avez activé `OPENCLAW_EXTRA_MOUNTS` ou `OPENCLAW_HOME_VOLUME`, le script de configuration écrit `docker-compose.extra.yml` ; incluez-le lors de l'exécution de Compose ailleurs :

```bash
docker compose -f docker-compose.yml -f docker-compose.extra.yml <command>
```

### Jeton de l'interface de contrôle + appairage (Docker)

Si vous voyez "non autorisé" ou "déconnecté (1008) : appairage requis", récupérez un nouveau lien de tableau de bord et approuvez l'appareil navigateur :

```bash
docker compose run --rm openclaw-cli dashboard --no-open
docker compose run --rm openclaw-cli devices list
docker compose run --rm openclaw-cli devices approve <requestId>
```

Plus de détails : [Tableau de bord](../web/dashboard.md), [Appareils](../cli/devices.md).

### Montages supplémentaires (optionnel)

Si vous voulez monter des répertoires hôte supplémentaires dans les conteneurs, définissez `OPENCLAW_EXTRA_MOUNTS` avant d'exécuter `docker-setup.sh`. Ceci accepte une liste séparée par des virgules de montages de liaison Docker et les applique à la fois à `openclaw-gateway` et `openclaw-cli` en générant `docker-compose.extra.yml`. Exemple :

```bash
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

Notes :

-   Les chemins doivent être partagés avec Docker Desktop sur macOS/Windows.
-   Chaque entrée doit être `source:cible[:options]` sans espaces, tabulations ou sauts de ligne.
-   Si vous modifiez `OPENCLAW_EXTRA_MOUNTS`, ré-exécutez `docker-setup.sh` pour régénérer le fichier compose supplémentaire.
-   `docker-compose.extra.yml` est généré. Ne le modifiez pas manuellement.

### Persister l'intégralité du répertoire home du conteneur (optionnel)

Si vous voulez que `/home/node` persiste à travers la recréation du conteneur, définissez un volume nommé via `OPENCLAW_HOME_VOLUME`. Ceci crée un volume Docker et le monte sur `/home/node`, tout en gardant les montages de liaison config/workspace standard. Utilisez un volume nommé ici (pas un chemin de liaison) ; pour les montages de liaison, utilisez `OPENCLAW_EXTRA_MOUNTS`. Exemple :

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
./docker-setup.sh
```

Vous pouvez combiner ceci avec des montages supplémentaires :

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

Notes :

-   Les volumes nommés doivent correspondre à `^[A-Za-z0-9][A-Za-z0-9_.-]*$`.
-   Si vous changez `OPENCLAW_HOME_VOLUME`, ré-exécutez `docker-setup.sh` pour régénérer le fichier compose supplémentaire.
-   Le volume nommé persiste jusqu'à sa suppression avec `docker volume rm `.

### Installer des paquets apt supplémentaires (optionnel)

Si vous avez besoin de paquets système dans l'image (par exemple, outils de construction ou bibliothèques multimédias), définissez `OPENCLAW_DOCKER_APT_PACKAGES` avant d'exécuter `docker-setup.sh`. Ceci installe les paquets pendant la construction de l'image, donc ils persistent même si le conteneur est supprimé. Exemple :

```bash
export OPENCLAW_DOCKER_APT_PACKAGES="ffmpeg build-essential"
./docker-setup.sh
```

Notes :

-   Ceci accepte une liste séparée par des espaces de noms de paquets apt.
-   Si vous changez `OPENCLAW_DOCKER_APT_PACKAGES`, ré-exécutez `docker-setup.sh` pour reconstruire l'image.

### Pré-installer les dépendances des extensions (optionnel)

Les extensions avec leur propre `package.json` (par ex. `diagnostics-otel`, `matrix`, `msteams`) installent leurs dépendances npm au premier chargement. Pour intégrer ces dépendances dans l'image à la place, définissez `OPENCLAW_EXTENSIONS` avant d'exécuter `docker-setup.sh` :

```bash
export OPENCLAW_EXTENSIONS="diagnostics-otel matrix"
./docker-setup.sh
```

Ou lors d'une construction directe :

```bash
docker build --build-arg OPENCLAW_EXTENSIONS="diagnostics-otel matrix" .
```

Notes :

-   Ceci accepte une liste séparée par des espaces de noms de répertoires d'extensions (sous `extensions/`).
-   Seules les extensions avec un `package.json` sont affectées ; les plugins légers sans `package.json` sont ignorés.
-   Si vous changez `OPENCLAW_EXTENSIONS`, ré-exécutez `docker-setup.sh` pour reconstruire l'image.

### Conteneur expert / complet (opt-in)

L'image Docker par défaut est **axée sur la sécurité** et s'exécute en tant qu'utilisateur non-root `node`. Ceci garde la surface d'attaque petite, mais cela signifie :

-   pas d'installation de paquets système à l'exécution
-   pas de Homebrew par défaut
-   pas de navigateurs Chromium/Playwright groupés

Si vous voulez un conteneur plus complet, utilisez ces options opt-in :

1.  **Persister `/home/node`** pour que les téléchargements de navigateurs et les caches d'outils survivent :

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
./docker-setup.sh
```

2.  **Intégrer les dépendances système dans l'image** (répétable + persistant) :

```bash
export OPENCLAW_DOCKER_APT_PACKAGES="git curl jq"
./docker-setup.sh
```

3.  **Installer les navigateurs Playwright sans `npx`** (évite les conflits de remplacement npm) :

```bash
docker compose run --rm openclaw-cli \
  node /app/node_modules/playwright-core/cli.js install chromium
```

Si vous avez besoin que Playwright installe des dépendances système, reconstruisez l'image avec `OPENCLAW_DOCKER_APT_PACKAGES` au lieu d'utiliser `--with-deps` à l'exécution.

4.  **Persister les téléchargements de navigateurs Playwright** :

-   Définissez `PLAYWRIGHT_BROWSERS_PATH=/home/node/.cache/ms-playwright` dans `docker-compose.yml`.
-   Assurez-vous que `/home/node` persiste via `OPENCLAW_HOME_VOLUME`, ou montez `/home/node/.cache/ms-playwright` via `OPENCLAW_EXTRA_MOUNTS`.

### Permissions + EACCES

L'image s'exécute en tant que `node` (uid 1000). Si vous voyez des erreurs de permission sur `/home/node/.openclaw`, assurez-vous que vos montages de liaison hôte sont possédés par l'uid 1000. Exemple (hôte Linux) :

```bash
sudo chown -R 1000:1000 /path/to/openclaw-config /path/to/openclaw-workspace
```

Si vous choisissez d'exécuter en tant que root pour plus de commodité, vous acceptez le compromis de sécurité.

### Reconstructions plus rapides (recommandé)

Pour accélérer les reconstructions, ordonnez votre Dockerfile pour que les couches de dépendances soient mises en cache. Ceci évite de ré-exécuter `pnpm install` à moins que les fichiers de verrouillage ne changent :

```dockerfile
FROM node:22-bookworm

# Installer Bun (requis pour les scripts de construction)
RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/root/.bun/bin:${PATH}"

RUN corepack enable

WORKDIR /app

# Mettre en cache les dépendances sauf si les métadonnées du paquet changent
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

### Configuration des canaux (optionnel)

Utilisez le conteneur CLI pour configurer les canaux, puis redémarrez la passerelle si nécessaire. WhatsApp (QR) :

```bash
docker compose run --rm openclaw-cli channels login
```

Telegram (jeton de bot) :

```bash
docker compose run --rm openclaw-cli channels add --channel telegram --token "<token>"
```

Discord (jeton de bot) :

```bash
docker compose run --rm openclaw-cli channels add --channel discord --token "<token>"
```

Docs : [WhatsApp](../channels/whatsapp.md), [Telegram](../channels/telegram.md), [Discord](../channels/discord.md)

### OAuth OpenAI Codex (Docker sans interface graphique)

Si vous choisissez OAuth OpenAI Codex dans l'assistant, il ouvre une URL de navigateur et essaie de capturer un rappel sur `http://127.0.0.1:1455/auth/callback`. Dans les configurations Docker ou sans interface graphique, ce rappel peut afficher une erreur de navigateur. Copiez l'URL de redirection complète sur laquelle vous atterrissez et collez-la à nouveau dans l'assistant pour terminer l'authentification.

### Vérifications de santé

Points de terminaison de sonde du conteneur (aucune authentification requise) :

```bash
curl -fsS http://127.0.0.1:18789/healthz
curl -fsS http://127.0.0.1:18789/readyz
```

Alias : `/health` et `/ready`. `/healthz` est une sonde de vitalité superficielle pour "le processus de passerelle est en cours". `/readyz` reste prêt pendant la période de grâce de démarrage, puis devient `503` seulement si les canaux gérés requis sont toujours déconnectés après la période de grâce ou se déconnectent plus tard. L'image Docker inclut une `HEALTHCHECK` intégrée qui envoie un ping à `/healthz` en arrière-plan. En termes simples : Docker continue de vérifier si OpenClaw est toujours réactif. Si les vérifications échouent continuellement, Docker marque le conteneur comme `unhealthy`, et les systèmes d'orchestration (politique de redémarrage Docker Compose, Swarm, Kubernetes, etc.) peuvent automatiquement le redémarrer ou le remplacer. Instantané de santé approfondi authentifié (passerelle + canaux) :

```bash
docker compose exec openclaw-gateway node dist/index.js health --token "$OPENCLAW_GATEWAY_TOKEN"
```

### Test de fumée de bout en bout (Docker)

```
scripts/e2e/onboard-docker.sh
```

### Test de fumée d'import QR (Docker)

```bash
pnpm test:docker:qr
```

### LAN vs boucle locale (Docker Compose)

`docker-setup.sh` définit par défaut `OPENCLAW_GATEWAY_BIND=lan` pour que l'accès hôte à `http://127.0.0.1:18789` fonctionne avec la publication de port Docker.

-   `lan` (par défaut) : le navigateur hôte + le CLI hôte peuvent atteindre le port de passerelle publié.
-   `loopback` : seuls les processus à l'intérieur de l'espace de noms réseau du conteneur peuvent atteindre directement la passerelle ; l'accès au port publié sur l'hôte peut échouer.

Le script de configuration fixe aussi `gateway.mode=local` après la configuration initiale pour que les commandes CLI Docker ciblent par défaut la boucle locale. Note sur la configuration héritée : utilisez les valeurs de mode de liaison dans `gateway.bind` (`lan` / `loopback` / `custom` / `tailnet` / `auto`), pas des alias d'hôte (`0.0.0.0`, `127.0.0.1`, `localhost`, `::`, `::1`). Si vous voyez `Gateway target: ws://172.x.x.x:18789` ou des erreurs répétées `pairing required` des commandes CLI Docker, exécutez :

```bash
docker compose run --rm openclaw-cli config set gateway.mode local
docker compose run --rm openclaw-cli config set gateway.bind lan
docker compose run --rm openclaw-cli devices list --url ws://127.0.0.1:18789
```

### Notes

-   La liaison de la passerelle est par défaut `lan` pour l'utilisation en conteneur (`OPENCLAW_GATEWAY_BIND`).
-   La commande CMD du Dockerfile utilise `--allow-unconfigured` ; la configuration montée avec `gateway.mode` différent de `local` démarrera quand même. Remplacez CMD pour appliquer la garde.
-   Le conteneur de passerelle est la source de vérité pour les sessions (`~/.openclaw/agents//sessions/`).

### Modèle de stockage

-   **Données hôtes persistantes :** Docker Compose monte par liaison `OPENCLAW_CONFIG_DIR` sur `/home/node/.openclaw` et `OPENCLAW_WORKSPACE_DIR` sur `/home/node/.openclaw/workspace`, donc ces chemins survivent au remplacement du conteneur.
-   **tmpfs éphémère du bac à sable :** quand `agents.defaults.sandbox` est activé, les conteneurs de bac à sable utilisent `tmpfs` pour `/tmp`, `/var/tmp`, et `/run`. Ces montages sont séparés de la pile Compose de niveau supérieur et disparaissent avec le conteneur de bac à sable.
-   **Points chauds de croissance du disque :** surveillez `media/`, `agents//sessions/sessions.json`, les fichiers JSONL de transcription, `cron/runs/*.jsonl`, et les logs de fichiers rotatifs sous `/tmp/openclaw/` (ou votre `logging.file` configuré). Si vous exécutez aussi l'application macOS en dehors de Docker, ses logs de service sont à nouveau séparés : `~/.openclaw/logs/gateway.log`, `~/.openclaw/logs/gateway.err.log`, et `/tmp/openclaw/openclaw-gateway.log`.

## Bac à sable des agents (passerelle hôte + outils Docker)

Plongée en profondeur : [Bac à sable](../gateway/sandboxing.md)

### Ce que cela fait

Quand `agents.defaults.sandbox` est activé, les **sessions non principales** exécutent les outils à l'intérieur d'un conteneur Docker. La passerelle reste sur votre hôte, mais l'exécution des outils est isolée :

-   portée : `"agent"` par défaut (un conteneur + espace de travail par agent)
-   portée : `"session"` pour l'isolation par session
-   dossier d'espace de travail par portée monté sur `/workspace`
-   accès optionnel à l'espace de travail de l'agent (`agents.defaults.sandbox.workspaceAccess`)
-   politique d'outils autoriser/interdire (interdire l'emporte)
-   les médias entrants sont copiés dans l'espace de travail actif du bac à sable (`media/inbound/*`) pour que les outils puissent les lire (avec `workspaceAccess: "rw"`, ceci atterrit dans l'espace de travail de l'agent)

Avertissement : `scope: "shared"` désactive l'isolation entre sessions. Toutes les sessions partagent un conteneur et un espace de travail.

### Profils de bac à sable par agent (multi-agent)

Si vous utilisez le routage multi-agent, chaque agent peut remplacer les paramètres de bac à sable + outils : `agents.list[].sandbox` et `agents.list[].tools` (plus `agents.list[].tools.sandbox.tools`). Ceci vous permet d'exécuter des niveaux d'accès mixtes dans une seule passerelle :

-   Accès complet (agent personnel)
-   Outils en lecture seule + espace de travail en lecture seule (agent famille/travail)
-   Pas d'outils système de fichiers/shell (agent public)

Voir [Bac à sable & Outils Multi-Agent](../tools/multi-agent-sandbox-tools.md) pour des exemples, la précédence et le dépannage.

### Comportement par défaut

-   Image : `openclaw-sandbox:bookworm-slim`
-   Un conteneur par agent
-   Accès à l'espace de travail de l'agent : `workspaceAccess: "none"` (par défaut) utilise `~/.openclaw/sandboxes`
    -   `"ro"` garde l'espace de travail du bac à sable sur `/workspace` et monte l'espace de travail de l'agent en lecture seule sur `/agent` (désactive `write`/`edit`/`apply_patch`)
    -   `"rw"` monte l'espace de travail de l'agent en lecture/écriture sur `/workspace`
-   Élagage automatique : inactif > 24h OU âge > 7j
-   Réseau : `none` par défaut (activer explicitement si vous avez besoin d'égress)
    -   `host` est bloqué.
    -   `container:` est bloqué par défaut (risque de jonction d'espace de noms).
-   Autorisation par défaut : `exec`, `process`, `read`, `write`, `edit`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
-   Interdiction par défaut : `browser`, `canvas`, `nodes`, `cron`, `discord`, `gateway`

### Activer le bac à sable

Si vous prévoyez d'installer des paquets dans `setupCommand`, notez :

-   Le `docker.network` par défaut est `"none"` (pas d'égress).
-   `docker.network: "host"` est bloqué.
-   `docker.network: "container:"` est bloqué par défaut.
-   Remplacement de secours : `agents.defaults.sandbox.docker.dangerouslyAllowContainerNamespaceJoin: true`.
-   `readOnlyRoot: true` bloque l'installation de paquets.
-   `user` doit être root pour `apt-get` (omettez `user` ou définissez `user: "0:0"`). OpenClaw recrée automatiquement les conteneurs quand `setupCommand` (ou la configuration docker) change, sauf si le conteneur a été **récemment utilisé** (dans les ~5 minutes). Les conteneurs chauds enregistrent un avertissement avec la commande exacte `openclaw sandbox recreate ...`.

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared (agent est la valeur par défaut)
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256,
          },
          seccompProfile: "/path/to/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"],
        },
        prune: {
          idleHours: 24, // 0 désactive l'élagage par inactivité
          max