

  Outils intégrés

  
# Outil PDF

`pdf` analyse un ou plusieurs documents PDF et renvoie du texte. Comportement rapide :

-   Mode fournisseur natif pour les fournisseurs de modèles Anthropic et Google.
-   Mode de repli par extraction pour les autres fournisseurs (extrait d'abord le texte, puis les images des pages si nécessaire).
-   Prend en charge une entrée unique (`pdf`) ou multiple (`pdfs`), maximum 10 PDFs par appel.

## Disponibilité

L'outil n'est enregistré que lorsque OpenClaw peut résoudre une configuration de modèle compatible PDF pour l'agent :

1.  `agents.defaults.pdfModel`
2.  repli sur `agents.defaults.imageModel`
3.  repli sur les valeurs par défaut du fournisseur en mode "meilleur effort" basées sur l'authentification disponible

Si aucun modèle utilisable ne peut être résolu, l'outil `pdf` n'est pas exposé.

## Référence des entrées

-   `pdf` (`string`) : un chemin ou URL de PDF
-   `pdfs` (`string[]`) : plusieurs chemins ou URLs de PDF, jusqu'à 10 au total
-   `prompt` (`string`) : consigne d'analyse, par défaut `Analyze this PDF document.`
-   `pages` (`string`) : filtre de pages comme `1-5` ou `1,3,7-9`
-   `model` (`string`) : remplacement optionnel du modèle (`provider/model`)
-   `maxBytesMb` (`number`) : limite de taille par PDF en Mo

Notes sur les entrées :

-   `pdf` et `pdfs` sont fusionnés et dédupliqués avant le chargement.
-   Si aucune entrée PDF n'est fournie, l'outil génère une erreur.
-   `pages` est analysé comme des numéros de pages basés sur 1, dédupliqués, triés et limités au nombre maximum de pages configuré.
-   `maxBytesMb` est par défaut `agents.defaults.pdfMaxBytesMb` ou `10`.

## Références PDF prises en charge

-   chemin de fichier local (y compris l'expansion `~`)
-   URL `file://`
-   URL `http://` et `https://`

Notes sur les références :

-   Les autres schémas d'URI (par exemple `ftp://`) sont rejetés avec `unsupported_pdf_reference`.
-   En mode sandbox, les URLs distantes `http(s)` sont rejetées.
-   Avec la politique de fichiers limitée à l'espace de travail activée, les chemins de fichiers locaux en dehors des racines autorisées sont rejetés.

## Modes d'exécution

### Mode fournisseur natif

Le mode natif est utilisé pour les fournisseurs `anthropic` et `google`. L'outil envoie les octets bruts du PDF directement aux APIs des fournisseurs. Limites du mode natif :

-   `pages` n'est pas pris en charge. S'il est défini, l'outil renvoie une erreur.

### Mode de repli par extraction

Le mode de repli est utilisé pour les fournisseurs non natifs. Flux :

1.  Extraire le texte des pages sélectionnées (jusqu'à `agents.defaults.pdfMaxPages`, par défaut `20`).
2.  Si la longueur du texte extrait est inférieure à `200` caractères, rendre les pages sélectionnées en images PNG et les inclure.
3.  Envoyer le contenu extrait plus la consigne au modèle sélectionné.

Détails du repli :

-   L'extraction d'image de page utilise un budget de pixels de `4 000 000`.
-   Si le modèle cible ne prend pas en charge l'entrée d'image et qu'il n'y a pas de texte extractible, l'outil génère une erreur.
-   Le repli par extraction nécessite `pdfjs-dist` (et `@napi-rs/canvas` pour le rendu d'image).

## Configuration

```json
{
  agents: {
    defaults: {
      pdfModel: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["openai/gpt-5-mini"],
      },
      pdfMaxBytesMb: 10,
      pdfMaxPages: 20,
    },
  },
}
```

Voir la [Référence de configuration](../gateway/configuration-reference.md) pour les détails complets des champs.

## Détails de la sortie

L'outil renvoie du texte dans `content[0].text` et des métadonnées structurées dans `details`. Champs `details` courants :

-   `model` : référence du modèle résolu (`provider/model`)
-   `native` : `true` pour le mode fournisseur natif, `false` pour le repli
-   `attempts` : tentatives de repli qui ont échoué avant le succès

Champs de chemin :

-   entrée PDF unique : `details.pdf`
-   entrées PDF multiples : `details.pdfs[]` avec des entrées `pdf`
-   métadonnées de réécriture de chemin sandbox (le cas échéant) : `rewrittenFrom`

## Comportement en cas d'erreur

-   Entrée PDF manquante : lance `pdf required: provide a path or URL to a PDF document`
-   Trop de PDFs : renvoie une erreur structurée dans `details.error = "too_many_pdfs"`
-   Schéma de référence non pris en charge : renvoie `details.error = "unsupported_pdf_reference"`
-   Mode natif avec `pages` : lance l'erreur claire `pages is not supported with native PDF providers`

## Exemples

PDF unique :

```json
{
  "pdf": "/tmp/report.pdf",
  "prompt": "Summarize this report in 5 bullets"
}
```

PDFs multiples :

```json
{
  "pdfs": ["/tmp/q1.pdf", "/tmp/q2.pdf"],
  "prompt": "Compare risks and timeline changes across both documents"
}
```

Modèle de repli avec filtre de pages :

```json
{
  "pdf": "https://example.com/report.pdf",
  "pages": "1-3,7",
  "model": "openai/gpt-5-mini",
  "prompt": "Extract only customer-impacting incidents"
}
```

[Diffs](./diffs.md)[Mode Élevé](./elevated.md)