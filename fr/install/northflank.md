

  Hébergement et déploiement

  
# Déployer sur Northflank

Déployez OpenClaw sur Northflank avec un modèle en un clic et terminez la configuration dans votre navigateur. C'est la méthode la plus simple « sans terminal sur le serveur » : Northflank exécute la Passerelle pour vous, et vous configurez tout via l'assistant web `/setup`.

## Comment commencer

1.  Cliquez sur [Déployer OpenClaw](https://northflank.com/stacks/deploy-openclaw) pour ouvrir le modèle.
2.  Créez un [compte sur Northflank](https://app.northflank.com/signup) si vous n'en avez pas déjà un.
3.  Cliquez sur **Déployer OpenClaw maintenant**.
4.  Définissez la variable d'environnement requise : `SETUP_PASSWORD`.
5.  Cliquez sur **Déployer la pile** pour construire et exécuter le modèle OpenClaw.
6.  Attendez que le déploiement se termine, puis cliquez sur **Voir les ressources**.
7.  Ouvrez le service OpenClaw.
8.  Ouvrez l'URL publique d'OpenClaw et terminez la configuration sur `/setup`.
9.  Ouvrez l'interface de contrôle sur `/openclaw`.

## Ce que vous obtenez

-   Passerelle OpenClaw hébergée + Interface de contrôle
-   Assistant de configuration web sur `/setup` (pas de commandes terminal)
-   Stockage persistant via le Volume Northflank (`/data`) pour que la configuration/les identifiants/l'espace de travail survivent aux redéploiements

## Déroulement de la configuration

1.  Visitez `https://<votre-domaine-northflank>/setup` et entrez votre `SETUP_PASSWORD`.
2.  Choisissez un modèle/fournisseur d'authentification et collez votre clé.
3.  (Optionnel) Ajoutez les jetons Telegram/Discord/Slack.
4.  Cliquez sur **Exécuter la configuration**.
5.  Ouvrez l'interface de contrôle sur `https://<votre-domaine-northflank>/openclaw`

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
4.  **Activer MESSAGE CONTENT INTENT** sous Bot → Intents du Gateway Privilégiés (requis, sinon le bot plantera au démarrage)
5.  Copiez le **Jeton du Bot** et collez-le dans `/setup`
6.  Invitez le bot sur votre serveur (Générateur d'URL OAuth2 ; portées : `bot`, `applications.commands`)

[Déployer sur Render](./render.md)[Canaux de développement](./development-channels.md)

---