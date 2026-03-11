

  Application compagnon macOS

  
# IPC macOS

**Modèle actuel :** un socket Unix local connecte le **service hôte node** à l'**application macOS** pour les approbations d'exécution et `system.run`. Un CLI de débogage `openclaw-mac` existe pour la découverte et les vérifications de connexion ; les actions de l'agent transitent toujours par le WebSocket de la Gateway et `node.invoke`. L'automatisation de l'interface utilisateur utilise PeekabooBridge.

## Objectifs

-   Une seule instance d'application GUI qui gère tout le travail lié à TCC (notifications, enregistrement d'écran, microphone, parole, AppleScript).
-   Une surface réduite pour l'automatisation : Gateway + commandes node, ainsi que PeekabooBridge pour l'automatisation de l'interface utilisateur.
-   Permissions prévisibles : toujours le même identifiant de bundle signé, lancé par launchd, afin que les autorisations TCC persistent.

## Fonctionnement

### Transport Gateway + node

-   L'application exécute la Gateway (mode local) et s'y connecte en tant que node.
-   Les actions de l'agent sont effectuées via `node.invoke` (par exemple `system.run`, `system.notify`, `canvas.*`).

### IPC service node + application

-   Un service hôte node sans interface se connecte au WebSocket de la Gateway.
-   Les requêtes `system.run` sont transmises à l'application macOS via un socket Unix local.
-   L'application exécute la commande dans le contexte de l'interface utilisateur, demande confirmation si nécessaire, et renvoie le résultat.

Diagramme (SCI) :

```
Agent -> Gateway -> Service Node (WS)
                      |  IPC (UDS + jeton + HMAC + TTL)
                      v
                  Application Mac (UI + TCC + system.run)
```

### PeekabooBridge (automatisation de l'interface utilisateur)

-   L'automatisation de l'interface utilisateur utilise un socket UNIX distinct nommé `bridge.sock` et le protocole JSON PeekabooBridge.
-   Ordre de préférence des hôtes (côté client) : Peekaboo.app → Claude.app → OpenClaw.app → exécution locale.
-   Sécurité : les hôtes du pont nécessit'un TeamID autorisé ; l'échappatoire DEBUG-only pour le même UID est protégée par `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1` (convention Peekaboo).
-   Voir : [Utilisation de PeekabooBridge](./peekaboo.md) pour plus de détails.

## Flux opérationnels

-   Redémarrage/reconstruction : `SIGN_IDENTITY="Apple Development:  ()" scripts/restart-mac.sh`
    -   Tue les instances existantes
    -   Construction Swift + packaging
    -   Écrit/amorce/démarre le LaunchAgent
-   Instance unique : l'application se termine prématurément si une autre instance avec le même identifiant de bundle est en cours d'exécution.

## Notes de renforcement de la sécurité

-   Préférer exiger une correspondance de TeamID pour toutes les surfaces privilégiées.
-   PeekabooBridge : `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1` (DEBUG-only) peut autoriser les appelants du même UID pour le développement local.
-   Toute la communication reste strictement locale ; aucun socket réseau n'est exposé.
-   Les invites TCC proviennent uniquement du bundle de l'application GUI ; maintenir l'identifiant de bundle signé stable entre les reconstructions.
-   Renforcement de l'IPC : mode socket `0600`, jeton, vérifications de l'UID du pair, défi/réponse HMAC, TTL court.

[Gateway sur macOS](./bundled-gateway.md)[Compétences](./skills.md)