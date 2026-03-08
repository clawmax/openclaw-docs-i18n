title: "Déployer OpenClaw Gateway sur un VPS Hetzner avec Docker"
description: "Guide étape par étape pour exécuter un OpenClaw Gateway persistant sur un VPS Hetzner en utilisant Docker. Apprenez à configurer l'état, intégrer les binaires et accéder via un tunnel SSH."
keywords: ["vps hetzner", "openclaw gateway", "déploiement docker", "état persistant", "tunnel ssh", "docker compose", "persistance des binaires", "openclaw auto-hébergé"]
---

  Hébergement et déploiement

  
# Hetzner

## Objectif

Exécuter un OpenClaw Gateway persistant sur un VPS Hetzner en utilisant Docker, avec un état durable, des binaires intégrés et un comportement de redémarrage sécurisé. Si vous voulez « OpenClaw 24/7 pour ~5€ », c'est la configuration fiable la plus simple. Les tarifs Hetzner changent ; choisissez le plus petit VPS Debian/Ubuntu et montez en puissance si vous rencontrez des OOM. Rappel du modèle de sécurité :

-   Les agents partagés par l'entreprise sont acceptables lorsque tout le monde est dans le même périmètre de confiance et que l'environnement d'exécution est réservé aux activités professionnelles.
-   Maintenez une séparation stricte : VPS/environnement d'exécution dédié + comptes dédiés ; aucun profil personnel Apple/Google/navigateur/gestionnaire de mots de passe sur cet hôte.
-   Si les utilisateurs sont antagonistes les uns envers les autres, séparez-les par passerelle/hôte/utilisateur OS.

Voir [Sécurité](../gateway/security.md) et [Hébergement VPS](../vps.md).

## Que faisons-nous (en termes simples) ?

-   Louer un petit serveur Linux (VPS Hetzner)
-   Installer Docker (environnement d'exécution d'application isolé)
-   Démarrer l'OpenClaw Gateway dans Docker
-   Persister `~/.openclaw` + `~/.openclaw/workspace` sur l'hôte (survit aux redémarrages/reconstructions)
-   Accéder à l'Interface de Contrôle depuis votre ordinateur portable via un tunnel SSH

La passerelle peut être accessible via :

-   Le transfert de port SSH depuis votre ordinateur portable
-   L'exposition directe du port si vous gérez vous-même le pare-feu et les jetons

Ce guide suppose Ubuntu ou Debian sur Hetzner.  
Si vous êtes sur un autre VPS Linux, adaptez les paquets en conséquence. Pour le flux Docker générique, voir [Docker](./docker.md).

* * *

## Chemin rapide (opérateurs expérimentés)

1.  Provisionner un VPS Hetzner
2.  Installer Docker
3.  Cloner le dépôt OpenClaw
4.  Créer les répertoires persistants sur l'hôte
5.  Configurer `.env` et `docker-compose.yml`
6.  Intégrer les binaires requis dans l'image
7.  `docker compose up -d`
8.  Vérifier la persistance et l'accès à la passerelle

* * *

## Ce dont vous avez besoin

-   VPS Hetzner avec accès root
-   Accès SSH depuis votre ordinateur portable
-   Confort basique avec SSH + copier/coller
-   ~20 minutes
-   Docker et Docker Compose
-   Identifiants d'authentification des modèles
-   Identifiants de fournisseur optionnels
    -   QR WhatsApp
    -   Jeton de bot Telegram
    -   OAuth Gmail

* * *

## 1) Provisionner le VPS

Créez un VPS Ubuntu ou Debian sur Hetzner. Connectez-vous en tant que root :

```bash
ssh root@YOUR_VPS_IP
```

Ce guide suppose que le VPS est persistant. Ne le traitez pas comme une infrastructure jetable.

* * *

## 2) Installer Docker (sur le VPS)

```bash
apt-get update
apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sh
```

Vérifiez :

```bash
docker --version
docker compose version
```

* * *

## 3) Cloner le dépôt OpenClaw

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

Ce guide suppose que vous allez construire une image personnalisée pour garantir la persistance des binaires.

* * *

## 4) Créer les répertoires persistants sur l'hôte

Les conteneurs Docker sont éphémères. Tout état de longue durée doit résider sur l'hôte.

```bash
mkdir -p /root/.openclaw/workspace

# Définir la propriété pour l'utilisateur du conteneur (uid 1000) :
chown -R 1000:1000 /root/.openclaw
```

* * *

## 5) Configurer les variables d'environnement

Créez `.env` à la racine du dépôt.

```
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-now
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/root/.openclaw
OPENCLAW_WORKSPACE_DIR=/root/.openclaw/workspace

GOG_KEYRING_PASSWORD=change-me-now
XDG_CONFIG_HOME=/home/node/.openclaw
```

Générez des secrets robustes :

```bash
openssl rand -hex 32
```

**Ne commitez pas ce fichier.**

* * *

## 6) Configuration Docker Compose

Créez ou mettez à jour `docker-compose.yml`.

```
services:
  openclaw-gateway:
    image: ${OPENCLAW_IMAGE}
    build: .
    restart: unless-stopped
    env_file:
      - .env
    environment:
      - HOME=/home/node
      - NODE_ENV=production
      - TERM=xterm-256color
      - OPENCLAW_GATEWAY_BIND=${OPENCLAW_GATEWAY_BIND}
      - OPENCLAW_GATEWAY_PORT=${OPENCLAW_GATEWAY_PORT}
      - OPENCLAW_GATEWAY_TOKEN=${OPENCLAW_GATEWAY_TOKEN}
      - GOG_KEYRING_PASSWORD=${GOG_KEYRING_PASSWORD}
      - XDG_CONFIG_HOME=${XDG_CONFIG_HOME}
      - PATH=/home/linuxbrew/.linuxbrew/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    volumes:
      - ${OPENCLAW_CONFIG_DIR}:/home/node/.openclaw
      - ${OPENCLAW_WORKSPACE_DIR}:/home/node/.openclaw/workspace
    ports:
      # Recommandé : garder la passerelle en boucle locale sur le VPS ; y accéder via un tunnel SSH.
      # Pour l'exposer publiquement, supprimez le préfixe `127.0.0.1:` et configurez le pare-feu en conséquence.
      - "127.0.0.1:${OPENCLAW_GATEWAY_PORT}:18789"
    command:
      [
        "node",
        "dist/index.js",
        "gateway",
        "--bind",
        "${OPENCLAW_GATEWAY_BIND}",
        "--port",
        "${OPENCLAW_GATEWAY_PORT}",
        "--allow-unconfigured",
      ]
```

`--allow-unconfigured` est uniquement pour la commodité de l'amorçage, ce n'est pas un remplacement pour une configuration correcte de la passerelle. Définissez toujours l'authentification (`gateway.auth.token` ou mot de passe) et utilisez des paramètres de liaison sûrs pour votre déploiement.

* * *

## 7) Intégrer les binaires requis dans l'image (critique)

Installer des binaires à l'intérieur d'un conteneur en cours d'exécution est un piège. Tout ce qui est installé au moment de l'exécution sera perdu au redémarrage. Tous les binaires externes requis par les compétences doivent être installés au moment de la construction de l'image. Les exemples ci-dessous montrent seulement trois binaires courants :

-   `gog` pour l'accès Gmail
-   `goplaces` pour Google Places
-   `wacli` pour WhatsApp

Ce sont des exemples, pas une liste exhaustive. Vous pouvez installer autant de binaires que nécessaire en utilisant le même modèle. Si vous ajoutez plus tard de nouvelles compétences qui dépendent de binaires supplémentaires, vous devez :

1.  Mettre à jour le Dockerfile
2.  Reconstruire l'image
3.  Redémarrer les conteneurs

**Exemple de Dockerfile**

```dockerfile
FROM node:22-bookworm

RUN apt-get update && apt-get install -y socat && rm -rf /var/lib/apt/lists/*

# Exemple binaire 1 : CLI Gmail
RUN curl -L https://github.com/steipete/gog/releases/latest/download/gog_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/gog

# Exemple binaire 2 : CLI Google Places
RUN curl -L https://github.com/steipete/goplaces/releases/latest/download/goplaces_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/goplaces

# Exemple binaire 3 : CLI WhatsApp
RUN curl -L https://github.com/steipete/wacli/releases/latest/download/wacli_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/wacli

# Ajoutez plus de binaires ci-dessous en utilisant le même modèle

WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN corepack enable
RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

* * *

## 8) Construire et lancer

```bash
docker compose build
docker compose up -d openclaw-gateway
```

Vérifiez les binaires :

```bash
docker compose exec openclaw-gateway which gog
docker compose exec openclaw-gateway which goplaces
docker compose exec openclaw-gateway which wacli
```

Résultat attendu :

```
/usr/local/bin/gog
/usr/local/bin/goplaces
/usr/local/bin/wacli
```

* * *

## 9) Vérifier la passerelle

```bash
docker compose logs -f openclaw-gateway
```

Succès :

```ini
[gateway] listening on ws://0.0.0.0:18789
```

Depuis votre ordinateur portable :

```bash
ssh -N -L 18789:127.0.0.1:18789 root@YOUR_VPS_IP
```

Ouvrez : `http://127.0.0.1:18789/` Collez votre jeton de passerelle.

* * *

## Ce qui persiste où (source de vérité)

OpenClaw s'exécute dans Docker, mais Docker n'est pas la source de vérité. Tout état de longue durée doit survivre aux redémarrages, reconstructions et arrêts.

| Composant | Emplacement | Mécanisme de persistance | Notes |
| --- | --- | --- | --- |
| Configuration de la passerelle | `/home/node/.openclaw/` | Montage de volume hôte | Inclut `openclaw.json`, jetons |
| Profils d'authentification des modèles | `/home/node/.openclaw/` | Montage de volume hôte | Jetons OAuth, clés API |
| Configurations des compétences | `/home/node/.openclaw/skills/` | Montage de volume hôte | État au niveau de la compétence |
| Espace de travail de l'agent | `/home/node/.openclaw/workspace/` | Montage de volume hôte | Code et artefacts de l'agent |
| Session WhatsApp | `/home/node/.openclaw/` | Montage de volume hôte | Préserve la connexion QR |
| Trousseau Gmail | `/home/node/.openclaw/` | Volume hôte + mot de passe | Requiert `GOG_KEYRING_PASSWORD` |
| Binaires externes | `/usr/local/bin/` | Image Docker | Doivent être intégrés à la construction |
| Runtime Node | Système de fichiers du conteneur | Image Docker | Reconstruit à chaque construction d'image |
| Paquets OS | Système de fichiers du conteneur | Image Docker | Ne pas installer au moment de l'exécution |
| Conteneur Docker | Éphémère | Redémarrable | Sûr à détruire |

* * *

## Infrastructure as Code (Terraform)

Pour les équipes préférant les flux de travail d'infrastructure-as-code, une configuration Terraform maintenue par la communauté fournit :

-   Configuration Terraform modulaire avec gestion d'état distant
-   Provisionnement automatisé via cloud-init
-   Scripts de déploiement (amorçage, déploiement, sauvegarde/restauration)
-   Durcissement de la sécurité (pare-feu, UFW, accès SSH uniquement)
-   Configuration de tunnel SSH pour l'accès à la passerelle

**Dépôts :**

-   Infrastructure : [openclaw-terraform-hetzner](https://github.com/andreesg/openclaw-terraform-hetzner)
-   Configuration Docker : [openclaw-docker-config](https://github.com/andreesg/openclaw-docker-config)

Cette approche complète la configuration Docker ci-dessus avec des déploiements reproductibles, une infrastructure versionnée et une reprise après sinistre automatisée.

> **Note :** Maintenu par la communauté. Pour les problèmes ou contributions, voir les liens des dépôts ci-dessus.

[Fly.io](./fly.md)[GCP](./gcp.md)