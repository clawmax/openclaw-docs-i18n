

  Messages et livraison

  
# Streaming et Découpage

OpenClaw possède deux couches de streaming distinctes :

-   **Streaming par blocs (canaux) :** émet des **blocs** complets au fur et à mesure que l'assistant écrit. Ce sont des messages de canal normaux (pas des deltas de tokens).
-   **Streaming d'aperçu (Telegram/Discord/Slack) :** met à jour un **message d'aperçu** temporaire pendant la génération.

Il n'y a **pas de streaming véritable par deltas de tokens** vers les messages des canaux aujourd'hui. Le streaming d'aperçu est basé sur les messages (envoi + modifications/ajouts).

## Streaming par blocs (messages de canal)

Le streaming par blocs envoie la sortie de l'assistant par morceaux grossiers dès qu'elle est disponible.

```
Model output
  └─ text_delta/events
       ├─ (blockStreamingBreak=text_end)
       │    └─ le découpeur émet des blocs au fur et à mesure que le tampon grossit
       └─ (blockStreamingBreak=message_end)
            └─ le découpeur vide le tampon à message_end
                   └─ envoi sur le canal (réponses par blocs)
```

Légende :

-   `text_delta/events` : événements du flux du modèle (peuvent être épars pour les modèles non-streaming).
-   `chunker` : `EmbeddedBlockChunker` appliquant des limites min/max + une préférence de coupure.
-   `channel send` : messages sortants réels (réponses par blocs).

**Contrôles :**

-   `agents.defaults.blockStreamingDefault` : `"on"`/`"off"` (désactivé par défaut).
-   Surcharges par canal : `*.blockStreaming` (et variantes par compte) pour forcer `"on"`/`"off"` par canal.
-   `agents.defaults.blockStreamingBreak` : `"text_end"` ou `"message_end"`.
-   `agents.defaults.blockStreamingChunk` : `{ minChars, maxChars, breakPreference? }`.
-   `agents.defaults.blockStreamingCoalesce` : `{ minChars?, maxChars?, idleMs? }` (fusionne les blocs streamés avant envoi).
-   Limite stricte par canal : `*.textChunkLimit` (ex. : `channels.whatsapp.textChunkLimit`).
-   Mode de découpage par canal : `*.chunkMode` (`length` par défaut, `newline` divise sur les lignes vides (limites de paragraphe) avant le découpage par longueur).
-   Limite souple Discord : `channels.discord.maxLinesPerMessage` (17 par défaut) divise les réponses longues pour éviter le découpage dans l'interface.

**Sémantique des limites :**

-   `text_end` : streamer les blocs dès que le découpeur les émet ; vider le tampon à chaque `text_end`.
-   `message_end` : attendre que le message de l'assistant se termine, puis vider la sortie tamponnée.

`message_end` utilise toujours le découpeur si le texte tamponné dépasse `maxChars`, il peut donc émettre plusieurs morceaux à la fin.

## Algorithme de découpage (bornes basse/haute)

Le découpage en blocs est implémenté par `EmbeddedBlockChunker` :

-   **Borne basse :** n'émet pas tant que le tampon n'atteint pas `minChars` (sauf si forcé).
-   **Borne haute :** préfère les coupures avant `maxChars` ; si forcé, divise à `maxChars`.
-   **Préférence de coupure :** `paragraph` → `newline` → `sentence` → `whitespace` → coupure dure.
-   **Clôtures de code :** ne jamais diviser à l'intérieur des clôtures ; quand forcé à `maxChars`, ferme + rouvre la clôture pour garder le Markdown valide.

`maxChars` est limité par le `textChunkLimit` du canal, vous ne pouvez donc pas dépasser les limites par canal.

## Fusion (regrouper les blocs streamés)

Lorsque le streaming par blocs est activé, OpenClaw peut **fusionner les morceaux de blocs consécutifs** avant de les envoyer. Cela réduit le "spam de lignes uniques" tout en fournissant une sortie progressive.

-   La fusion attend des **pauses d'inactivité** (`idleMs`) avant de vider le tampon.
-   Les tampons sont limités par `maxChars` et se videront s'ils le dépassent.
-   `minChars` empêche l'envoi de fragments minuscules jusqu'à ce que suffisamment de texte s'accumule (la vidange finale envoie toujours le texte restant).
-   Le séparateur est dérivé de `blockStreamingChunk.breakPreference` (`paragraph` → `\n\n`, `newline` → `\n`, `sentence` → espace).
-   Les surcharges par canal sont disponibles via `*.blockStreamingCoalesce` (y compris les configurations par compte).
-   La valeur par défaut de `minChars` pour la fusion est augmentée à 1500 pour Signal/Slack/Discord sauf si elle est surchargée.

## Rythme naturel entre les blocs

Lorsque le streaming par blocs est activé, vous pouvez ajouter une **pause aléatoire** entre les réponses par blocs (après le premier bloc). Cela rend les réponses multi-bulles plus naturelles.

-   Configuration : `agents.defaults.humanDelay` (surcharge par agent via `agents.list[].humanDelay`).
-   Modes : `off` (par défaut), `natural` (800–2500ms), `custom` (`minMs`/`maxMs`).
-   S'applique uniquement aux **réponses par blocs**, pas aux réponses finales ou aux résumés d'outils.

## "Streamer des morceaux ou tout"

Cela correspond à :

-   **Streamer des morceaux :** `blockStreamingDefault: "on"` + `blockStreamingBreak: "text_end"` (émettre au fur et à mesure). Les canaux non-Telegram ont aussi besoin de `*.blockStreaming: true`.
-   **Streamer tout à la fin :** `blockStreamingBreak: "message_end"` (vider une fois, éventuellement plusieurs morceaux si très long).
-   **Pas de streaming par blocs :** `blockStreamingDefault: "off"` (seulement la réponse finale).

**Note canal :** Le streaming par blocs est **désactivé sauf si** `*.blockStreaming` est explicitement défini sur `true`. Les canaux peuvent streamer un aperçu en direct (`channels..streaming`) sans réponses par blocs. Rappel de l'emplacement de la configuration : les valeurs par défaut `blockStreaming*` se trouvent sous `agents.defaults`, pas dans la configuration racine.

## Modes de streaming d'aperçu

Clé canonique : `channels..streaming` Modes :

-   `off` : désactive le streaming d'aperçu.
-   `partial` : un seul aperçu qui est remplacé par le texte le plus récent.
-   `block` : l'aperçu se met à jour par étapes découpées/ajoutées.
-   `progress` : aperçu de progression/statut pendant la génération, réponse finale à la fin.

### Correspondance par canal

| Canal | `off` | `partial` | `block` | `progress` |
| --- | --- | --- | --- | --- |
| Telegram | ✅ | ✅ | ✅ | correspond à `partial` |
| Discord | ✅ | ✅ | ✅ | correspond à `partial` |
| Slack | ✅ | ✅ | ✅ | ✅ |

Uniquement Slack :

-   `channels.slack.nativeStreaming` active/désactive les appels d'API de streaming natif de Slack lorsque `streaming=partial` (par défaut : `true`).

Migration des clés héritées :

-   Telegram : `streamMode` + booléen `streaming` migrent automatiquement vers l'énumération `streaming`.
-   Discord : `streamMode` + booléen `streaming` migrent automatiquement vers l'énumération `streaming`.
-   Slack : `streamMode` migre automatiquement vers l'énumération `streaming` ; le booléen `streaming` migre automatiquement vers `nativeStreaming`.

### Comportement à l'exécution

Telegram :

-   Utilise l'API Bot `sendMessageDraft` dans les messages privés quand disponible, et `sendMessage` + `editMessageText` pour les mises à jour d'aperçu dans les groupes/sujets.
-   Le streaming d'aperçu est ignoré lorsque le streaming par blocs Telegram est explicitement activé (pour éviter un double streaming).
-   `/reasoning stream` peut écrire le raisonnement dans l'aperçu.

Discord :

-   Utilise l'envoi + la modification de messages d'aperçu.
-   Le mode `block` utilise le découpage de brouillon (`draftChunk`).
-   Le streaming d'aperçu est ignoré lorsque le streaming par blocs Discord est explicitement activé.

Slack :

-   `partial` peut utiliser le streaming natif de Slack (`chat.startStream`/`append`/`stop`) quand disponible.
-   `block` utilise des aperçus de brouillon de type ajout.
-   `progress` utilise un texte d'aperçu de statut, puis la réponse finale.

[Messages](./messages.md)[Politique de nouvelle tentative](./retry.md)