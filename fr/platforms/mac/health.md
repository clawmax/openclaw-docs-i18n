

  Application compagnon macOS

  
# Vérifications de santé

Comment voir si le canal lié est sain depuis l'application de la barre de menus.

## Barre de menus

-   Le point de statut reflète désormais la santé de Baileys :
    -   Vert : lié + socket ouvert récemment.
    -   Orange : connexion/en cours de reconnexion.
    -   Rouge : déconnecté ou échec du sondage.
-   La ligne secondaire affiche "lié · auth 12m" ou montre la raison de l'échec.
-   L'élément de menu "Exécuter la vérification de santé" déclenche un sondage à la demande.

## Paramètres

-   L'onglet Général contient une carte Santé affichant : l'âge de l'authentification liée, le chemin/compteur du stockage de session, l'heure de la dernière vérification, la dernière erreur/code de statut, et des boutons pour Exécuter la vérification de santé / Afficher les journaux.
-   Utilise un instantané en cache pour que l'interface se charge instantanément et dégrade gracieusement en cas de déconnexion.
-   **L'onglet Canaux** expose le statut du canal + les contrôles pour WhatsApp/Telegram (QR de connexion, déconnexion, sondage, dernière déconnexion/erreur).

## Fonctionnement du sondage

-   L'application exécute `openclaw health --json` via `ShellExecutor` toutes les ~60s et à la demande. Le sondage charge les identifiants et rapporte le statut sans envoyer de messages.
-   Met en cache le dernier bon instantané et la dernière erreur séparément pour éviter le scintillement ; affiche l'horodatage de chacun.

## En cas de doute

-   Vous pouvez toujours utiliser le flux CLI dans [Santé de la passerelle](../../gateway/health.md) (`openclaw status`, `openclaw status --deep`, `openclaw health --json`) et suivre `/tmp/openclaw/openclaw-*.log` pour `web-heartbeat` / `web-reconnect`.

[Cycle de vie de la passerelle](./child-process.md)[Icône de la barre de menus](./icon.md)