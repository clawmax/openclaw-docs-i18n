

  Configuration et exploitation

  
# Journalisation

Pour une vue d'ensemble utilisateur (CLI + Interface de Contrôle + configuration), consultez [/logging](../logging.md). OpenClaw a deux "surfaces" de journalisation :

-   **Sortie console** (ce que vous voyez dans le terminal / l'Interface de Débogage).
-   **Logs fichiers** (lignes JSON) écrits par le logger de la passerelle.

## Logger basé sur fichier

-   Le fichier de log rotatif par défaut se trouve sous `/tmp/openclaw/` (un fichier par jour) : `openclaw-YYYY-MM-DD.log`
    -   La date utilise le fuseau horaire local de l'hôte de la passerelle.
-   Le chemin du fichier de log et son niveau peuvent être configurés via `~/.openclaw/openclaw.json` :
    -   `logging.file`
    -   `logging.level`

Le format du fichier est un objet JSON par ligne. L'onglet Logs de l'Interface de Contrôle suit ce fichier via la passerelle (`logs.tail`). La CLI peut faire de même :

```bash
openclaw logs --follow
```

**Verbose vs. niveaux de log**

-   Les **logs fichiers** sont contrôlés exclusivement par `logging.level`.
-   `--verbose` n'affecte que la **verbosité de la console** (et le style des logs WS) ; il **n'augmente pas** le niveau de log des fichiers.
-   Pour capturer les détails uniquement visibles en mode verbose dans les logs fichiers, définissez `logging.level` sur `debug` ou `trace`.

## Capture console

La CLI capture `console.log/info/warn/error/debug/trace` et les écrit dans les logs fichiers, tout en les affichant sur stdout/stderr. Vous pouvez ajuster la verbosité de la console indépendamment via :

-   `logging.consoleLevel` (par défaut `info`)
-   `logging.consoleStyle` (`pretty` | `compact` | `json`)

## Rédaction des résumés d'outils

Les résumés d'outils verbeux (ex. `🛠️ Exec: ...`) peuvent masquer les jetons sensibles avant qu'ils n'atteignent le flux console. Cela concerne **uniquement les outils** et ne modifie pas les logs fichiers.

-   `logging.redactSensitive`: `off` | `tools` (par défaut : `tools`)
-   `logging.redactPatterns`: tableau de chaînes d'expressions régulières (remplace les valeurs par défaut)
    -   Utilisez des chaînes d'expressions régulières brutes (auto `gi`), ou `/pattern/flags` si vous avez besoin de drapeaux personnalisés.
    -   Les correspondances sont masquées en conservant les 6 premiers + 4 derniers caractères (longueur >= 18), sinon `***`.
    -   Les valeurs par défaut couvrent les assignations de clés courantes, les drapeaux CLI, les champs JSON, les en-têtes bearer, les blocs PEM et les préfixes de jetons populaires.

## Logs WebSocket de la passerelle

La passerelle affiche les logs du protocole WebSocket dans deux modes :

-   **Mode normal (sans `--verbose`)** : seuls les résultats RPC "intéressants" sont affichés :
    -   les erreurs (`ok=false`)
    -   les appels lents (seuil par défaut : `>= 50ms`)
    -   les erreurs d'analyse
-   **Mode verbeux (`--verbose`)** : affiche tout le trafic de requêtes/réponses WS.

### Style des logs WS

`openclaw gateway` supporte un commutateur de style par passerelle :

-   `--ws-log auto` (par défaut) : le mode normal est optimisé ; le mode verbeux utilise une sortie compacte
-   `--ws-log compact` : sortie compacte (requête/réponse appariée) en mode verbeux
-   `--ws-log full` : sortie complète par trame en mode verbeux
-   `--compact` : alias pour `--ws-log compact`

Exemples :

```bash
# optimisé (uniquement erreurs/lent)
openclaw gateway

# affiche tout le trafic WS (apparié)
openclaw gateway --verbose --ws-log compact

# affiche tout le trafic WS (méta complet)
openclaw gateway --verbose --ws-log full
```

## Formatage console (journalisation par sous-système)

Le formateur de console est **conscient du TTY** et affiche des lignes cohérentes et préfixées. Les loggers de sous-système gardent la sortie groupée et facile à parcourir. Comportement :

-   **Préfixes de sous-système** sur chaque ligne (ex. `[gateway]`, `[canvas]`, `[tailscale]`)
-   **Couleurs par sous-système** (stable par sous-système) plus coloration par niveau
-   **Couleur lorsque la sortie est un TTY ou que l'environnement ressemble à un terminal riche** (`TERM`/`COLORTERM`/`TERM_PROGRAM`), respecte `NO_COLOR`
-   **Préfixes de sous-système raccourcis** : supprime le préfixe `gateway/` + `channels/`, conserve les 2 derniers segments (ex. `whatsapp/outbound`)
-   **Sous-loggers par sous-système** (préfixe automatique + champ structuré `{ subsystem }`)
-   **`logRaw()`** pour les sorties QR/UX (pas de préfixe, pas de formatage)
-   **Styles console** (ex. `pretty | compact | json`)
-   **Niveau de log console** séparé du niveau de log fichier (le fichier conserve tous les détails quand `logging.level` est défini sur `debug`/`trace`)
-   **Les corps des messages WhatsApp** sont journalisés au niveau `debug` (utilisez `--verbose` pour les voir)

Cela maintient la stabilité des logs fichiers existants tout en rendant la sortie interactive facile à parcourir.

[Docteur](./doctor.md)[Verrou de la passerelle](./gateway-lock.md)

---