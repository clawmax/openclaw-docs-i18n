

  Configuration et opérations

  
# Verrou de Passerelle

Dernière mise à jour : 2025-12-11

## Pourquoi

-   Garantir qu'une seule instance de passerelle s'exécute par port de base sur le même hôte ; les passerelles supplémentaires doivent utiliser des profils isolés et des ports uniques.
-   Survivre aux plantages/SIGKILL sans laisser de fichiers de verrou obsolètes.
-   Échouer rapidement avec une erreur claire lorsque le port de contrôle est déjà occupé.

## Mécanisme

-   La passerelle lie l'écouteur WebSocket (par défaut `ws://127.0.0.1:18789`) immédiatement au démarrage en utilisant un écouteur TCP exclusif.
-   Si la liaison échoue avec `EADDRINUSE`, le démarrage lance `GatewayLockError("another gateway instance is already listening on ws://127.0.0.1:")`.
-   Le système d'exploitation libère l'écouteur automatiquement à toute sortie de processus, y compris les plantages et SIGKILL—aucun fichier de verrou séparé ou étape de nettoyage n'est nécessaire.
-   À l'arrêt, la passerelle ferme le serveur WebSocket et le serveur HTTP sous-jacent pour libérer le port rapidement.

## Surface d'erreur

-   Si un autre processus détient le port, le démarrage lance `GatewayLockError("another gateway instance is already listening on ws://127.0.0.1:")`.
-   Les autres échecs de liaison se manifestent comme `GatewayLockError("failed to bind gateway socket on ws://127.0.0.1:: …")`.

## Notes opérationnelles

-   Si le port est occupé par un *autre* processus, l'erreur est la même ; libérez le port ou choisissez-en un autre avec `openclaw gateway --port `.
-   L'application macOS maintient toujours sa propre garde PID légère avant de lancer la passerelle ; le verrou d'exécution est appliqué par la liaison WebSocket.

[Journalisation](./logging.md)[Exécution en arrière-plan et outil de processus](./background-process.md)