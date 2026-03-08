

  Médias et appareils

  
# Dépannage des nœuds

Utilisez cette page lorsqu'un nœud est visible en statut mais que les outils du nœud échouent.

## Échelle de commandes

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Puis exécutez les vérifications spécifiques aux nœuds :

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
```

Signaux de bon fonctionnement :

-   Le nœud est connecté et appairé pour le rôle `node`.
-   `nodes describe` inclut la capacité que vous appelez.
-   Les approbations d'exécution montrent le mode/liste blanche attendu.

## Exigences en premier plan

`canvas.*`, `camera.*`, et `screen.*` sont en premier plan uniquement sur les nœuds iOS/Android. Vérification et correction rapides :

```bash
openclaw nodes describe --node <idOrNameOrIp>
openclaw nodes canvas snapshot --node <idOrNameOrIp>
openclaw logs --follow
```

Si vous voyez `NODE_BACKGROUND_UNAVAILABLE`, amenez l'application du nœud au premier plan et réessayez.

## Matrice des permissions

| Capacité | iOS | Android | Application nœud macOS | Code d'échec typique |
| --- | --- | --- | --- | --- |
| `camera.snap`, `camera.clip` | Caméra (+ micro pour l'audio du clip) | Caméra (+ micro pour l'audio du clip) | Caméra (+ micro pour l'audio du clip) | `*_PERMISSION_REQUIRED` |
| `screen.record` | Enregistrement d'écran (+ micro optionnel) | Invite de capture d'écran (+ micro optionnel) | Enregistrement d'écran | `*_PERMISSION_REQUIRED` |
| `location.get` | Pendant l'utilisation ou Toujours (selon le mode) | Localisation premier plan/arrière-plan selon le mode | Permission de localisation | `LOCATION_PERMISSION_REQUIRED` |
| `system.run` | n/a (chemin hôte du nœud) | n/a (chemin hôte du nœud) | Approbations d'exécution requises | `SYSTEM_RUN_DENIED` |

## Appairage versus approbations

Ce sont des barrières différentes :

1.  **Appairage de l'appareil** : ce nœud peut-il se connecter à la passerelle ?
2.  **Approbations d'exécution** : ce nœud peut-il exécuter une commande shell spécifique ?

Vérifications rapides :

```bash
openclaw devices list
openclaw nodes status
openclaw approvals get --node <idOrNameOrIp>
openclaw approvals allowlist add --node <idOrNameOrIp> "/usr/bin/uname"
```

Si l'appairage est manquant, approuvez d'abord l'appareil du nœud. Si l'appairage est correct mais que `system.run` échoue, corrigez les approbations/liste blanche d'exécution.

## Codes d'erreur courants des nœuds

-   `NODE_BACKGROUND_UNAVAILABLE` → l'application est en arrière-plan ; amenez-la au premier plan.
-   `CAMERA_DISABLED` → le bouton caméra est désactivé dans les paramètres du nœud.
-   `*_PERMISSION_REQUIRED` → permission OS manquante/refusée.
-   `LOCATION_DISABLED` → le mode de localisation est désactivé.
-   `LOCATION_PERMISSION_REQUIRED` → le mode de localisation demandé n'est pas accordé.
-   `LOCATION_BACKGROUND_UNAVAILABLE` → l'application est en arrière-plan mais seule la permission "Pendant l'utilisation" existe.
-   `SYSTEM_RUN_DENIED: approval required` → la demande d'exécution nécessite une approbation explicite.
-   `SYSTEM_RUN_DENIED: allowlist miss` → commande bloquée par le mode liste blanche. Sur les hôtes de nœuds Windows, les formes de wrapper shell comme `cmd.exe /c ...` sont traitées comme des échecs de liste blanche en mode liste blanche, sauf si elles sont approuvées via le flux de demande.

## Boucle de récupération rapide

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
openclaw logs --follow
```

Si toujours bloqué :

-   Ré-approuvez l'appairage de l'appareil.
-   Rouvrez l'application du nœud (premier plan).
-   Re-accordez les permissions OS.
-   Recréez/ajustez la politique d'approbation d'exécution.

Liens connexes :

-   [/nodes/index](./index.md)
-   [/nodes/camera](./camera.md)
-   [/nodes/location-command](./location-command.md)
-   [/tools/exec-approvals](../tools/exec-approvals.md)
-   [/gateway/pairing](../gateway/pairing.md)

[Nœuds](../nodes.md)[Compréhension des médias](./media-understanding.md)