

  Application compagnon macOS

  
# Pont Peekaboo

OpenClaw peut héberger **PeekabooBridge** en tant que courtier d'automatisation d'interface local et conscient des permissions. Cela permet à la CLI `peekaboo` de piloter l'automatisation d'interface tout en réutilisant les permissions TCC de l'application macOS.

## Ce que c'est (et ce que ce n'est pas)

-   **Hôte** : OpenClaw.app peut agir comme un hôte PeekabooBridge.
-   **Client** : utilisez la CLI `peekaboo` (pas de surface `openclaw ui ...` distincte).
-   **Interface** : les superpositions visuelles restent dans Peekaboo.app ; OpenClaw est un hôte courtier léger.

## Activer le pont

Dans l'application macOS :

-   Paramètres → **Activer le pont Peekaboo**

Lorsqu'il est activé, OpenClaw démarre un serveur socket UNIX local. S'il est désactivé, l'hôte est arrêté et `peekaboo` reviendra aux autres hôtes disponibles.

## Ordre de découverte des clients

Les clients Peekaboo essaient généralement les hôtes dans cet ordre :

1.  Peekaboo.app (expérience utilisateur complète)
2.  Claude.app (si installé)
3.  OpenClaw.app (courtier léger)

Utilisez `peekaboo bridge status --verbose` pour voir quel hôte est actif et quel chemin de socket est utilisé. Vous pouvez le remplacer avec :

```bash
export PEEKABOO_BRIDGE_SOCKET=/path/to/bridge.sock
```

## Sécurité et permissions

-   Le pont valide les **signatures de code de l'appelant** ; une liste autorisée de TeamID est appliquée (TeamID de l'hôte Peekaboo + TeamID de l'application OpenClaw).
-   Les requêtes expirent après ~10 secondes.
-   Si les permissions requises sont manquantes, le pont renvoie un message d'erreur clair au lieu de lancer les Préférences Système.

## Comportement des instantanés (automatisation)

Les instantanés sont stockés en mémoire et expirent automatiquement après une courte fenêtre. Si vous avez besoin d'une rétention plus longue, recapturez depuis le client.

## Dépannage

-   Si `peekaboo` signale "le client du pont n'est pas autorisé", assurez-vous que le client est correctement signé ou exécutez l'hôte avec `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1` en mode **débogage** uniquement.
-   Si aucun hôte n'est trouvé, ouvrez l'une des applications hôtes (Peekaboo.app ou OpenClaw.app) et confirmez que les permissions sont accordées.

[Compétences](./skills.md)

---