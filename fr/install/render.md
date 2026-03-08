

  Hébergement et déploiement

  
# Déployer sur Render

Déployez OpenClaw sur Render en utilisant l'Infrastructure as Code. Le fichier `render.yaml` Blueprint inclus définit toute votre pile de manière déclarative : service, disque, variables d'environnement, vous permettant ainsi de déployer en un seul clic et de versionner votre infrastructure avec votre code.

## Prérequis

-   Un [compte Render](https://render.com) (niveau gratuit disponible)
-   Une clé API de votre [fournisseur de modèle](../providers.md) préféré

## Déployer avec un Blueprint Render

[Déployer sur Render](https://render.com/deploy?repo=https://github.com/openclaw/openclaw) Cliquer sur ce lien va :

1.  Créer un nouveau service Render à partir du Blueprint `render.yaml` à la racine de ce dépôt.
2.  Vous inviter à définir le `SETUP_PASSWORD`
3.  Construire l'image Docker et déployer

Une fois déployé, l'URL de votre service suit le modèle `https://<nom-du-service>.onrender.com`.

## Comprendre le Blueprint

Les Blueprints Render sont des fichiers YAML qui définissent votre infrastructure. Le fichier `render.yaml` dans ce dépôt configure tout ce qui est nécessaire pour exécuter OpenClaw :

```yaml
services:
  - type: web
    name: openclaw
    runtime: docker
    plan: starter
    healthCheckPath: /health
    envVars:
      - key: PORT
        value: "8080"
      - key: SETUP_PASSWORD
        sync: false # demande lors du déploiement
      - key: OPENCLAW_STATE_DIR
        value: /data/.openclaw
      - key: OPENCLAW_WORKSPACE_DIR
        value: /data/workspace
      - key: OPENCLAW_GATEWAY_TOKEN
        generateValue: true # génère automatiquement un jeton sécurisé
    disk:
      name: openclaw-data
      mountPath: /data
      sizeGB: 1
```

Fonctionnalités clés du Blueprint utilisées :

| Fonctionnalité | Objectif |
| --- | --- |
| `runtime: docker` | Construit à partir du Dockerfile du dépôt |
| `healthCheckPath` | Render surveille `/health` et redémarre les instances défaillantes |
| `sync: false` | Demande la valeur pendant le déploiement (secrets) |
| `generateValue: true` | Génère automatiquement une valeur cryptographiquement sécurisée |
| `disk` | Stockage persistant qui survit aux redéploiements |

## Choisir un plan

| Plan | Arrêt automatique | Disque | Idéal pour |
| --- | --- | --- | --- |
| Gratuit | Après 15 min d'inactivité | Non disponible | Tests, démos |
| Starter | Jamais | 1GB+ | Usage personnel, petites équipes |
| Standard+ | Jamais | 1GB+ | Production, canaux multiples |

Le Blueprint utilise par défaut `starter`. Pour utiliser le niveau gratuit, changez `plan: free` dans le `render.yaml` de votre fork (mais notez : l'absence de disque persistant signifie que la configuration est réinitialisée à chaque déploiement).

## Après le déploiement

### Compléter l'assistant de configuration

1.  Accédez à `https://<votre-service>.onrender.com/setup`
2.  Entrez votre `SETUP_PASSWORD`
3.  Sélectionnez un fournisseur de modèle et collez votre clé API
4.  Configurez éventuellement les canaux de messagerie (Telegram, Discord, Slack)
5.  Cliquez sur **Exécuter la configuration**

### Accéder à l'interface de contrôle

Le tableau de bord web est disponible à l'adresse `https://<votre-service>.onrender.com/openclaw`.

## Fonctionnalités du tableau de bord Render

### Journaux

Consultez les journaux en temps réel dans **Tableau de bord → votre service → Journaux**. Filtrez par :

-   Journaux de construction (création de l'image Docker)
-   Journaux de déploiement (démarrage du service)
-   Journaux d'exécution (sortie de l'application)

### Accès shell

Pour le débogage, ouvrez une session shell via **Tableau de bord → votre service → Shell**. Le disque persistant est monté sur `/data`.

### Variables d'environnement

Modifiez les variables dans **Tableau de bord → votre service → Environnement**. Les changements déclenchent un redéploiement automatique.

### Déploiement automatique

Si vous utilisez le dépôt OpenClaw d'origine, Render ne redéploiera pas automatiquement votre OpenClaw. Pour le mettre à jour, exécutez une synchronisation manuelle du Blueprint depuis le tableau de bord.

## Domaine personnalisé

1.  Allez dans **Tableau de bord → votre service → Paramètres → Domaines personnalisés**
2.  Ajoutez votre domaine
3.  Configurez le DNS comme indiqué (CNAME vers `*.onrender.com`)
4.  Render provisionne un certificat TLS automatiquement

## Mise à l'échelle

Render prend en charge la mise à l'échelle horizontale et verticale :

-   **Verticale** : Changez de plan pour obtenir plus de CPU/RAM
-   **Horizontale** : Augmentez le nombre d'instances (plan Standard et supérieur)

Pour OpenClaw, la mise à l'échelle verticale est généralement suffisante. La mise à l'échelle horizontale nécessite des sessions persistantes ou une gestion d'état externe.

## Sauvegardes et migration

Exportez votre configuration et votre espace de travail à tout moment :

```
https://<votre-service>.onrender.com/setup/export
```

Cela télécharge une sauvegarde portable que vous pouvez restaurer sur n'importe quel hôte OpenClaw.

## Dépannage

### Le service ne démarre pas

Vérifiez les journaux de déploiement dans le tableau de bord Render. Problèmes courants :

-   `SETUP_PASSWORD` manquant — le Blueprint le demande, mais vérifiez qu'il est défini
-   Incompatibilité de port — assurez-vous que `PORT=8080` correspond au port exposé dans le Dockerfile

### Démarrages à froid lents (niveau gratuit)

Les services du niveau gratuit s'arrêtent après 15 minutes d'inactivité. La première requête après l'arrêt prend quelques secondes pendant que le conteneur démarre. Passez au plan Starter pour un service toujours actif.

### Perte de données après un redéploiement

Cela se produit sur le niveau gratuit (pas de disque persistant). Passez à un plan payant, ou exportez régulièrement votre configuration via `/setup/export`.

### Échecs des vérifications de santé

Render attend une réponse 200 de `/health` dans les 30 secondes. Si les constructions réussissent mais que les déploiements échouent, le service peut mettre trop de temps à démarrer. Vérifiez :

-   Les journaux de construction pour des erreurs
-   Si le conteneur s'exécute localement avec `docker build && docker run`

[Déployer sur Railway](./railway.md)[Déployer sur Northflank](./northflank.md)