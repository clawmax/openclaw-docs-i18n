

  Outils intégrés

  
# Diffs

`diffs` est un outil plugin optionnel avec un guide système intégré concis et une compétence associée qui transforme le contenu des changements en un artefact diff en lecture seule pour les agents. Il accepte soit :

-   le texte `before` et `after`
-   un `patch` unifié

Il peut retourner :

-   une URL de visualisation de passerelle pour une présentation sur le canevas
-   un chemin de fichier rendu (PNG ou PDF) pour la livraison dans un message
-   les deux sorties en un seul appel

Lorsqu'il est activé, le plugin ajoute un guide d'utilisation concis dans l'espace du prompt système et expose également une compétence détaillée pour les cas où l'agent a besoin d'instructions plus complètes.

## Démarrage rapide

1.  Activez le plugin.
2.  Appelez `diffs` avec `mode: "view"` pour les flux orientés canevas.
3.  Appelez `diffs` avec `mode: "file"` pour les flux de livraison de fichiers dans le chat.
4.  Appelez `diffs` avec `mode: "both"` lorsque vous avez besoin des deux artefacts.

## Activer le plugin

```json
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
      },
    },
  },
}
```

## Désactiver le guide système intégré

Si vous souhaitez garder l'outil `diffs` activé mais désactiver son guide intégré dans le prompt système, définissez `plugins.entries.diffs.hooks.allowPromptInjection` sur `false` :

```json
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
        hooks: {
          allowPromptInjection: false,
        },
      },
    },
  },
}
```

Cela bloque le hook `before_prompt_build` du plugin diffs tout en gardant le plugin, l'outil et la compétence associée disponibles. Si vous souhaitez désactiver à la fois le guide et l'outil, désactivez plutôt le plugin.

## Flux de travail typique de l'agent

1.  L'agent appelle `diffs`.
2.  L'agent lit les champs `details`.
3.  L'agent soit :
    -   ouvre `details.viewerUrl` avec `canvas present`
    -   envoie `details.filePath` avec `message` en utilisant `path` ou `filePath`
    -   fait les deux

## Exemples d'entrée

Avant et après :

```json
{
  "before": "# Hello\n\nOne",
  "after": "# Hello\n\nTwo",
  "path": "docs/example.md",
  "mode": "view"
}
```

Patch :

```json
{
  "patch": "diff --git a/src/example.ts b/src/example.ts\n--- a/src/example.ts\n+++ b/src/example.ts\n@@ -1 +1 @@\n-const x = 1;\n+const x = 2;\n",
  "mode": "both"
}
```

## Référence des entrées de l'outil

Tous les champs sont optionnels sauf indication contraire :

-   `before` (`string`) : texte original. Requis avec `after` lorsque `patch` est omis.
-   `after` (`string`) : texte mis à jour. Requis avec `before` lorsque `patch` est omis.
-   `patch` (`string`) : texte de diff unifié. Mutuellement exclusif avec `before` et `after`.
-   `path` (`string`) : nom de fichier d'affichage pour le mode avant et après.
-   `lang` (`string`) : indication de remplacement de langue pour le mode avant et après.
-   `title` (`string`) : remplacement du titre du visualiseur.
-   `mode` (`"view" | "file" | "both"`) : mode de sortie. Par défaut à la valeur par défaut du plugin `defaults.mode`.
-   `theme` (`"light" | "dark"`) : thème du visualiseur. Par défaut à la valeur par défaut du plugin `defaults.theme`.
-   `layout` (`"unified" | "split"`) : disposition du diff. Par défaut à la valeur par défaut du plugin `defaults.layout`.
-   `expandUnchanged` (`boolean`) : développer les sections inchangées lorsque le contexte complet est disponible. Option par appel uniquement (pas une clé par défaut du plugin).
-   `fileFormat` (`"png" | "pdf"`) : format de fichier rendu. Par défaut à la valeur par défaut du plugin `defaults.fileFormat`.
-   `fileQuality` (`"standard" | "hq" | "print"`) : préréglage de qualité pour le rendu PNG ou PDF.
-   `fileScale` (`number`) : remplacement de l'échelle de l'appareil (`1`\-`4`).
-   `fileMaxWidth` (`number`) : largeur de rendu maximale en pixels CSS (`640`\-`2400`).
-   `ttlSeconds` (`number`) : TTL de l'artefact du visualiseur en secondes. Par défaut 1800, max 21600.
-   `baseUrl` (`string`) : remplacement de l'origine de l'URL du visualiseur. Doit être `http` ou `https`, sans requête/fragment.

Validation et limites :

-   `before` et `after` maximum 512 KiB chacun.
-   `patch` maximum 2 MiB.
-   `path` maximum 2048 octets.
-   `lang` maximum 128 octets.
-   `title` maximum 1024 octets.
-   Limite de complexité du patch : max 128 fichiers et 120000 lignes au total.
-   `patch` et `before` ou `after` ensemble sont rejetés.
-   Limites de sécurité des fichiers rendus (s'appliquent au PNG et PDF) :
    -   `fileQuality: "standard"` : max 8 MP (8 000 000 pixels rendus).
    -   `fileQuality: "hq"` : max 14 MP (14 000 000 pixels rendus).
    -   `fileQuality: "print"` : max 24 MP (24 000 000 pixels rendus).
    -   Le PDF a également un maximum de 50 pages.

## Contrat des détails de sortie

L'outil retourne des métadonnées structurées sous `details`. Champs partagés pour les modes qui créent un visualiseur :

-   `artifactId`
-   `viewerUrl`
-   `viewerPath`
-   `title`
-   `expiresAt`
-   `inputKind`
-   `fileCount`
-   `mode`

Champs de fichier lorsque le PNG ou PDF est rendu :

-   `filePath`
-   `path` (même valeur que `filePath`, pour la compatibilité avec l'outil message)
-   `fileBytes`
-   `fileFormat`
-   `fileQuality`
-   `fileScale`
-   `fileMaxWidth`

Résumé du comportement des modes :

-   `mode: "view"` : uniquement les champs du visualiseur.
-   `mode: "file"` : uniquement les champs de fichier, pas d'artefact de visualiseur.
-   `mode: "both"` : champs du visualiseur plus champs de fichier. Si le rendu du fichier échoue, le visualiseur retourne quand même avec `fileError`.

## Sections inchangées réduites

-   Le visualiseur peut afficher des lignes comme `N lignes non modifiées`.
-   Les contrôles de développement sur ces lignes sont conditionnels et non garantis pour chaque type d'entrée.
-   Les contrôles de développement apparaissent lorsque le diff rendu a des données de contexte extensibles, ce qui est typique pour une entrée avant et après.
-   Pour de nombreuses entrées de patch unifié, les corps de contexte omis ne sont pas disponibles dans les blocs de patch analysés, donc la ligne peut apparaître sans contrôles de développement. C'est un comportement attendu.
-   `expandUnchanged` s'applique uniquement lorsque le contexte extensible existe.

## Valeurs par défaut du plugin

Définissez les valeurs par défaut à l'échelle du plugin dans `~/.openclaw/openclaw.json` :

```json
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
        config: {
          defaults: {
            fontFamily: "Fira Code",
            fontSize: 15,
            lineSpacing: 1.6,
            layout: "unified",
            showLineNumbers: true,
            diffIndicators: "bars",
            wordWrap: true,
            background: true,
            theme: "dark",
            fileFormat: "png",
            fileQuality: "standard",
            fileScale: 2,
            fileMaxWidth: 960,
            mode: "both",
          },
        },
      },
    },
  },
}
```

Valeurs par défaut supportées :

-   `fontFamily`
-   `fontSize`
-   `lineSpacing`
-   `layout`
-   `showLineNumbers`
-   `diffIndicators`
-   `wordWrap`
-   `background`
-   `theme`
-   `fileFormat`
-   `fileQuality`
-   `fileScale`
-   `fileMaxWidth`
-   `mode`

Les paramètres explicites de l'outil remplacent ces valeurs par défaut.

## Configuration de sécurité

-   `security.allowRemoteViewer` (`boolean`, par défaut `false`)
    -   `false` : les requêtes non-loopback vers les routes du visualiseur sont refusées.
    -   `true` : les visualiseurs distants sont autorisés si le chemin tokenisé est valide.

Exemple :

```json
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
        config: {
          security: {
            allowRemoteViewer: false,
          },
        },
      },
    },
  },
}
```

## Cycle de vie et stockage des artefacts

-   Les artefacts sont stockés sous le sous-dossier temporaire : `$TMPDIR/openclaw-diffs`.
-   Les métadonnées de l'artefact du visualiseur contiennent :
    -   un ID d'artefact aléatoire (20 caractères hexadécimaux)
    -   un token aléatoire (48 caractères hexadécimaux)
    -   `createdAt` et `expiresAt`
    -   le chemin stocké `viewer.html`
-   Le TTL par défaut du visualiseur est de 30 minutes lorsqu'il n'est pas spécifié.
-   Le TTL maximum accepté pour le visualiseur est de 6 heures.
-   Le nettoyage s'exécute de manière opportuniste après la création d'un artefact.
-   Les artefacts expirés sont supprimés.
-   Un nettoyage de secours supprime les dossiers obsolètes de plus de 24 heures lorsque les métadonnées sont manquantes.

## URL du visualiseur et comportement réseau

Route du visualiseur :

-   `/plugins/diffs/view/{artifactId}/{token}`

Assets du visualiseur :

-   `/plugins/diffs/assets/viewer.js`
-   `/plugins/diffs/assets/viewer-runtime.js`

Comportement de construction de l'URL :

-   Si `baseUrl` est fourni, il est utilisé après validation stricte.
-   Sans `baseUrl`, l'URL du visualiseur par défaut est le loopback `127.0.0.1`.
-   Si le mode de liaison de la passerelle est `custom` et que `gateway.customBindHost` est défini, cet hôte est utilisé.

Règles pour `baseUrl` :

-   Doit être `http://` ou `https://`.
-   Les requêtes et fragments sont rejetés.
-   L'origine plus un chemin de base optionnel est autorisé.

## Modèle de sécurité

Renforcement du visualiseur :

-   Loopback uniquement par défaut.
-   Chemins de visualiseur tokenisés avec validation stricte de l'ID et du token.
-   CSP de la réponse du visualiseur :
    -   `default-src 'none'`
    -   scripts et assets uniquement depuis soi-même
    -   pas de `connect-src` sortant
-   Limitation des échecs d'accès distant lorsque l'accès distant est activé :
    -   40 échecs par 60 secondes
    -   verrouillage de 60 secondes (`429 Too Many Requests`)

Renforcement du rendu de fichier :

-   Le routage des requêtes du navigateur de capture d'écran est interdit par défaut.
-   Seuls les assets locaux du visualiseur provenant de `http://127.0.0.1/plugins/diffs/assets/*` sont autorisés.
-   Les requêtes réseau externes sont bloquées.

## Exigences du navigateur pour le mode fichier

`mode: "file"` et `mode: "both"` nécessitent un navigateur compatible Chromium. Ordre de résolution :

1.  `browser.executablePath` dans la configuration OpenClaw.
2.  Variables d'environnement :
    -   `OPENCLAW_BROWSER_EXECUTABLE_PATH`
    -   `BROWSER_EXECUTABLE_PATH`
    -   `PLAYWRIGHT_CHROMIUM_EXECUTABLE_PATH`
3.  Secours de découverte de commande/chemin de la plateforme.

Texte d'échec courant :

-   `Diff PNG/PDF rendering requires a Chromium-compatible browser...`

Corrigez en installant Chrome, Chromium, Edge ou Brave, ou en définissant l'une des options de chemin d'exécutable ci-dessus.

## Dépannage

Erreurs de validation des entrées :

-   `Provide patch or both before and after text.`
    -   Incluez à la fois `before` et `after`, ou fournissez `patch`.
-   `Provide either patch or before/after input, not both.`
    -   Ne mélangez pas les modes d'entrée.
-   `Invalid baseUrl: ...`
    -   Utilisez une origine `http(s)` avec un chemin optionnel, sans requête/fragment.
-   `{field} exceeds maximum size (...)`
    -   Réduisez la taille de la charge utile.
-   Rejet d'un patch volumineux
    -   Réduisez le nombre de fichiers ou le total de lignes du patch.

Problèmes d'accessibilité du visualiseur :

-   L'URL du visualiseur se résout par défaut en `127.0.0.1`.
-   Pour les scénarios d'accès distant, soit :
    -   passez `baseUrl` par appel d'outil, soit
    -   utilisez `gateway.bind=custom` et `gateway.customBindHost`
-   Activez `security.allowRemoteViewer` uniquement lorsque vous avez l'intention d'un accès externe au visualiseur.

La ligne de lignes non modifiées n'a pas de bouton de développement :

-   Cela peut se produire pour une entrée de patch lorsque le patch ne transporte pas de contexte extensible.
-   C'est attendu et n'indique pas un échec du visualiseur.

Artefact introuvable :

-   L'artefact a expiré en raison du TTL.
-   Le token ou le chemin a changé.
-   Le nettoyage a supprimé les données obsolètes.

## Guide opérationnel

-   Préférez `mode: "view"` pour les revues interactives locales sur le canevas.
-   Préférez `mode: "file"` pour les canaux de chat sortants qui nécessitent une pièce jointe.
-   Gardez `allowRemoteViewer` désactivé sauf si votre déploiement nécessite des URLs de visualiseur distantes.
-   Définissez un `ttlSeconds` court explicite pour les diffs sensibles.
-   Évitez d'envoyer des secrets dans l'entrée diff lorsque ce n'est pas nécessaire.
-   Si votre canal compresse agressivement les images (par exemple Telegram ou WhatsApp), préférez la sortie PDF (`fileFormat: "pdf"`).

Moteur de rendu de diff :

## Documentation connexe

-   [Vue d'ensemble des outils](../tools.md)
-   [Plugins](./plugin.md)
-   [Navigateur](./browser.md)

[Perplexity Sonar](../perplexity.md)[Outil PDF](./pdf.md)