

  Application compagnon macOS

  
# Cycle de vie de la passerelle

L'application macOS **gère la passerelle via launchd** par défaut et ne lance pas la passerelle en tant que processus enfant. Elle tente d'abord de s'attacher à une passerelle déjà en cours d'exécution sur le port configuré ; si aucune n'est accessible, elle active le service launchd via l'interface CLI externe `openclaw` (pas d'exécution embarquée). Cela vous offre un démarrage automatique fiable à la connexion et un redémarrage en cas de plantage. Le mode processus enfant (passerelle lancée directement par l'application) **n'est pas utilisé** actuellement. Si vous avez besoin d'un couplage plus étroit avec l'interface utilisateur, exécutez la passerelle manuellement dans un terminal.

## Comportement par défaut (launchd)

-   L'application installe un LaunchAgent par utilisateur étiqueté `ai.openclaw.gateway` (ou `ai.openclaw.` lors de l'utilisation de `--profile`/`OPENCLAW_PROFILE` ; l'ancienne nomenclature `com.openclaw.*` est prise en charge).
-   Lorsque le mode Local est activé, l'application s'assure que le LaunchAgent est chargé et démarre la passerelle si nécessaire.
-   Les journaux sont écrits dans le chemin de journalisation de la passerelle launchd (visible dans les Paramètres de débogage).

Commandes courantes :

```bash
launchctl kickstart -k gui/$UID/ai.openclaw.gateway
launchctl bootout gui/$UID/ai.openclaw.gateway
```

Remplacez l'étiquette par `ai.openclaw.` lors de l'exécution d'un profil nommé.

## Versions de développement non signées

`scripts/restart-mac.sh --no-sign` est destiné aux compilations locales rapides lorsque vous ne possédez pas de clés de signature. Pour empêcher launchd de pointer vers un binaire relais non signé, il :

-   Écrit `~/.openclaw/disable-launchagent`.

Les exécutions signées de `scripts/restart-mac.sh` effacent cette substitution si le marqueur est présent. Pour réinitialiser manuellement :

```bash
rm ~/.openclaw/disable-launchagent
```

## Mode attach-only

Pour forcer l'application macOS à **ne jamais installer ni gérer launchd**, lancez-la avec `--attach-only` (ou `--no-launchd`). Cela définit `~/.openclaw/disable-launchagent`, de sorte que l'application s'attache uniquement à une passerelle déjà en cours d'exécution. Vous pouvez activer/désactiver le même comportement dans les Paramètres de débogage.

## Mode distant

Le mode distant ne démarre jamais une passerelle locale. L'application utilise un tunnel SSH vers l'hôte distant et se connecte via ce tunnel.

## Pourquoi nous préférons launchd

-   Démarrage automatique à la connexion.
-   Sémantique de redémarrage/KeepAlive intégrée.
-   Journaux et supervision prévisibles.

Si un véritable mode processus enfant devait être à nouveau nécessaire, il devrait être documenté comme un mode distinct et explicite réservé au développement.

[Canvas](./canvas.md)[Contrôles de santé](./health.md)

---