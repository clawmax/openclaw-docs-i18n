

  Hébergement et déploiement

  
# Déployer sur Railway

Déployez OpenClaw sur Railway avec un modèle en un clic et terminez la configuration dans votre navigateur. C'est le chemin le plus simple « sans terminal sur le serveur » : Railway exécute la Passerelle pour vous, et vous configurez tout via l'assistant web `/setup`.

## Liste de contrôle rapide (nouveaux utilisateurs)

1.  Cliquez sur **Déployer sur Railway** (ci-dessous).
2.  Ajoutez un **Volume** monté sur `/data`.
3.  Définissez les **Variables** requises (au moins `SETUP_PASSWORD`).
4.  Activez le **Proxy HTTP** sur le port `8080`.
5.  Ouvrez `https://<votre-domaine-railway>/setup` et terminez l'assistant.

## Déploiement en un clic

[Déployer sur Railway](https://railway.com/deploy/clawdbot-railway-template) Après le déploiement, trouvez votre URL publique dans **Railway → votre service → Paramètres → Domaines**. Railway vous donnera soit :

-   un domaine généré (souvent `https://<quelque-chose>.up.railway.app`), soit
-   utilisera votre domaine personnalisé si vous en avez attaché un.

Ensuite, ouvrez :

-   `https://<votre-domaine-railway>/setup` — assistant de configuration (protégé par mot de passe)
-   `https://<votre-domaine-railway>/openclaw` — Interface de contrôle

## Ce que vous obtenez

-   Passerelle OpenClaw hébergée + Interface de contrôle
-   Assistant de configuration web à `/setup` (pas de commandes terminal)
-   Stockage persistant via le Volume Railway (`/data`) pour que la configuration/les identifiants/l'espace de travail survivent aux redéploiements
-   Export de sauvegarde à `/setup/export` pour migrer hors de Railway plus tard

## Paramètres Railway requis

### Réseau public

Activez le **Proxy HTTP** pour le service.

-   Port : `8080`

### Volume (requis)

Attachez un volume monté sur :

-   `/data`

### Variables

Définissez ces variables sur le service :

-   `SETUP_PASSWORD` (requis)
-   `PORT=8080` (requis — doit correspondre au port dans Réseau public)
-   `OPENCLAW_STATE_DIR=/data/.openclaw` (recommandé)
-   `OPENCLAW_WORKSPACE_DIR=/data/workspace` (recommandé)
-   `OPENCLAW_GATEWAY_TOKEN` (recommandé ; traitez-le comme un secret administrateur)

## Flux de configuration

1.  Visitez `https://<votre-domaine-railway>/setup` et entrez votre `SETUP_PASSWORD`.
2.  Choisissez un modèle/fournisseur d'authentification et collez votre clé.
3.  (Optionnel) Ajoutez les jetons Telegram/Discord/Slack.
4.  Cliquez sur **Exécuter la configuration**.

Si les messages privés Telegram sont configurés pour l'appairage, l'assistant de configuration peut approuver le code d'appairage.

## Obtenir les jetons de discussion

### Jeton de bot Telegram

1.  Envoyez un message à `@BotFather` dans Telegram
2.  Exécutez `/newbot`
3.  Copiez le jeton (ressemble à `123456789:AA...`)
4.  Collez-le dans `/setup`

### Jeton de bot Discord

1.  Allez sur [https://discord.com/developers/applications](https://discord.com/developers/applications)
2.  **Nouvelle Application** → choisissez un nom
3.  **Bot** → **Ajouter un Bot**
4.  **Activer l'INTENTION DE CONTENU DES MESSAGES** sous Bot → Intentions de passerelle privilégiées (requis, sinon le bot plantera au démarrage)
5.  Copiez le **Jeton du Bot** et collez-le dans `/setup`
6.  Invitez le bot sur votre serveur (Générateur d'URL OAuth2 ; portées : `bot`, `applications.commands`)

## Sauvegardes & migration

Téléchargez une sauvegarde à :

-   `https://<votre-domaine-railway>/setup/export`

Cela exporte l'état OpenClaw + l'espace de travail afin que vous puissiez migrer vers un autre hôte sans perdre la configuration ou la mémoire.

[exe.dev](./exe-dev.md)[Déployer sur Render](./render.md)

---