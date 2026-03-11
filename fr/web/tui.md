

  Interfaces web

  
# TUI

## Démarrage rapide

1.  Démarrez la Passerelle.

```bash
openclaw gateway
```

2.  Ouvrez le TUI.

```bash
openclaw tui
```

3.  Tapez un message et appuyez sur Entrée.

Passerelle distante :

```bash
openclaw tui --url ws://<host>:<port> --token <gateway-token>
```

Utilisez `--password` si votre Passerelle utilise l'authentification par mot de passe.

## Ce que vous voyez

-   En-tête : URL de connexion, agent actuel, session actuelle.
-   Journal de discussion : messages de l'utilisateur, réponses de l'assistant, notifications système, cartes d'outils.
-   Ligne d'état : état de la connexion/exécution (connexion en cours, en cours d'exécution, diffusion, inactif, erreur).
-   Pied de page : état de la connexion + agent + session + modèle + réflexion/verbeux/raisonnement + compteurs de jetons + livraison.
-   Saisie : éditeur de texte avec saisie semi-automatique.

## Modèle mental : agents + sessions

-   Les agents sont des identifiants uniques (par ex. `main`, `research`). La Passerelle expose la liste.
-   Les sessions appartiennent à l'agent actuel.
-   Les clés de session sont stockées sous la forme `agent::`.
    -   Si vous tapez `/session main`, le TUI l'étend en `agent::main`.
    -   Si vous tapez `/session agent:other:main`, vous basculez explicitement vers cette session d'agent.
-   Portée de la session :
    -   `per-sender` (par défaut) : chaque agent possède de nombreuses sessions.
    -   `global` : le TUI utilise toujours la session `global` (le sélecteur peut être vide).
-   L'agent actuel + la session sont toujours visibles dans le pied de page.

## Envoi + livraison

-   Les messages sont envoyés à la Passerelle ; la livraison aux fournisseurs est désactivée par défaut.
-   Activez la livraison :
    -   `/deliver on`
    -   ou le panneau Paramètres
    -   ou démarrez avec `openclaw tui --deliver`

## Sélecteurs + superpositions

-   Sélecteur de modèle : liste les modèles disponibles et définit le remplacement de session.
-   Sélecteur d'agent : choisissez un agent différent.
-   Sélecteur de session : n'affiche que les sessions pour l'agent actuel.
-   Paramètres : active/désactive la livraison, l'expansion de la sortie des outils et la visibilité de la réflexion.

## Raccourcis clavier

-   Entrée : envoyer le message
-   Échap : interrompre l'exécution active
-   Ctrl+C : effacer la saisie (appuyez deux fois pour quitter)
-   Ctrl+D : quitter
-   Ctrl+L : sélecteur de modèle
-   Ctrl+G : sélecteur d'agent
-   Ctrl+P : sélecteur de session
-   Ctrl+O : activer/désactiver l'expansion de la sortie des outils
-   Ctrl+T : activer/désactiver la visibilité de la réflexion (recharge l'historique)

## Commandes slash

Principales :

-   `/help`
-   `/status`
-   `/agent ` (ou `/agents`)
-   `/session ` (ou `/sessions`)
-   `/model <provider/model>` (ou `/models`)

Contrôles de session :

-   `/think <off|minimal|low|medium|high>`
-   `/verbose <on|full|off>`
-   `/reasoning <on|off|stream>`
-   `/usage <off|tokens|full>`
-   `/elevated <on|off|ask|full>` (alias : `/elev`)
-   `/activation <mention|always>`
-   `/deliver <on|off>`

Cycle de vie de la session :

-   `/new` ou `/reset` (réinitialiser la session)
-   `/abort` (interrompre l'exécution active)
-   `/settings`
-   `/exit`

Les autres commandes slash de la Passerelle (par exemple, `/context`) sont transmises à la Passerelle et affichées comme sortie système. Voir [Commandes slash](../tools/slash-commands.md).

## Commandes shell locales

-   Précédez une ligne de `!` pour exécuter une commande shell locale sur l'hôte du TUI.
-   Le TUI demande une confirmation par session pour autoriser l'exécution locale ; refuser garde `!` désactivé pour la session.
-   Les commandes s'exécutent dans un shell nouveau et non interactif, dans le répertoire de travail du TUI (pas de `cd`/env persistant).
-   Les commandes shell locales reçoivent `OPENCLAW_SHELL=tui-local` dans leur environnement.
-   Un simple `!` est envoyé comme un message normal ; les espaces en début de ligne ne déclenchent pas l'exécution locale.

## Sortie des outils

-   Les appels d'outils s'affichent sous forme de cartes avec arguments + résultats.
-   Ctrl+O bascule entre les vues réduite/développée.
-   Pendant l'exécution des outils, les mises à jour partielles sont diffusées dans la même carte.

## Historique + diffusion

-   À la connexion, le TUI charge le dernier historique (200 messages par défaut).
-   Les réponses en diffusion se mettent à jour sur place jusqu'à leur finalisation.
-   Le TUI écoute également les événements d'outils de l'agent pour des cartes d'outils plus riches.

## Détails de la connexion

-   Le TUI s'enregistre auprès de la Passerelle en tant que `mode: "tui"`.
-   Les reconnexions affichent un message système ; les lacunes d'événements sont signalées dans le journal.

## Options

-   `--url ` : URL WebSocket de la Passerelle (par défaut : configuration ou `ws://127.0.0.1:`)
-   `--token ` : Jeton de la Passerelle (si requis)
-   `--password ` : Mot de passe de la Passerelle (si requis)
-   `--session ` : Clé de session (par défaut : `main`, ou `global` lorsque la portée est globale)
-   `--deliver` : Livrer les réponses de l'assistant au fournisseur (désactivé par défaut)
-   `--thinking ` : Remplacer le niveau de réflexion pour les envois
-   `--timeout-ms ` : Délai d'attente de l'agent en ms (par défaut : `agents.defaults.timeoutSeconds`)

Remarque : lorsque vous définissez `--url`, le TUI ne revient pas aux informations d'identification de la configuration ou de l'environnement. Passez `--token` ou `--password` explicitement. L'absence d'informations d'identification explicites est une erreur.

## Dépannage

Aucune sortie après l'envoi d'un message :

-   Exécutez `/status` dans le TUI pour confirmer que la Passerelle est connectée et inactive/occupée.
-   Vérifiez les journaux de la Passerelle : `openclaw logs --follow`.
-   Confirmez que l'agent peut s'exécuter : `openclaw status` et `openclaw models status`.
-   Si vous attendez des messages dans un canal de discussion, activez la livraison (`/deliver on` ou `--deliver`).
-   `--history-limit ` : Nombre d'entrées d'historique à charger (200 par défaut)

## Dépannage de la connexion

-   `déconnecté` : assurez-vous que la Passerelle est en cours d'exécution et que vos `--url/--token/--password` sont corrects.
-   Aucun agent dans le sélecteur : vérifiez `openclaw agents list` et votre configuration de routage.
-   Sélecteur de session vide : vous êtes peut-être en portée globale ou n'avez pas encore de sessions.

[WebChat](./webchat.md)

---