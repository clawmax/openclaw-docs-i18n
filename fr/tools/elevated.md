

  Outils intégrés

  
# Mode Élevé

## Ce qu'il fait

-   `/elevated on` s'exécute sur l'hôte de passerelle et conserve les approbations exec (identique à `/elevated ask`).
-   `/elevated full` s'exécute sur l'hôte de passerelle **et** approuve automatiquement exec (ignore les approbations exec).
-   `/elevated ask` s'exécute sur l'hôte de passerelle mais conserve les approbations exec (identique à `/elevated on`).
-   `on`/`ask` ne forcent **pas** `exec.security=full` ; la politique de sécurité/demande configurée s'applique toujours.
-   Ne modifie le comportement que lorsque l'agent est **sandboxé** (sinon exec s'exécute déjà sur l'hôte).
-   Formes de directive : `/elevated on|off|ask|full`, `/elev on|off|ask|full`.
-   Seuls `on|off|ask|full` sont acceptés ; toute autre valeur renvoie un indice et ne change pas l'état.

## Ce qu'il contrôle (et ce qu'il ne contrôle pas)

-   **Portes de disponibilité** : `tools.elevated` est la base globale. `agents.list[].tools.elevated` peut restreindre davantage l'élévation par agent (les deux doivent l'autoriser).
-   **État par session** : `/elevated on|off|ask|full` définit le niveau d'élévation pour la clé de session actuelle.
-   **Directive en ligne** : `/elevated on|ask|full` à l'intérieur d'un message s'applique uniquement à ce message.
-   **Groupes** : Dans les discussions de groupe, les directives élevées ne sont honorées que lorsque l'agent est mentionné. Les messages uniquement de commande qui contournent les exigences de mention sont traités comme mentionnés.
-   **Exécution sur l'hôte** : elevated force `exec` sur l'hôte de passerelle ; `full` définit également `security=full`.
-   **Approbations** : `full` ignore les approbations exec ; `on`/`ask` les respectent lorsque les règles de liste d'autorisation/demande l'exigent.
-   **Agents non sandboxés** : sans effet sur l'emplacement ; n'affecte que le contrôle d'accès, la journalisation et l'état.
-   **La politique d'outil s'applique toujours** : si `exec` est refusé par la politique d'outil, elevated ne peut pas être utilisé.
-   **Séparé de `/exec`** : `/exec` ajuste les paramètres par défaut de session pour les expéditeurs autorisés et ne nécessite pas elevated.

## Ordre de résolution

1.  Directive en ligne sur le message (s'applique uniquement à ce message).
2.  Remplacement de session (défini en envoyant un message uniquement de directive).
3.  Valeur par défaut globale (`agents.defaults.elevatedDefault` dans la configuration).

## Définir une valeur par défaut de session

-   Envoyez un message qui est **uniquement** la directive (les espaces blancs sont autorisés), par ex. `/elevated full`.
-   Une réponse de confirmation est envoyée (`Mode élevé défini sur full...` / `Mode élevé désactivé.`).
-   Si l'accès élevé est désactivé ou si l'expéditeur n'est pas sur la liste d'autorisation approuvée, la directive répond avec une erreur exploitable et ne change pas l'état de la session.
-   Envoyez `/elevated` (ou `/elevated:`) sans argument pour voir le niveau d'élévation actuel.

## Disponibilité + listes d'autorisation

-   Porte de fonctionnalité : `tools.elevated.enabled` (la valeur par défaut peut être désactivée via la configuration même si le code la prend en charge).
-   Liste d'autorisation des expéditeurs : `tools.elevated.allowFrom` avec des listes d'autorisation par fournisseur (par ex. `discord`, `whatsapp`).
-   Les entrées de liste d'autorisation sans préfixe ne correspondent qu'aux valeurs d'identité limitées à l'expéditeur (`SenderId`, `SenderE164`, `From`) ; les champs de routage des destinataires ne sont jamais utilisés pour l'autorisation élevée.
-   Les métadonnées d'expéditeur modifiables nécessitent des préfixes explicites :
    -   `name:` correspond à `SenderName`
    -   `username:` correspond à `SenderUsername`
    -   `tag:` correspond à `SenderTag`
    -   `id:`, `from:`, `e164:` sont disponibles pour un ciblage explicite de l'identité
-   Porte par agent : `agents.list[].tools.elevated.enabled` (optionnel ; ne peut que restreindre davantage).
-   Liste d'autorisation par agent : `agents.list[].tools.elevated.allowFrom` (optionnel ; lorsqu'il est défini, l'expéditeur doit correspondre aux listes d'autorisation **globales et** par agent).
-   Solution de repli Discord : si `tools.elevated.allowFrom.discord` est omis, la liste `channels.discord.allowFrom` est utilisée comme solution de repli (héritage : `channels.discord.dm.allowFrom`). Définissez `tools.elevated.allowFrom.discord` (même `[]`) pour la remplacer. Les listes d'autorisation par agent n'utilisent **pas** la solution de repli.
-   Toutes les portes doivent être franchies ; sinon, elevated est traité comme indisponible.

## Journalisation + état

-   Les appels exec élevés sont journalisés au niveau info.
-   L'état de la session inclut le mode élevé (par ex. `elevated=ask`, `elevated=full`).

[Outil PDF](./pdf.md)[Outil Exec](./exec.md)