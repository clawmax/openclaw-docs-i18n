

  Aide

  
# Dépannage

Si vous n'avez que 2 minutes, utilisez cette page comme porte d'entrée de triage.

## Premières 60 secondes

Exécutez cette échelle exacte dans l'ordre :

```bash
openclaw status
openclaw status --all
openclaw gateway probe
openclaw gateway status
openclaw doctor
openclaw channels status --probe
openclaw logs --follow
```

Un bon résultat en une ligne :

-   `openclaw status` → affiche les canaux configurés et aucune erreur d'authentification évidente.
-   `openclaw status --all` → le rapport complet est présent et partageable.
-   `openclaw gateway probe` → la cible de la passerelle attendue est joignable.
-   `openclaw gateway status` → `Runtime: running` et `RPC probe: ok`.
-   `openclaw doctor` → aucune erreur de configuration/service bloquante.
-   `openclaw channels status --probe` → les canaux rapportent `connected` ou `ready`.
-   `openclaw logs --follow` → activité régulière, pas d'erreurs fatales répétées.

## Contexte long Anthropic 429

Si vous voyez : `HTTP 429: rate_limit_error: Extra usage is required for long context requests`, allez à [/gateway/troubleshooting#anthropic-429-extra-usage-required-for-long-context](../gateway/troubleshooting.md#anthropic-429-extra-usage-required-for-long-context).

## L'installation du plugin échoue avec des extensions openclaw manquantes

Si l'installation échoue avec `package.json missing openclaw.extensions`, le package du plugin utilise un format ancien qu'OpenClaw n'accepte plus. Corrigez dans le package du plugin :

1.  Ajoutez `openclaw.extensions` au `package.json`.
2.  Pointez les entrées vers les fichiers d'exécution compilés (généralement `./dist/index.js`).
3.  Republiez le plugin et exécutez `openclaw plugins install <npm-spec>` à nouveau.

Exemple :

```json
{
  "name": "@openclaw/my-plugin",
  "version": "1.2.3",
  "openclaw": {
    "extensions": ["./dist/index.js"]
  }
}
```

Référence : [/tools/plugin#distribution-npm](../tools/plugin.md#distribution-npm)

## Arbre de décision

```bash
openclaw status
openclaw gateway status
openclaw channels status --probe
openclaw pairing list --channel <channel> [--account <id>]
openclaw logs --follow
```

Un bon résultat ressemble à :

-   `Runtime: running`
-   `RPC probe: ok`
-   Votre canal affiche connecté/prêt dans `channels status --probe`
-   L'expéditeur apparaît approuvé (ou la politique de MP est ouverte/liste autorisée)

Signatures courantes dans les journaux :

-   `drop guild message (mention required` → la mention obligatoire a bloqué le message dans Discord.
-   `pairing request` → l'expéditeur n'est pas approuvé et attend l'approbation de jumelage en MP.
-   `blocked` / `allowlist` dans les journaux du canal → l'expéditeur, la salle ou le groupe est filtré.

Pages approfondies :

-   [/gateway/troubleshooting#no-replies](../gateway/troubleshooting.md#no-replies)
-   [/channels/troubleshooting](../channels/troubleshooting.md)
-   [/channels/pairing](../channels/pairing.md)

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Un bon résultat ressemble à :

-   `Dashboard: http://...` est affiché dans `openclaw gateway status`
-   `RPC probe: ok`
-   Pas de boucle d'authentification dans les journaux

Signatures courantes dans les journaux :

-   `device identity required` → le contexte HTTP/non sécurisé ne peut pas terminer l'authentification de l'appareil.
-   `unauthorized` / boucle de reconnexion → mauvais jeton/mot de passe ou incompatibilité du mode d'authentification.
-   `gateway connect failed:` → l'interface cible la mauvaise URL/port ou une passerelle injoignable.

Pages approfondies :

-   [/gateway/troubleshooting#dashboard-control-ui-connectivity](../gateway/troubleshooting.md#dashboard-control-ui-connectivity)
-   [/web/control-ui](../web/control-ui.md)
-   [/gateway/authentication](../gateway/authentication.md)

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Un bon résultat ressemble à :

-   `Service: ... (loaded)`
-   `Runtime: running`
-   `RPC probe: ok`

Signatures courantes dans les journaux :

-   `Gateway start blocked: set gateway.mode=local` → le mode de la passerelle n'est pas défini/est distant.
-   `refusing to bind gateway ... without auth` → liaison non locale sans jeton/mot de passe.
-   `another gateway instance is already listening` ou `EADDRINUSE` → le port est déjà utilisé.

Pages approfondies :

-   [/gateway/troubleshooting#gateway-service-not-running](../gateway/troubleshooting.md#gateway-service-not-running)
-   [/gateway/background-process](../gateway/background-process.md)
-   [/gateway/configuration](../gateway/configuration.md)

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Un bon résultat ressemble à :

-   Le transport du canal est connecté.
-   Les vérifications de jumelage/liste autorisée passent.
-   Les mentions sont détectées là où c'est requis.

Signatures courantes dans les journaux :

-   `mention required` → la mention obligatoire de groupe a bloqué le traitement.
-   `pairing` / `pending` → l'expéditeur en MP n'est pas encore approuvé.
-   `not_in_channel`, `missing_scope`, `Forbidden`, `401/403` → problème de jeton d'autorisation du canal.

Pages approfondies :

-   [/gateway/troubleshooting#channel-connected-messages-not-flowing](../gateway/troubleshooting.md#channel-connected-messages-not-flowing)
-   [/channels/troubleshooting](../channels/troubleshooting.md)

```bash
openclaw status
openclaw gateway status
openclaw cron status
openclaw cron list
openclaw cron runs --id <jobId> --limit 20
openclaw logs --follow
```

Un bon résultat ressemble à :

-   `cron.status` montre activé avec un prochain réveil.
-   `cron runs` montre des entrées `ok` récentes.
-   Le heartbeat est activé et n'est pas en dehors des heures d'activité.

Signatures courantes dans les journaux :

-   `cron: scheduler disabled; jobs will not run automatically` → le cron est désactivé.
-   `heartbeat skipped` avec `reason=quiet-hours` → en dehors des heures d'activité configurées.
-   `requests-in-flight` → la voie principale est occupée ; le réveil du heartbeat a été différé.
-   `unknown accountId` → le compte cible de livraison du heartbeat n'existe pas.

Pages approfondies :

-   [/gateway/troubleshooting#cron-and-heartbeat-delivery](../gateway/troubleshooting.md#cron-and-heartbeat-delivery)
-   [/automation/troubleshooting](../automation/troubleshooting.md)
-   [/gateway/heartbeat](../gateway/heartbeat.md)

```bash
openclaw status
openclaw gateway status
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw logs --follow
```

Un bon résultat ressemble à :

-   Le nœud est listé comme connecté et jumelé pour le rôle `node`.
-   La capacité existe pour la commande que vous invoquez.
-   L'état de permission est accordé pour l'outil.

Signatures courantes dans les journaux :

-   `NODE_BACKGROUND_UNAVAILABLE` → amenez l'application du nœud au premier plan.
-   `*_PERMISSION_REQUIRED` → la permission du système d'exploitation a été refusée/est manquante.
-   `SYSTEM_RUN_DENIED: approval required` → l'approbation d'exécution est en attente.
-   `SYSTEM_RUN_DENIED: allowlist miss` → la commande n'est pas sur la liste autorisée d'exécution.

Pages approfondies :

-   [/gateway/troubleshooting#node-paired-tool-fails](../gateway/troubleshooting.md#node-paired-tool-fails)
-   [/nodes/troubleshooting](../nodes/troubleshooting.md)
-   [/tools/exec-approvals](../tools/exec-approvals.md)

```bash
openclaw status
openclaw gateway status
openclaw browser status
openclaw logs --follow
openclaw doctor
```

Un bon résultat ressemble à :

-   Le statut du navigateur montre `running: true` et un navigateur/profil choisi.
-   Le profil `openclaw` démarre ou le relais `chrome` a un onglet attaché.

Signatures courantes dans les journaux :

-   `Failed to start Chrome CDP on port` → le lancement du navigateur local a échoué.
-   `browser.executablePath not found` → le chemin du binaire configuré est incorrect.
-   `Chrome extension relay is running, but no tab is connected` → l'extension n'est pas attachée.
-   `Browser attachOnly is enabled ... not reachable` → le profil attaché uniquement n'a pas de cible CDP active.

Pages approfondies :

-   [/gateway/troubleshooting#browser-tool-fails](../gateway/troubleshooting.md#browser-tool-fails)
-   [/tools/browser-linux-troubleshooting](../tools/browser-linux-troubleshooting.md)
-   [/tools/chrome-extension](../tools/chrome-extension.md)

[Aide](../help.md)[FAQ](./faq.md)