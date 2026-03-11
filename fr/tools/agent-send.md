

  Coordination d'agents

  
# Agent Send

`openclaw agent` exécute un tour d'agent unique sans nécessiter de message de chat entrant. Par défaut, il passe **par la Gateway** ; ajoutez `--local` pour forcer l'exécution du runtime embarqué sur la machine actuelle.

## Comportement

-   Requis : `--message `
-   Sélection de session :
    -   `--to ` dérive la clé de session (les cibles de groupe/canal préservent l'isolation ; les discussions directes sont réduites à `main`), **ou**
    -   `--session-id ` réutilise une session existante par son id, **ou**
    -   `--agent ` cible un agent configuré directement (utilise la clé de session `main` de cet agent)
-   Exécute le même runtime d'agent embarqué que pour les réponses entrantes normales.
-   Les drapeaux de réflexion/verbose persistent dans le stockage de session.
-   Sortie :
    -   par défaut : imprime le texte de la réponse (plus les lignes `MEDIA:`)
    -   `--json` : imprime une charge utile structurée + métadonnées
-   Livraison optionnelle vers un canal avec `--deliver` + `--channel` (les formats de cible correspondent à `openclaw message --target`).
-   Utilisez `--reply-channel`/`--reply-to`/`--reply-account` pour remplacer la livraison sans changer la session.

Si la Gateway est inaccessible, la CLI **bascule** vers l'exécution locale embarquée.

## Exemples

```bash
openclaw agent --to +15555550123 --message "status update"
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --to +15555550123 --message "Trace logs" --verbose on --json
openclaw agent --to +15555550123 --message "Summon reply" --deliver
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```

## Drapeaux

-   `--local` : exécuter localement (nécessite les clés API du fournisseur de modèle dans votre shell)
-   `--deliver` : envoyer la réponse sur le canal choisi
-   `--channel` : canal de livraison (`whatsapp|telegram|discord|googlechat|slack|signal|imessage`, par défaut : `whatsapp`)
-   `--reply-to` : remplacement de la cible de livraison
-   `--reply-channel` : remplacement du canal de livraison
-   `--reply-account` : remplacement de l'id du compte de livraison
-   `--thinking <off|minimal|low|medium|high|xhigh>` : persister le niveau de réflexion (modèles GPT-5.2 + Codex uniquement)
-   `--verbose <on|full|off>` : persister le niveau verbose
-   `--timeout ` : remplacer le timeout de l'agent
-   `--json` : sortie JSON structurée

[Dépannage Navigateur](./browser-linux-troubleshooting.md)[Sous-Agents](./subagents.md)

---