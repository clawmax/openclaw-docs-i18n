

  Commandes CLI

  
# system

Aides de niveau système pour la passerelle : mettre en file d'attente des événements système, contrôler les battements de cœur et afficher la présence.

## Commandes courantes

```bash
openclaw system event --text "Check for urgent follow-ups" --mode now
openclaw system heartbeat enable
openclaw system heartbeat last
openclaw system presence
```

## system event

Met en file d'attente un événement système sur la session **principale**. Le prochain battement de cœur l'injectera sous la forme d'une ligne `System:` dans l'invite. Utilisez `--mode now` pour déclencher le battement de cœur immédiatement ; `next-heartbeat` attend le prochain tick programmé. Drapeaux :

-   `--text ` : texte de l'événement système requis.
-   `--mode ` : `now` ou `next-heartbeat` (par défaut).
-   `--json` : sortie lisible par machine.

## system heartbeat last|enable|disable

Contrôles des battements de cœur :

-   `last` : affiche le dernier événement de battement de cœur.
-   `enable` : réactive les battements de cœur (à utiliser s'ils ont été désactivés).
-   `disable` : met en pause les battements de cœur.

Drapeaux :

-   `--json` : sortie lisible par machine.

## system presence

Liste les entrées de présence système actuelles que la passerelle connaît (nœuds, instances et lignes d'état similaires). Drapeaux :

-   `--json` : sortie lisible par machine.

## Notes

-   Nécessite une passerelle en cours d'exécution accessible par votre configuration actuelle (locale ou distante).
-   Les événements système sont éphémères et ne sont pas conservés entre les redémarrages.

[status](./status.md)[tui](./tui.md)

---