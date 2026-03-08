title: "Guide des commandes d'audit de sécurité et de correction de l'interface CLI OpenClaw"
description: "Apprenez à utiliser l'outil d'audit de sécurité de l'interface CLI OpenClaw pour identifier les vulnérabilités et appliquer des corrections automatiques pour une configuration sécurisée."
keywords: ["sécurité openclaw", "audit de sécurité cli", "outil d'audit de sécurité", "correctifs de sécurité", "durcissement de configuration", "vulnérabilités de sécurité", "paramètres sandbox", "sécurité de la passerelle"]
---

  Commandes CLI

  
# security

Outils de sécurité (audit + correctifs optionnels). Liens utiles :

-   Guide de sécurité : [Sécurité](../gateway/security.md)

## Audit

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
openclaw security audit --json
```

L'audit avertit lorsque plusieurs expéditeurs de messages directs partagent la session principale et recommande le **mode DM sécurisé** : `session.dmScope="per-channel-peer"` (ou `per-account-channel-peer` pour les canaux multi-comptes) pour les boîtes de réception partagées. Ceci est destiné au durcissement des boîtes de réception coopératives/partagées. Une passerelle unique partagée par des opérateurs mutuellement non fiables/antagonistes n'est pas une configuration recommandée ; séparez les limites de confiance avec des passerelles distinctes (ou des utilisateurs/hôtes système distincts). Il émet également `security.trust_model.multi_user_heuristic` lorsque la configuration suggère un accès entrant probablement multi-utilisateur (par exemple, politique de groupe/DM ouverte, cibles de groupe configurées, ou règles d'expéditeur avec caractères génériques), et rappelle qu'OpenClaw est par défaut un modèle de confiance d'assistant personnel. Pour les configurations multi-utilisateurs intentionnelles, les recommandations de l'audit sont de sandboxer toutes les sessions, de limiter l'accès au système de fichiers à l'espace de travail défini, et de ne pas placer d'identités ou d'informations d'identification personnelles/privées sur ce runtime. Il avertit également lorsque de petits modèles (`<=300B`) sont utilisés sans sandboxing et avec les outils web/navigateur activés. Pour l'accès entrant par webhook, il avertit lorsque `hooks.defaultSessionKey` n'est pas défini, lorsque les remplacements de `sessionKey` par requête sont activés, et lorsque ces remplacements sont activés sans `hooks.allowedSessionKeyPrefixes`. Il avertit également lorsque les paramètres Docker du sandbox sont configurés alors que le mode sandbox est désactivé, lorsque `gateway.nodes.denyCommands` utilise des entrées de type motif/inefficaces (correspondance exacte du nom de commande du nœud uniquement, pas de filtrage par texte shell), lorsque `gateway.nodes.allowCommands` active explicitement des commandes de nœud dangereuses, lorsque le profil global `tools.profile="minimal"` est remplacé par des profils d'outils d'agent, lorsque des groupes ouverts exposent des outils runtime/système de fichiers sans gardes sandbox/espace de travail, et lorsque des outils d'extension de plugin installés peuvent être accessibles sous une politique d'outils permissive. Il signale également `gateway.allowRealIpFallback=true` (risque d'usurpation d'en-tête si les proxies sont mal configurés) et `discovery.mdns.mode="full"` (fuite de métadonnées via les enregistrements TXT mDNS). Il avertit également lorsque le navigateur sandbox utilise le réseau Docker `bridge` sans `sandbox.browser.cdpSourceRange`. Il signale également les modes de réseau Docker sandbox dangereux (y compris `host` et les jonctions d'espace de noms `container:*`). Il avertit également lorsque les conteneurs Docker de navigateur sandbox existants ont des étiquettes de hachage manquantes/obsolètes (par exemple, des conteneurs pré-migration sans `openclaw.browserConfigEpoch`) et recommande `openclaw sandbox recreate --browser --all`. Il avertit également lorsque les enregistrements d'installation de plugin/hook basés sur npm ne sont pas épinglés, manquent de métadonnées d'intégrité, ou divergent des versions de paquets actuellement installées. Il avertit lorsque les listes d'autorisation de canaux reposent sur des noms/emails/étiquettes mutables plutôt que sur des ID stables (Discord, Slack, Google Chat, MS Teams, Mattermost, périmètres IRC le cas échéant). Il avertit lorsque `gateway.auth.mode="none"` laisse les API HTTP de la passerelle accessibles sans secret partagé (`/tools/invoke` plus tout point de terminaison `/v1/*` activé). Les paramètres préfixés par `dangerous`/`dangerously` sont des dérogations explicites de l'opérateur en cas d'urgence ; en activer un n'est pas, en soi, un rapport de vulnérabilité de sécurité. Pour l'inventaire complet des paramètres dangereux, consultez la section "Résumé des indicateurs dangereux ou non sécurisés" dans [Sécurité](../gateway/security.md).

## Sortie JSON

Utilisez `--json` pour les vérifications CI/politiques :

```bash
openclaw security audit --json | jq '.summary'
openclaw security audit --deep --json | jq '.findings[] | select(.severity=="critical") | .checkId'
```

Si `--fix` et `--json` sont combinés, la sortie inclut à la fois les actions de correction et le rapport final :

```bash
openclaw security audit --fix --json | jq '{fix: .fix.ok, summary: .report.summary}'
```

## Ce que `--fix` modifie

`--fix` applique des remédiations sûres et déterministes :

-   change les `groupPolicy="open"` courants en `groupPolicy="allowlist"` (y compris les variantes de compte dans les canaux pris en charge)
-   définit `logging.redactSensitive` de `"off"` à `"tools"`
-   resserre les permissions pour l'état/la configuration et les fichiers sensibles courants (`credentials/*.json`, `auth-profiles.json`, `sessions.json`, session `*.jsonl`)

`--fix` ne fait **pas** :

-   rotation des jetons/mots de passe/clés API
-   désactivation d'outils (`gateway`, `cron`, `exec`, etc.)
-   modification des choix de liaison/auth/exposition réseau de la passerelle
-   suppression ou réécriture de plugins/compétences

[secrets](./secrets.md)[sessions](./sessions.md)