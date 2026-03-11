

  Protocoles et APIs

  
# Protocole Bridge

Le protocole Bridge est un transport de nœud **legacy** (TCP JSONL). Les nouveaux clients nœud doivent utiliser le protocole unifié Gateway WebSocket. Si vous développez un opérateur ou un client nœud, utilisez le [protocole Gateway](./protocol.md). **Note :** Les versions actuelles d'OpenClaw ne fournissent plus l'écouteur TCP bridge ; ce document est conservé à titre de référence historique. Les anciennes clés de configuration `bridge.*` ne font plus partie du schéma de configuration.

## Pourquoi nous avons les deux

-   **Limite de sécurité** : le bridge expose une petite liste d'autorisation au lieu de la surface API complète de la gateway.
-   **Appairage + identité du nœud** : l'admission des nœuds est gérée par la gateway et liée à un jeton par nœud.
-   **UX de découverte** : les nœuds peuvent découvrir les gateways via Bonjour sur le LAN, ou se connecter directement via un tailnet.
-   **WS en boucle locale** : le plan de contrôle WS complet reste local sauf s'il est tunnelisé via SSH.

## Transport

-   TCP, un objet JSON par ligne (JSONL).
-   TLS optionnel (lorsque `bridge.tls.enabled` est vrai).
-   L'ancien port d'écoute par défaut était `18790` (les versions actuelles ne démarrent pas de TCP bridge).

Lorsque TLS est activé, les enregistrements TXT de découverte incluent `bridgeTls=1` ainsi que `bridgeTlsSha256` comme indice non secret. Notez que les enregistrements TXT Bonjour/mDNS ne sont pas authentifiés ; les clients ne doivent pas traiter l'empreinte annoncée comme un pin autoritaire sans intention explicite de l'utilisateur ou une autre vérification hors bande.

## Handshake + appairage

1.  Le client envoie `hello` avec les métadonnées du nœud + le jeton (s'il est déjà appairé).
2.  S'il n'est pas appairé, la gateway répond `error` (`NOT_PAIRED`/`UNAUTHORIZED`).
3.  Le client envoie `pair-request`.
4.  La gateway attend l'approbation, puis envoie `pair-ok` et `hello-ok`.

`hello-ok` renvoie `serverName` et peut inclure `canvasHostUrl`.

## Trames

Client → Gateway :

-   `req` / `res` : RPC gateway à portée limitée (chat, sessions, config, health, voicewake, skills.bins)
-   `event` : signaux du nœud (transcription vocale, requête d'agent, abonnement au chat, cycle de vie exec)

Gateway → Client :

-   `invoke` / `invoke-res` : commandes du nœud (`canvas.*`, `camera.*`, `screen.record`, `location.get`, `sms.send`)
-   `event` : mises à jour du chat pour les sessions abonnées
-   `ping` / `pong` : keepalive

L'application de la liste d'autorisation legacy se trouvait dans `src/gateway/server-bridge.ts` (supprimée).

## Événements du cycle de vie Exec

Les nœuds peuvent émettre des événements `exec.finished` ou `exec.denied` pour exposer l'activité de `system.run`. Ceux-ci sont mappés vers des événements système dans la gateway. (Les nœuds legacy peuvent encore émettre `exec.started`.) Champs de la charge utile (tous optionnels sauf indication contraire) :

-   `sessionKey` (requis) : session de l'agent qui recevra l'événement système.
-   `runId` : identifiant unique de l'exécution pour le regroupement.
-   `command` : chaîne de commande brute ou formatée.
-   `exitCode`, `timedOut`, `success`, `output` : détails de fin d'exécution (finished uniquement).
-   `reason` : raison du refus (denied uniquement).

## Utilisation avec Tailnet

-   Liez le bridge à une IP tailnet : `bridge.bind: "tailnet"` dans `~/.openclaw/openclaw.json`.
-   Les clients se connectent via le nom MagicDNS ou l'IP tailnet.
-   Bonjour ne traverse **pas** les réseaux ; utilisez un hôte/port manuel ou DNS‑SD étendu si nécessaire.

## Gestion des versions

Bridge est actuellement en **version implicite v1** (pas de négociation min/max). La rétrocompatibilité est attendue ; ajoutez un champ de version du protocole bridge avant toute modification cassante.

[Protocole Gateway](./protocol.md)[Complétions de Chat OpenAI](./openai-http-api.md)

---