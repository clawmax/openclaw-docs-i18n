

  Plateformes de messagerie

  
# Google Chat

Statut : prêt pour les messages privés + espaces via les webhooks de l'API Google Chat (HTTP uniquement).

## Installation rapide (débutant)

1.  Créez un projet Google Cloud et activez l'**API Google Chat**.
    -   Allez à : [Identifiants de l'API Google Chat](https://console.cloud.google.com/apis/api/chat.googleapis.com/credentials)
    -   Activez l'API si elle ne l'est pas déjà.
2.  Créez un **Compte de service** :
    -   Cliquez sur **Créer des identifiants** > **Compte de service**.
    -   Nommez-le comme vous voulez (par ex., `openclaw-chat`).
    -   Laissez les permissions vides (cliquez sur **Continuer**).
    -   Laissez les principaux avec accès vides (cliquez sur **Terminé**).
3.  Créez et téléchargez la **Clé JSON** :
    -   Dans la liste des comptes de service, cliquez sur celui que vous venez de créer.
    -   Allez dans l'onglet **Clés**.
    -   Cliquez sur **Ajouter une clé** > **Créer une nouvelle clé**.
    -   Sélectionnez **JSON** et cliquez sur **Créer**.
4.  Stockez le fichier JSON téléchargé sur votre hôte de passerelle (par ex., `~/.openclaw/googlechat-service-account.json`).
5.  Créez une application Google Chat dans la [Configuration Chat de la console Google Cloud](https://console.cloud.google.com/apis/api/chat.googleapis.com/hangouts-chat) :
    -   Remplissez les **Informations sur l'application** :
        -   **Nom de l'application** : (par ex. `OpenClaw`)
        -   **URL de l'avatar** : (par ex. `https://openclaw.ai/logo.png`)
        -   **Description** : (par ex. `Assistant IA Personnel`)
    -   Activez les **Fonctionnalités interactives**.
    -   Sous **Fonctionnalités**, cochez **Rejoindre les espaces et les conversations de groupe**.
    -   Sous **Paramètres de connexion**, sélectionnez **URL du point de terminaison HTTP**.
    -   Sous **Déclencheurs**, sélectionnez **Utiliser une URL de point de terminaison HTTP commune pour tous les déclencheurs** et définissez-la sur l'URL publique de votre passerelle suivie de `/googlechat`.
        -   *Astuce : Exécutez `openclaw status` pour trouver l'URL publique de votre passerelle.*
    -   Sous **Visibilité**, cochez **Rendre cette application Chat disponible à des personnes et groupes spécifiques dans **.
    -   Entrez votre adresse e-mail (par ex. `user@example.com`) dans la zone de texte.
    -   Cliquez sur **Enregistrer** en bas.
6.  **Activez le statut de l'application** :
    -   Après l'enregistrement, **rafraîchissez la page**.
    -   Cherchez la section **Statut de l'application** (généralement en haut ou en bas après l'enregistrement).
    -   Changez le statut en **En direct - disponible pour les utilisateurs**.
    -   Cliquez à nouveau sur **Enregistrer**.
7.  Configurez OpenClaw avec le chemin du compte de service + l'audience du webhook :
    -   Variable d'env : `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE=/chemin/vers/service-account.json`
    -   Ou config : `channels.googlechat.serviceAccountFile: "/chemin/vers/service-account.json"`.
8.  Définissez le type d'audience du webhook + sa valeur (correspond à la configuration de votre application Chat).
9.  Démarrez la passerelle. Google Chat enverra des requêtes POST à votre chemin de webhook.

## Ajouter à Google Chat

Une fois la passerelle en cours d'exécution et votre e-mail ajouté à la liste de visibilité :

1.  Allez sur [Google Chat](https://chat.google.com/).
2.  Cliquez sur l'icône **+** (plus) à côté de **Messages directs**.
3.  Dans la barre de recherche (où vous ajoutez habituellement des personnes), tapez le **Nom de l'application** que vous avez configuré dans la console Google Cloud.
    -   **Note** : Le bot n'apparaîtra *pas* dans la liste de navigation "Marketplace" car c'est une application privée. Vous devez le rechercher par son nom.
4.  Sélectionnez votre bot dans les résultats.
5.  Cliquez sur **Ajouter** ou **Discuter** pour démarrer une conversation 1:1.
6.  Envoyez "Bonjour" pour déclencher l'assistant !

## URL publique (Webhook uniquement)

Les webhooks Google Chat nécessitent un point de terminaison HTTPS public. Pour la sécurité, **exposez uniquement le chemin `/googlechat`** à Internet. Gardez le tableau de bord OpenClaw et les autres points de terminaison sensibles sur votre réseau privé.

### Option A : Tailscale Funnel (Recommandé)

Utilisez Tailscale Serve pour le tableau de bord privé et Funnel pour le chemin de webhook public. Cela garde `/` privé tout en exposant uniquement `/googlechat`.

1.  **Vérifiez à quelle adresse votre passerelle est liée :**
    
    Copier
    
    ```bash
    ss -tlnp | grep 18789
    ```
    
    Notez l'adresse IP (par ex., `127.0.0.1`, `0.0.0.0`, ou votre IP Tailscale comme `100.x.x.x`).
2.  **Exposez le tableau de bord uniquement au tailnet (port 8443) :**
    
    Copier
    
    ```bash
    # Si lié à localhost (127.0.0.1 ou 0.0.0.0) :
    tailscale serve --bg --https 8443 http://127.0.0.1:18789
    
    # Si lié uniquement à l'IP Tailscale (par ex., 100.106.161.80) :
    tailscale serve --bg --https 8443 http://100.106.161.80:18789
    ```
    
3.  **Exposez uniquement le chemin du webhook publiquement :**
    
    Copier
    
    ```bash
    # Si lié à localhost (127.0.0.1 ou 0.0.0.0) :
    tailscale funnel --bg --set-path /googlechat http://127.0.0.1:18789/googlechat
    
    # Si lié uniquement à l'IP Tailscale (par ex., 100.106.161.80) :
    tailscale funnel --bg --set-path /googlechat http://100.106.161.80:18789/googlechat
    ```
    
4.  **Autorisez l'accès Funnel au nœud :** Si demandé, visitez l'URL d'autorisation affichée dans la sortie pour activer Funnel pour ce nœud dans la politique de votre tailnet.
5.  **Vérifiez la configuration :**
    
    Copier
    
    ```
    tailscale serve status
    tailscale funnel status
    ```
    

Votre URL de webhook publique sera : `https://<nom-du-nœud>..ts.net/googlechat` Votre tableau de bord privé reste uniquement accessible au tailnet : `https://<nom-du-nœud>..ts.net:8443/` Utilisez l'URL publique (sans `:8443`) dans la configuration de l'application Google Chat.

> Note : Cette configuration persiste après les redémarrages. Pour la supprimer plus tard, exécutez `tailscale funnel reset` et `tailscale serve reset`.

### Option B : Reverse Proxy (Caddy)

Si vous utilisez un reverse proxy comme Caddy, proxyez uniquement le chemin spécifique :

```
your-domain.com {
    reverse_proxy /googlechat* localhost:18789
}
```

Avec cette configuration, toute requête vers `your-domain.com/` sera ignorée ou renverra une erreur 404, tandis que `your-domain.com/googlechat` sera acheminée en toute sécurité vers OpenClaw.

### Option C : Tunnel Cloudflare

Configurez les règles d'ingress de votre tunnel pour router uniquement le chemin du webhook :

-   **Chemin** : `/googlechat` -> `http://localhost:18789/googlechat`
-   **Règle par défaut** : HTTP 404 (Non trouvé)

## Fonctionnement

1.  Google Chat envoie des requêtes POST de webhook à la passerelle. Chaque requête inclut un en-tête `Authorization: Bearer `.
    -   OpenClaw vérifie l'authentification Bearer avant de lire/analyser les corps complets des webhooks lorsque l'en-tête est présent.
    -   Les requêtes de complément Google Workspace qui contiennent `authorizationEventObject.systemIdToken` dans le corps sont prises en charge via un budget de corps pré-authentification plus strict.
2.  OpenClaw vérifie le jeton par rapport au `audienceType` + `audience` configuré :
    -   `audienceType: "app-url"` → l'audience est votre URL de webhook HTTPS.
    -   `audienceType: "project-number"` → l'audience est le numéro du projet Cloud.
3.  Les messages sont acheminés par espace :
    -   Les messages privés utilisent la clé de session `agent::googlechat:dm:`.
    -   Les espaces utilisent la clé de session `agent::googlechat:group:`.
4.  L'accès en message privé est par appairage par défaut. Les expéditeurs inconnus reçoivent un code d'appairage ; approuvez avec :
    -   `openclaw pairing approve googlechat `
5.  Les espaces de groupe nécessitent une mention @ par défaut. Utilisez `botUser` si la détection de mention nécessite le nom d'utilisateur de l'application.

## Cibles

Utilisez ces identifiants pour la livraison et les listes autorisées :

-   Messages directs : `users/` (recommandé).
-   L'e-mail brut `name@example.com` est mutable et n'est utilisé que pour la correspondance directe dans les listes autorisées lorsque `channels.googlechat.dangerouslyAllowNameMatching: true`.
-   Obsolète : `users/` est traité comme un identifiant utilisateur, pas comme une liste autorisée d'e-mails.
-   Espaces : `spaces/`.

## Points clés de configuration

```json
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      // or serviceAccountRef: { source: "file", provider: "filemain", id: "/channels/googlechat/serviceAccount" }
      audienceType: "app-url",
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890", // optionnel ; aide à la détection des mentions
      dm: {
        policy: "pairing",
        allowFrom: ["users/1234567890"],
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": {
          allow: true,
          requireMention: true,
          users: ["users/1234567890"],
          systemPrompt: "Réponses courtes uniquement.",
        },
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```

Notes :

-   Les identifiants du compte de service peuvent également être passés en ligne avec `serviceAccount` (chaîne JSON).
-   `serviceAccountRef` est également pris en charge (SecretRef env/fichier), y compris les références par compte sous `channels.googlechat.accounts..serviceAccountRef`.
-   Le chemin de webhook par défaut est `/googlechat` si `webhookPath` n'est pas défini.
-   `dangerouslyAllowNameMatching` réactive la correspondance de principal d'e-mail mutable pour les listes autorisées (mode de compatibilité de secours).
-   Les réactions sont disponibles via l'outil `reactions` et `channels action` lorsque `actions.reactions` est activé.
-   `typingIndicator` prend en charge `none`, `message` (par défaut) et `reaction` (la réaction nécessite l'OAuth utilisateur).
-   Les pièces jointes sont téléchargées via l'API Chat et stockées dans le pipeline média (taille limitée par `mediaMaxMb`).

Détails de référence des secrets : [Gestion des secrets](../gateway/secrets.md).

## Dépannage

### 405 Méthode non autorisée

Si Google Cloud Logs Explorer affiche des erreurs comme :

```bash
status code: 405, reason phrase: HTTP error response: HTTP/1.1 405 Method Not Allowed
```

Cela signifie que le gestionnaire de webhook n'est pas enregistré. Causes courantes :

1.  **Canal non configuré** : La section `channels.googlechat` est manquante dans votre configuration. Vérifiez avec :
    
    Copier
    
    ```bash
    openclaw config get channels.googlechat
    ```
    
    Si cela renvoie "Chemin de configuration non trouvé", ajoutez la configuration (voir [Points clés de configuration](#config-highlights)).
2.  **Plugin non activé** : Vérifiez le statut du plugin :
    
    Copier
    
    ```bash
    openclaw plugins list | grep googlechat
    ```
    
    S'il affiche "disabled", ajoutez `plugins.entries.googlechat.enabled: true` à votre configuration.
3.  **Passerelle non redémarrée** : Après avoir ajouté la configuration, redémarrez la passerelle :
    
    Copier
    
    ```bash
    openclaw gateway restart
    ```
    

Vérifiez que le canal est en cours d'exécution :

```bash
openclaw channels status
# Devrait afficher : Google Chat default: enabled, configured, ...
```

### Autres problèmes

-   Vérifiez `openclaw channels status --probe` pour les erreurs d'authentification ou la configuration d'audience manquante.
-   Si aucun message n'arrive, confirmez l'URL du webhook de l'application Chat + les abonnements aux événements.
-   Si la mention bloque les réponses, définissez `botUser` sur le nom de ressource utilisateur de l'application et vérifiez `requireMention`.
-   Utilisez `openclaw logs --follow` lors de l'envoi d'un message test pour voir si les requêtes atteignent la passerelle.

Documentation associée :

-   [Configuration de la passerelle](../gateway/configuration.md)
-   [Sécurité](../gateway/security.md)
-   [Réactions](../tools/reactions.md)

[Feishu](./feishu.md)[iMessage](./imessage.md)