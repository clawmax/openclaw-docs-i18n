title: "Guide de dépannage et solutions pour la passerelle OpenClaw"
description: "Résolvez les problèmes courants de la passerelle OpenClaw : service non exécuté, absence de réponses, erreurs 429, connectivité du tableau de bord et problèmes de flux de messages avec des commandes étape par étape."
keywords: ["dépannage openclaw", "passerelle non exécutée", "erreur 429 anthropic", "connectivité du tableau de bord", "problèmes de flux de messages", "appairage de canal", "cron heartbeat", "échec de l'outil navigateur"]
---

  Configuration et opérations

  
# Dépannage

Cette page est le runbook approfondi. Commencez par [/help/troubleshooting](../help/troubleshooting.md) si vous voulez d'abord le flux de triage rapide.

## Échelle de commandes

Exécutez celles-ci en premier, dans cet ordre :

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Signaux sains attendus :

-   `openclaw gateway status` affiche `Runtime: running` et `RPC probe: ok`.
-   `openclaw doctor` ne signale aucun problème de configuration/service bloquant.
-   `openclaw channels status --probe` montre des canaux connectés/prêts.

## Anthropic 429 usage supplémentaire requis pour le contexte long

Utilisez ceci lorsque les journaux/erreurs incluent : `HTTP 429: rate_limit_error: Extra usage is required for long context requests`.

```bash
openclaw logs --follow
openclaw models status
openclaw config get agents.defaults.models
```

Recherchez :

-   Le modèle Anthropic Opus/Sonnet sélectionné a `params.context1m: true`.
-   L'identifiant Anthropic actuel n'est pas éligible pour l'utilisation à contexte long.
-   Les requêtes échouent uniquement sur les sessions longues/exécutions de modèles nécessitant le chemin bêta 1M.

Options de correction :

1.  Désactivez `context1m` pour ce modèle pour revenir à la fenêtre de contexte normale.
2.  Utilisez une clé API Anthropic avec facturation, ou activez l'utilisation supplémentaire Anthropic sur le compte d'abonnement.
3.  Configurez des modèles de repli pour que les exécutions continuent lorsque les requêtes à contexte long d'Anthropic sont rejetées.

Liens connexes :

-   [/providers/anthropic](../providers/anthropic.md)
-   [/reference/token-use](../reference/token-use.md)
-   [/help/faq#why-am-i-seeing-http-429-ratelimiterror-from-anthropic](../help/faq.md#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)

## Absence de réponses

Si les canaux sont actifs mais que rien ne répond, vérifiez le routage et la politique avant de reconnecter quoi que ce soit.

```bash
openclaw status
openclaw channels status --probe
openclaw pairing list --channel <channel> [--account <id>]
openclaw config get channels
openclaw logs --follow
```

Recherchez :

-   Appairage en attente pour les expéditeurs de messages privés.
-   Gestion des mentions de groupe (`requireMention`, `mentionPatterns`).
-   Incompatibilités de liste autorisée de canal/groupe.

Signatures courantes :

-   `drop guild message (mention required` → message de groupe ignoré jusqu'à mention.
-   `pairing request` → l'expéditeur a besoin d'approbation.
-   `blocked` / `allowlist` → l'expéditeur/le canal a été filtré par la politique.

Liens connexes :

-   [/channels/troubleshooting](../channels/troubleshooting.md)
-   [/channels/pairing](../channels/pairing.md)
-   [/channels/groups](../channels/groups.md)

## Connectivité de l'interface de contrôle du tableau de bord

Lorsque le tableau de bord/l'interface de contrôle ne se connecte pas, validez l'URL, le mode d'authentification et les hypothèses de contexte sécurisé.

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
openclaw doctor
openclaw gateway status --json
```

Recherchez :

-   URL de sonde et URL du tableau de bord correctes.
-   Incompatibilité de mode d'authentification/jeton entre le client et la passerelle.
-   Utilisation HTTP là où l'identité de l'appareil est requise.

Signatures courantes :

-   `device identity required` → contexte non sécurisé ou authentification d'appareil manquante.
-   `device nonce required` / `device nonce mismatch` → le client ne termine pas le flux d'authentification d'appareil basé sur un défi (`connect.challenge` + `device.nonce`).
-   `device signature invalid` / `device signature expired` → le client a signé la mauvaise charge utile (ou un horodatage périmé) pour la poignée de main actuelle.
-   `unauthorized` / boucle de reconnexion → incompatibilité de mot de passe/jeton.
-   `gateway connect failed:` → mauvais hôte/port/URL cible.

Vérification de la migration de l'authentification d'appareil v2 :

```bash
openclaw --version
openclaw doctor
openclaw gateway status
```

Si les journaux montrent des erreurs de nonce/signature, mettez à jour le client de connexion et vérifiez qu'il :

1.  attend `connect.challenge`
2.  signe la charge utile liée au défi
3.  envoie `connect.params.device.nonce` avec le même nonce de défi

Liens connexes :

-   [/web/control-ui](../web/control-ui.md)
-   [/gateway/authentication](./authentication.md)
-   [/gateway/remote](./remote.md)

## Service de passerelle non exécuté

Utilisez ceci lorsque le service est installé mais que le processus ne reste pas actif.

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
openclaw doctor
openclaw gateway status --deep
```

Recherchez :

-   `Runtime: stopped` avec des indices de sortie.
-   Incompatibilité de configuration de service (`Config (cli)` vs `Config (service)`).
-   Conflits de port/écouteur.

Signatures courantes :

-   `Gateway start blocked: set gateway.mode=local` → le mode de passerelle locale n'est pas activé. Correction : définissez `gateway.mode="local"` dans votre configuration (ou exécutez `openclaw configure`). Si vous exécutez OpenClaw via Podman en utilisant l'utilisateur dédié `openclaw`, la configuration se trouve dans `~openclaw/.openclaw/openclaw.json`.
-   `refusing to bind gateway ... without auth` → liaison non-boucle locale sans jeton/mot de passe.
-   `another gateway instance is already listening` / `EADDRINUSE` → conflit de port.

Liens connexes :

-   [/gateway/background-process](./background-process.md)
-   [/gateway/configuration](./configuration.md)
-   [/gateway/doctor](./doctor.md)

## Canal connecté mais messages ne circulant pas

Si l'état du canal est connecté mais que le flux de messages est mort, concentrez-vous sur la politique, les permissions et les règles de livraison spécifiques au canal.

```bash
openclaw channels status --probe
openclaw pairing list --channel <channel> [--account <id>]
openclaw status --deep
openclaw logs --follow
openclaw config get channels
```

Recherchez :

-   Politique de messages privés (`pairing`, `allowlist`, `open`, `disabled`).
-   Liste autorisée de groupe et exigences de mention.
-   Permissions/portées d'API de canal manquantes.

Signatures courantes :

-   `mention required` → message ignoré par la politique de mention de groupe.
-   `pairing` / traces d'approbation en attente → l'expéditeur n'est pas approuvé.
-   `missing_scope`, `not_in_channel`, `Forbidden`, `401/403` → problème d'authentification/permissions du canal.

Liens connexes :

-   [/channels/troubleshooting](../channels/troubleshooting.md)
-   [/channels/whatsapp](../channels/whatsapp.md)
-   [/channels/telegram](../channels/telegram.md)
-   [/channels/discord](../channels/discord.md)

## Cron et livraison du heartbeat

Si le cron ou le heartbeat ne s'est pas exécuté ou n'a pas été livré, vérifiez d'abord l'état du planificateur, puis la cible de livraison.

```bash
openclaw cron status
openclaw cron list
openclaw cron runs --id <jobId> --limit 20
openclaw system heartbeat last
openclaw logs --follow
```

Recherchez :

-   Cron activé et prochain réveil présent.
-   Historique d'exécution des tâches (`ok`, `skipped`, `error`).
-   Raisons de saut du heartbeat (`quiet-hours`, `requests-in-flight`, `alerts-disabled`).

Signatures courantes :

-   `cron: scheduler disabled; jobs will not run automatically` → cron désactivé.
-   `cron: timer tick failed` → le tick du planificateur a échoué ; vérifiez les erreurs de fichier/journal/exécution.
-   `heartbeat skipped` avec `reason=quiet-hours` → en dehors de la fenêtre d'heures actives.
-   `heartbeat: unknown accountId` → identifiant de compte invalide pour la cible de livraison du heartbeat.
-   `heartbeat skipped` avec `reason=dm-blocked` → la cible du heartbeat a été résolue vers une destination de type message privé alors que `agents.defaults.heartbeat.directPolicy` (ou le remplacement par agent) est défini sur `block`.

Liens connexes :

-   [/automation/troubleshooting](../automation/troubleshooting.md)
-   [/automation/cron-jobs](../automation/cron-jobs.md)
-   [/gateway/heartbeat](./heartbeat.md)

## Outil de nœud appairé échoue

Si un nœud est appairé mais que les outils échouent, isolez l'état de premier plan, de permission et d'approbation.

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
openclaw logs --follow
openclaw status
```

Recherchez :

-   Nœud en ligne avec les capacités attendues.
-   Autorisations système accordées pour caméra/micro/localisation/écran.
-   Approbations d'exécution et état de la liste autorisée.

Signatures courantes :

-   `NODE_BACKGROUND_UNAVAILABLE` → l'application du nœud doit être au premier plan.
-   `*_PERMISSION_REQUIRED` / `LOCATION_PERMISSION_REQUIRED` → permission système manquante.
-   `SYSTEM_RUN_DENIED: approval required` → approbation d'exécution en attente.
-   `SYSTEM_RUN_DENIED: allowlist miss` → commande bloquée par la liste autorisée.

Liens connexes :

-   [/nodes/troubleshooting](../nodes/troubleshooting.md)
-   [/nodes/index](../nodes/index.md)
-   [/tools/exec-approvals](../tools/exec-approvals.md)

## Outil navigateur échoue

Utilisez ceci lorsque les actions de l'outil navigateur échouent même si la passerelle elle-même est saine.

```bash
openclaw browser status
openclaw browser start --browser-profile openclaw
openclaw browser profiles
openclaw logs --follow
openclaw doctor
```

Recherchez :

-   Chemin d'exécutable de navigateur valide.
-   Accessibilité du profil CDP.
-   Attachement de l'onglet de relais d'extension pour `profile="chrome"`.

Signatures courantes :

-   `Failed to start Chrome CDP on port` → échec du lancement du processus du navigateur.
-   `browser.executablePath not found` → le chemin configuré est invalide.
-   `Chrome extension relay is running, but no tab is connected` → relais d'extension non attaché.
-   `Browser attachOnly is enabled ... not reachable` → le profil en attachement seule n'a pas de cible accessible.

Liens connexes :

-   [/tools/browser-linux-troubleshooting](../tools/browser-linux-troubleshooting.md)
-   [/tools/chrome-extension](../tools/chrome-extension.md)
-   [/tools/browser](../tools/browser.md)

## Si vous avez mis à niveau et que quelque chose a soudainement cessé de fonctionner

La plupart des dysfonctionnements post-mise à niveau sont dus à une dérive de configuration ou à des valeurs par défaut plus strictes maintenant appliquées.

### 1) Comportement de l'authentification et du remplacement d'URL modifié

```bash
openclaw gateway status
openclaw config get gateway.mode
openclaw config get gateway.remote.url
openclaw config get gateway.auth.mode
```

À vérifier :

-   Si `gateway.mode=remote`, les appels CLI peuvent cibler une passerelle distante alors que votre service local fonctionne.
-   Les appels explicites `--url` ne reviennent pas aux identifiants stockés.

Signatures courantes :

-   `gateway connect failed:` → mauvaise URL cible.
-   `unauthorized` → point de terminaison accessible mais mauvaise authentification.

### 2) Les garde-fous de liaison et d'authentification sont plus stricts

```bash
openclaw config get gateway.bind
openclaw config get gateway.auth.token
openclaw gateway status
openclaw logs --follow
```

À vérifier :

-   Les liaisons non-boucle locale (`lan`, `tailnet`, `custom`) nécessitent une authentification configurée.
-   Les anciennes clés comme `gateway.token` ne remplacent pas `gateway.auth.token`.

Signatures courantes :

-   `refusing to bind gateway ... without auth` → incompatibilité liaison+authentification.
-   `RPC probe: failed` alors que le runtime est en cours d'exécution → passerelle active mais inaccessible avec l'authentification/URL actuelle.

### 3) L'état d'appairage et d'identité de l'appareil a changé

```bash
openclaw devices list
openclaw pairing list --channel <channel> [--account <id>]
openclaw logs --follow
openclaw doctor
```

À vérifier :

-   Approbations d'appareil en attente pour le tableau de bord/les nœuds.
-   Approbations d'appairage de messages privés en attente après des changements de politique ou d'identité.

Signatures courantes :

-   `device identity required` → authentification de l'appareil non satisfaite.
-   `pairing required` → l'expéditeur/l'appareil doit être approuvé.

Si la configuration du service et le runtime sont toujours en désaccord après les vérifications, réinstallez les métadonnées du service à partir du même répertoire de profil/état :

```bash
openclaw gateway install --force
openclaw gateway restart
```

Liens connexes :

-   [/gateway/pairing](./pairing.md)
-   [/gateway/authentication](./authentication.md)
-   [/gateway/background-process](./background-process.md)

[Passerelles multiples](./multiple-gateways.md)[Sécurité](./security.md)