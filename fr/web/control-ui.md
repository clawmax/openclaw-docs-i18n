title: "Guide de l'interface utilisateur de contrôle OpenClaw pour l'interface web et le tableau de bord"
description: "Apprenez à utiliser le tableau de bord web de l'interface utilisateur de contrôle OpenClaw pour gérer les agents IA, configurer les canaux, gérer les tâches cron et vous connecter via Tailscale ou localhost."
keywords: ["interface utilisateur de contrôle openclaw", "tableau de bord web", "websocket de la passerelle", "intégration tailscale", "appairage d'appareil", "gestion d'agent IA", "tâches cron", "interface de chat"]
---

  Interfaces web

  
# Interface de contrôle

L'interface de contrôle est une petite **application monopage Vite + Lit** servie par la Passerelle :

-   par défaut : `http://<hôte>:18789/`
-   préfixe optionnel : définir `gateway.controlUi.basePath` (par ex. `/openclaw`)

Elle parle **directement au WebSocket de la Passerelle** sur le même port.

## Ouverture rapide (local)

Si la Passerelle fonctionne sur le même ordinateur, ouvrez :

-   [http://127.0.0.1:18789/](http://127.0.0.1:18789/) (ou [http://localhost:18789/](http://localhost:18789/))

Si la page ne se charge pas, démarrez d'abord la Passerelle : `openclaw gateway`. L'authentification est fournie lors de la poignée de main WebSocket via :

-   `connect.params.auth.token`
-   `connect.params.auth.password` Le panneau des paramètres du tableau de bord vous permet de stocker un jeton ; les mots de passe ne sont pas conservés. L'assistant de configuration génère un jeton de passerelle par défaut, collez-le donc ici lors de la première connexion.

## Appairage d'appareil (première connexion)

Lorsque vous vous connectez à l'interface de contrôle depuis un nouveau navigateur ou appareil, la Passerelle nécessite une **approbation d'appairage unique** — même si vous êtes sur le même Tailnet avec `gateway.auth.allowTailscale: true`. Il s'agit d'une mesure de sécurité pour empêcher les accès non autorisés. **Ce que vous verrez :** "déconnecté (1008) : appairage requis" **Pour approuver l'appareil :**

```bash
# Lister les demandes en attente
openclaw devices list

# Approuver par ID de demande
openclaw devices approve <requestId>
```

Une fois approuvé, l'appareil est mémorisé et ne nécessitera pas de ré-approbation à moins que vous ne le révoquiez avec `openclaw devices revoke --device  --role `. Voir [CLI Appareils](../cli/devices.md) pour la rotation et la révocation des jetons. **Notes :**

-   Les connexions locales (`127.0.0.1`) sont automatiquement approuvées.
-   Les connexions distantes (LAN, Tailnet, etc.) nécessitent une approbation explicite.
-   Chaque profil de navigateur génère un ID d'appareil unique, donc changer de navigateur ou effacer les données du navigateur nécessitera un ré-appairage.

## Support des langues

L'interface de contrôle peut se localiser au premier chargement en fonction de la langue de votre navigateur, et vous pouvez la modifier ultérieurement via le sélecteur de langue dans la carte Accès.

-   Langues prises en charge : `en`, `zh-CN`, `zh-TW`, `pt-BR`, `de`, `es`
-   Les traductions non anglaises sont chargées à la demande dans le navigateur.
-   La langue sélectionnée est enregistrée dans le stockage du navigateur et réutilisée lors des visites futures.
-   Les clés de traduction manquantes reviennent à l'anglais.

## Ce qu'elle peut faire (aujourd'hui)

-   Discuter avec le modèle via le WS de la Passerelle (`chat.history`, `chat.send`, `chat.abort`, `chat.inject`)
-   Diffuser les appels d'outils + cartes de sortie d'outils en direct dans le Chat (événements de l'agent)
-   Canaux : WhatsApp/Telegram/Discord/Slack + canaux de plugin (Mattermost, etc.) statut + connexion QR + configuration par canal (`channels.status`, `web.login.*`, `config.patch`)
-   Instances : liste de présence + actualisation (`system-presence`)
-   Sessions : liste + remplacements de réflexion/verbeux par session (`sessions.list`, `sessions.patch`)
-   Tâches cron : liste/ajout/édition/exécution/activation/désactivation + historique d'exécution (`cron.*`)
-   Compétences : statut, activation/désactivation, installation, mises à jour des clés API (`skills.*`)
-   Nœuds : liste + capacités (`node.list`)
-   Approbations d'exécution : modifier les listes d'autorisation de la passerelle ou des nœuds + demander la politique pour `exec host=gateway/node` (`exec.approvals.*`)
-   Configuration : voir/modifier `~/.openclaw/openclaw.json` (`config.get`, `config.set`)
-   Configuration : appliquer + redémarrer avec validation (`config.apply`) et réveiller la dernière session active
-   Les écritures de configuration incluent une protection par hachage de base pour éviter les écrasements d'éditions concurrentes
-   Schéma de configuration + rendu de formulaire (`config.schema`, incluant les schémas de plugin et de canal) ; l'éditeur JSON brut reste disponible
-   Débogage : instantanés de statut/santé/modèles + journal des événements + appels RPC manuels (`status`, `health`, `models.list`)
-   Journaux : suivi en direct des journaux de fichiers de la passerelle avec filtre/export (`logs.tail`)
-   Mise à jour : exécuter une mise à jour de paquet/git + redémarrer (`update.run`) avec un rapport de redémarrage

Notes du panneau des tâches cron :

-   Pour les tâches isolées, la livraison par défaut est l'annonce du résumé. Vous pouvez passer à aucune si vous voulez des exécutions internes uniquement.
-   Les champs canal/cible apparaissent lorsque l'annonce est sélectionnée.
-   Le mode webhook utilise `delivery.mode = "webhook"` avec `delivery.to` défini sur une URL de webhook HTTP(S) valide.
-   Pour les tâches de session principale, les modes de livraison webhook et aucun sont disponibles.
-   Les contrôles d'édition avancés incluent la suppression après exécution, l'annulation du remplacement de l'agent, les options cron exactes/échelonnées, les remplacements de modèle/réflexion de l'agent, et les bascules de livraison au mieux.
-   La validation du formulaire est en ligne avec des erreurs au niveau des champs ; les valeurs invalides désactivent le bouton de sauvegarde jusqu'à correction.
-   Définissez `cron.webhookToken` pour envoyer un jeton porteur dédié, s'il est omis, le webhook est envoyé sans en-tête d'authentification.
-   Solution de secours obsolète : les tâches héritées stockées avec `notify: true` peuvent toujours utiliser `cron.webhook` jusqu'à migration.

## Comportement du chat

-   `chat.send` est **non bloquant** : il accuse réception immédiatement avec `{ runId, status: "started" }` et la réponse est diffusée via les événements `chat`.
-   Le renvoi avec la même `idempotencyKey` renvoie `{ status: "in_flight" }` pendant l'exécution, et `{ status: "ok" }` après achèvement.
-   Les réponses `chat.history` sont limitées en taille pour la sécurité de l'interface utilisateur. Lorsque les entrées de transcription sont trop volumineuses, la Passerelle peut tronquer les champs de texte longs, omettre les blocs de métadonnées lourds et remplacer les messages trop volumineux par un espace réservé (`[chat.history omis : message trop volumineux]`).
-   `chat.inject` ajoute une note de l'assistant à la transcription de la session et diffuse un événement `chat` pour les mises à jour de l'interface utilisateur uniquement (pas d'exécution d'agent, pas de livraison de canal).
-   Arrêter :
    -   Cliquez sur **Arrêter** (appelle `chat.abort`)
    -   Tapez `/stop` (ou des phrases d'interruption autonomes comme `stop`, `stop action`, `stop run`, `stop openclaw`, `please stop`) pour interrompre hors bande
    -   `chat.abort` prend en charge `{ sessionKey }` (pas de `runId`) pour interrompre toutes les exécutions actives de cette session
-   Rétention partielle après interruption :
    -   Lorsqu'une exécution est interrompue, le texte partiel de l'assistant peut toujours être affiché dans l'interface utilisateur
    -   La Passerelle conserve le texte partiel de l'assistant interrompu dans l'historique de transcription lorsqu'il existe une sortie mise en mémoire tampon
    -   Les entrées conservées incluent des métadonnées d'interruption afin que les consommateurs de transcription puissent distinguer les parties interrompues de la sortie d'achèvement normale

## Accès Tailnet (recommandé)

### Serve Tailscale intégré (préféré)

Gardez la Passerelle en boucle locale et laissez Tailscale Serve la proxifier avec HTTPS :

```bash
openclaw gateway --tailscale serve
```

Ouvrez :

-   `https:///` (ou votre `gateway.controlUi.basePath` configuré)

Par défaut, les requêtes Serve de l'interface de contrôle/WebSocket peuvent s'authentifier via les en-têtes d'identité Tailscale (`tailscale-user-login`) lorsque `gateway.auth.allowTailscale` est `true`. OpenClaw vérifie l'identité en résolvant l'adresse `x-forwarded-for` avec `tailscale whois` et en la faisant correspondre à l'en-tête, et n'accepte celles-ci que lorsque la requête atteint la boucle locale avec les en-têtes `x-forwarded-*` de Tailscale. Définissez `gateway.auth.allowTailscale: false` (ou forcez `gateway.auth.mode: "password"`) si vous voulez exiger un jeton/mot de passe même pour le trafic Serve. L'authentification Serve sans jeton suppose que l'hôte de la passerelle est de confiance. Si du code local non fiable peut s'exécuter sur cet hôte, exigez une authentification par jeton/mot de passe.

### Lier à tailnet + jeton

```bash
openclaw gateway --bind tailnet --token "$(openssl rand -hex 32)"
```

Puis ouvrez :

-   `http://<tailscale-ip>:18789/` (ou votre `gateway.controlUi.basePath` configuré)

Collez le jeton dans les paramètres de l'interface utilisateur (envoyé en tant que `connect.params.auth.token`).

## HTTP non sécurisé

Si vous ouvrez le tableau de bord en HTTP simple (`http://<lan-ip>` ou `http://<tailscale-ip>`), le navigateur fonctionne dans un **contexte non sécurisé** et bloque WebCrypto. Par défaut, OpenClaw **bloque** les connexions de l'interface de contrôle sans identité d'appareil. **Solution recommandée :** utilisez HTTPS (Tailscale Serve) ou ouvrez l'interface localement :

-   `https:///` (Serve)
-   `http://127.0.0.1:18789/` (sur l'hôte de la passerelle)

**Comportement de la bascule d'authentification non sécurisée :**

```json
{
  gateway: {
    controlUi: { allowInsecureAuth: true },
    bind: "tailnet",
    auth: { mode: "token", token: "replace-me" },
  },
}
```

`allowInsecureAuth` ne contourne pas l'identité de l'appareil de l'interface de contrôle ni les vérifications d'appairage. **Uniquement en cas d'urgence :**

```json
{
  gateway: {
    controlUi: { dangerouslyDisableDeviceAuth: true },
    bind: "tailnet",
    auth: { mode: "token", token: "replace-me" },
  },
}
```

`dangerouslyDisableDeviceAuth` désactive les vérifications d'identité de l'appareil de l'interface de contrôle et constitue une dégradation sévère de la sécurité. Revenez rapidement après une utilisation d'urgence. Voir [Tailscale](../gateway/tailscale.md) pour les conseils de configuration HTTPS.

## Construction de l'interface utilisateur

La Passerelle sert les fichiers statiques depuis `dist/control-ui`. Construisez-les avec :

```bash
pnpm ui:build # installe automatiquement les dépendances de l'interface au premier lancement
```

Base absolue optionnelle (lorsque vous voulez des URL d'actifs fixes) :

```
OPENCLAW_CONTROL_UI_BASE_PATH=/openclaw/ pnpm ui:build
```

Pour le développement local (serveur de développement séparé) :

```bash
pnpm ui:dev # installe automatiquement les dépendances de l'interface au premier lancement
```

Puis pointez l'interface utilisateur vers votre URL WS de Passerelle (par ex. `ws://127.0.0.1:18789`).

## Débogage/test : serveur de développement + Passerelle distante

L'interface de contrôle est constituée de fichiers statiques ; la cible WebSocket est configurable et peut être différente de l'origine HTTP. C'est pratique lorsque vous voulez le serveur de développement Vite localement mais que la Passerelle fonctionne ailleurs.

1.  Démarrez le serveur de développement de l'interface : `pnpm ui:dev`
2.  Ouvrez une URL comme :

```
http://localhost:5173/?gatewayUrl=ws://<gateway-host>:18789
```

Authentification unique optionnelle (si nécessaire) :

```
http://localhost:5173/?gatewayUrl=wss://<gateway-host>:18789#token=<gateway-token>
```

Notes :

-   `gatewayUrl` est stocké dans localStorage après le chargement et retiré de l'URL.
-   `token` est importé en mémoire pour l'onglet courant et retiré de l'URL ; il n'est pas stocké dans localStorage.
-   `password` est conservé en mémoire uniquement.
-   Lorsque `gatewayUrl` est défini, l'interface utilisateur ne revient pas aux informations d'identification de configuration ou d'environnement. Fournissez `token` (ou `password`) explicitement. L'absence d'informations d'identification explicites est une erreur.
-   Utilisez `wss://` lorsque la Passerelle est derrière TLS (Tailscale Serve, proxy HTTPS, etc.).
-   `gatewayUrl` n'est accepté que dans une fenêtre de niveau supérieur (non intégrée) pour empêcher le détournement de clic.
-   Les déploiements d'interface de contrôle non en boucle locale doivent définir `gateway.controlUi.allowedOrigins` explicitement (origines complètes). Cela inclut les configurations de développement à distance.
-   `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback=true` active le mode de secours d'origine basé sur l'en-tête Host, mais il s'agit d'un mode de sécurité dangereux.

Exemple :

```json
{
  gateway: {
    controlUi: {
      allowedOrigins: ["http://localhost:5173"],
    },
  },
}
```

Détails de la configuration d'accès à distance : [Accès à distance](../gateway/remote.md).

[Web](../web.md)[Tableau de bord](./dashboard.md)