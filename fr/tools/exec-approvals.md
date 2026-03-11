

  Outils intégrés

  
# Approbations Exec

Les approbations exec sont **l'application compagnon / le garde-fou de l'hôte du nœud** permettant à un agent sandboxé d'exécuter des commandes sur un hôte réel (`gateway` ou `node`). Considérez-les comme un verrouillage de sécurité : les commandes ne sont autorisées que lorsque la politique + la liste d'autorisation + (optionnellement) l'approbation de l'utilisateur sont toutes d'accord. Les approbations exec s'ajoutent à la politique des outils et au contrôle élevé (sauf si le mode élevé est défini sur `full`, ce qui ignore les approbations). La politique effective est la plus stricte entre `tools.exec.*` et les valeurs par défaut des approbations ; si un champ d'approbation est omis, la valeur `tools.exec` est utilisée. Si l'interface utilisateur de l'application compagnon n'est **pas disponible**, toute demande nécessitant une invite est résolue par le **secours de demande** (par défaut : refuser).

## Où cela s'applique

Les approbations exec sont appliquées localement sur l'hôte d'exécution :

-   **hôte de la passerelle** → processus `openclaw` sur la machine de la passerelle
-   **hôte du nœud** → exécuteur de nœud (application compagnon macOS ou hôte de nœud sans interface)

Répartition macOS :

-   Le **service de l'hôte du nœud** transmet `system.run` à **l'application macOS** via IPC local.
-   **L'application macOS** applique les approbations + exécute la commande dans le contexte de l'interface utilisateur.

## Paramètres et stockage

Les approbations résident dans un fichier JSON local sur l'hôte d'exécution : `~/.openclaw/exec-approvals.json` Exemple de schéma :

```json
{
  "version": 1,
  "socket": {
    "path": "~/.openclaw/exec-approvals.sock",
    "token": "base64url-token"
  },
  "defaults": {
    "security": "deny",
    "ask": "on-miss",
    "askFallback": "deny",
    "autoAllowSkills": false
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "askFallback": "deny",
      "autoAllowSkills": true,
      "allowlist": [
        {
          "id": "B0C8C0B3-2C2D-4F8A-9A3C-5A4B3C2D1E0F",
          "pattern": "~/Projects/**/bin/rg",
          "lastUsedAt": 1737150000000,
          "lastUsedCommand": "rg -n TODO",
          "lastResolvedPath": "/Users/user/Projects/.../bin/rg"
        }
      ]
    }
  }
}
```

## Paramètres de politique

### Sécurité (exec.security)

-   **deny** : bloquer toutes les demandes d'exécution hôte.
-   **allowlist** : autoriser uniquement les commandes figurant dans la liste d'autorisation.
-   **full** : tout autoriser (équivalent au mode élevé).

### Demande (exec.ask)

-   **off** : ne jamais demander.
-   **on-miss** : demander uniquement lorsque la liste d'autorisation ne correspond pas.
-   **always** : demander pour chaque commande.

### Secours de demande (askFallback)

Si une demande est requise mais qu'aucune interface utilisateur n'est accessible, le secours décide :

-   **deny** : bloquer.
-   **allowlist** : autoriser uniquement si la liste d'autorisation correspond.
-   **full** : autoriser.

## Liste d'autorisation (par agent)

Les listes d'autorisation sont **par agent**. Si plusieurs agents existent, basculez l'agent que vous modifiez dans l'application macOS. Les motifs sont des **correspondances de glob insensibles à la casse**. Les motifs doivent se résoudre en **chemins de binaires** (les entrées basées uniquement sur le nom de base sont ignorées). Les entrées héritées `agents.default` sont migrées vers `agents.main` au chargement. Exemples :

-   `~/Projects/**/bin/peekaboo`
-   `~/.local/bin/*`
-   `/opt/homebrew/bin/rg`

Chaque entrée de la liste d'autorisation suit :

-   **id** UUID stable utilisé pour l'identité dans l'interface utilisateur (optionnel)
-   **dernière utilisation** horodatage
-   **dernière commande utilisée**
-   **dernier chemin résolu**

## Auto-autorisation des CLI de compétences

Lorsque **Auto-autorisation des CLI de compétences** est activé, les exécutables référencés par des compétences connues sont traités comme étant sur liste d'autorisation sur les nœuds (nœud macOS ou hôte de nœud sans interface). Cela utilise `skills.bins` via le RPC de la passerelle pour récupérer la liste des binaires de compétences. Désactivez cette option si vous souhaitez des listes d'autorisation manuelles strictes.

## Binaires sûrs (stdin uniquement)

`tools.exec.safeBins` définit une petite liste de binaires **stdin uniquement** (par exemple `jq`) qui peuvent s'exécuter en mode liste d'autorisation **sans** entrées explicites dans la liste d'autorisation. Les binaires sûrs rejettent les arguments de fichiers positionnels et les jetons de type chemin, ils ne peuvent donc opérer que sur le flux entrant. La validation est déterministe à partir de la forme des arguments uniquement (pas de vérifications d'existence sur le système de fichiers de l'hôte), ce qui empêche un comportement d'oracle d'existence de fichier basé sur les différences d'autorisation/refus. Les options orientées fichier sont refusées pour les binaires sûrs par défaut (par exemple `sort -o`, `sort --output`, `sort --files0-from`, `sort --compress-program`, `wc --files0-from`, `jq -f/--from-file`, `grep -f/--file`). Les binaires sûrs appliquent également une politique de drapeaux explicite par binaire pour les options qui rompent le comportement stdin uniquement (par exemple `sort -o/--output/--compress-program` et les drapeaux récursifs de grep). Les binaires sûrs forcent également les jetons d'arguments à être traités comme **texte littéral** au moment de l'exécution (pas d'expansion de glob et pas d'expansion de `$VARS`) pour les segments stdin uniquement, de sorte que des motifs comme `*` ou `$HOME/...` ne peuvent pas être utilisés pour introduire en contrebande des lectures de fichiers. Les binaires sûrs doivent également se résoudre à partir de répertoires binaires de confiance (valeurs par défaut du système plus le `PATH` du processus de la passerelle au démarrage). Cela bloque les tentatives de détournement de PATH à la portée de la demande. L'enchaînement de shell et les redirections ne sont pas auto-autorisés en mode liste d'autorisation. L'enchaînement de shell (`&&`, `||`, `;`) est autorisé lorsque chaque segment de premier niveau satisfait la liste d'autorisation (y compris les binaires sûrs ou l'auto-autorisation des compétences). Les redirections restent non prises en charge en mode liste d'autorisation. La substitution de commande (`$()` / backticks) est rejetée lors de l'analyse de la liste d'autorisation, y compris à l'intérieur des guillemets doubles ; utilisez des guillemets simples si vous avez besoin du texte littéral `$()`. Sur les approbations de l'application compagnon macOS, le texte shell brut contenant une syntaxe de contrôle ou d'expansion de shell (`&&`, `||`, `;`, `|`, ```, `$`, `<`, `>`, `(`, `)`) est traité comme un échec de liste d'autorisation, sauf si le binaire shell lui-même est sur liste d'autorisation. Binaires sûrs par défaut : `jq`, `cut`, `uniq`, `head`, `tail`, `tr`, `wc`. `grep` et `sort` ne figurent pas dans la liste par défaut. Si vous les ajoutez, conservez des entrées explicites dans la liste d'autorisation pour leurs workflows non stdin. Pour `grep` en mode binaire sûr, fournissez le motif avec `-e`/`--regexp` ; la forme positionnelle du motif est rejetée pour que les opérandes de fichier ne puissent pas être introduits en contrebande comme positionnels ambigus.

## Édition via l'interface de contrôle

Utilisez la carte **Interface de contrôle → Nœuds → Approbations Exec** pour modifier les valeurs par défaut, les remplacements par agent et les listes d'autorisation. Choisissez une portée (Valeurs par défaut ou un agent), ajustez la politique, ajoutez/supprimez des motifs de liste d'autorisation, puis **Enregistrer**. L'interface affiche les métadonnées de **dernière utilisation** par motif pour que vous puissiez garder la liste propre. Le sélecteur de cible choisit **Passerelle** (approbations locales) ou un **Nœud**. Les nœuds doivent annoncer `system.execApprovals.get/set` (application macOS ou hôte de nœud sans interface). Si un nœud n'annonce pas encore les approbations exec, modifiez directement son fichier local `~/.openclaw/exec-approvals.json`. CLI : `openclaw approvals` prend en charge l'édition sur la passerelle ou le nœud (voir [CLI Approbations](/cli/approvals)).

## Flux d'approbation

Lorsqu'une demande est requise, la passerelle diffuse `exec.approval.requested` aux clients opérateurs. L'interface de contrôle et l'application macOS la résolvent via `exec.approval.resolve`, puis la passerelle transmet la demande approuvée à l'hôte du nœud. Lorsque des approbations sont requises, l'outil exec retourne immédiatement avec un identifiant d'approbation. Utilisez cet identifiant pour corréler les événements système ultérieurs (`Exec finished` / `Exec denied`). Si aucune décision n'arrive avant le délai d'attente, la demande est traitée comme un délai d'approbation et signalée comme un motif de refus. La boîte de dialogue de confirmation inclut :

-   commande + arguments
-   répertoire de travail courant (cwd)
-   identifiant de l'agent
-   chemin de l'exécutable résolu
-   hôte + métadonnées de politique

Actions :

-   **Autoriser une fois** → exécuter maintenant
-   **Toujours autoriser** → ajouter à la liste d'autorisation + exécuter
-   **Refuser** → bloquer

## Transfert des approbations vers les canaux de discussion

Vous pouvez transférer les demandes d'approbation exec vers n'importe quel canal de discussion (y compris les canaux de plugins) et les approuver avec `/approve`. Cela utilise le pipeline de livraison sortant normal. Configuration :

```json
{
  approvals: {
    exec: {
      enabled: true,
      mode: "session", // "session" | "targets" | "both"
      agentFilter: ["main"],
      sessionFilter: ["discord"], // sous-chaîne ou regex
      targets: [
        { channel: "slack", to: "U12345678" },
        { channel: "telegram", to: "123456789" },
      ],
    },
  },
}
```

Répondre dans le chat :

```bash
/approve  allow-once
/approve  allow-always
/approve  deny
```

### Flux IPC macOS

```
Gateway -> Node Service (WS)
                 |  IPC (UDS + token + HMAC + TTL)
                 v
             Mac App (UI + approvals + system.run)
```

Notes de sécurité :

-   Mode socket Unix `0600`, token stocké dans `exec-approvals.json`.
-   Vérification de pair même UID.
-   Défi/réponse (nonce + HMAC token + hachage de la demande) + TTL court.

## Événements système

Le cycle de vie de l'exécution est signalé sous forme de messages système :

-   `Exec running` (uniquement si la commande dépasse le seuil de notification d'exécution)
-   `Exec finished`
-   `Exec denied`

Ceux-ci sont publiés dans la session de l'agent après que le nœud a signalé l'événement. Les approbations exec sur l'hôte de la passerelle émettent les mêmes événements de cycle de vie lorsque la commande se termine (et optionnellement lorsqu'elle s'exécute plus longtemps que le seuil). Les exécutions soumises à approbation réutilisent l'identifiant d'approbation comme `runId` dans ces messages pour une corrélation facile.

## Implications

-   **full** est puissant ; préférez les listes d'autorisation lorsque c'est possible.
-   **ask** vous garde dans la boucle tout en permettant des approbations rapides.
-   Les listes d'autorisation par agent empêchent les approbations d'un agent de fuiter vers d'autres.
-   Les approbations ne s'appliquent qu'aux demandes d'exécution hôte provenant d'**expéditeurs autorisés**. Les expéditeurs non autorisés ne peuvent pas émettre `/exec`.
-   `/exec security=full` est une commodité au niveau de la session pour les opérateurs autorisés et ignore les approbations par conception. Pour bloquer fermement l'exécution hôte, définissez la sécurité des approbations sur `deny` ou refusez l'outil `exec` via la politique des outils.

Liens connexes :

-   [Outil Exec](/tools/exec)
-   [Mode élevé](/tools/elevated)
-   [Compétences](/tools/skills)

[Outil Exec](/tools/exec)[Firecrawl](/tools/firecrawl)