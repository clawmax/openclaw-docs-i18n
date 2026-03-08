

  Détails internes des concepts

  
# Formatage Markdown

OpenClaw formate le Markdown sortant en le convertissant d'abord en une représentation intermédiaire (IR) partagée avant de produire un rendu spécifique à chaque canal. L'IR conserve le texte source intact tout en portant des étendues de style/liens, permettant ainsi au découpage et au rendu de rester cohérents d'un canal à l'autre.

## Objectifs

-   **Cohérence :** une seule étape d'analyse, plusieurs moteurs de rendu.
-   **Découpage sécurisé :** diviser le texte avant le rendu pour que le formatage en ligne ne soit jamais coupé entre les segments.
-   **Adaptation au canal :** mapper la même IR vers le mrkdwn de Slack, le HTML de Telegram et les plages de style de Signal sans ré-analyser le Markdown.

## Pipeline

1.  **Analyser le Markdown -> IR**
    -   L'IR est du texte brut plus des étendues de style (gras/italique/barré/code/spoiler) et des étendues de liens.
    -   Les décalages sont en unités de code UTF-16 pour que les plages de style de Signal s'alignent avec son API.
    -   Les tableaux sont analysés uniquement lorsqu'un canal opte pour la conversion de tableaux.
2.  **Découper l'IR (formatage en premier)**
    -   Le découpage s'effectue sur le texte de l'IR avant le rendu.
    -   Le formatage en ligne n'est pas divisé entre les segments ; les étendues sont découpées par segment.
3.  **Rendu par canal**
    -   **Slack :** jetons mrkdwn (gras/italique/barré/code), liens sous la forme `<url|label>`.
    -   **Telegram :** balises HTML (``, ``, ``, ``, ``, ``).
    -   **Signal :** texte brut + plages `text-style` ; les liens deviennent `label (url)` lorsque le libellé diffère.

## Exemple d'IR

Markdown d'entrée :

```bash
Hello **world** — see [docs](https://docs.openclaw.ai).
```

IR (schématique) :

```json
{
  "text": "Hello world — see docs.",
  "styles": [{ "start": 6, "end": 11, "style": "bold" }],
  "links": [{ "start": 19, "end": 23, "href": "https://docs.openclaw.ai" }]
}
```

## Où est-ce utilisé

-   Les adaptateurs sortants de Slack, Telegram et Signal effectuent le rendu à partir de l'IR.
-   Les autres canaux (WhatsApp, iMessage, MS Teams, Discord) utilisent toujours du texte brut ou leurs propres règles de formatage, avec la conversion des tableaux Markdown appliquée avant le découpage lorsqu'elle est activée.

## Gestion des tableaux

Les tableaux Markdown ne sont pas pris en charge de manière cohérente par les clients de messagerie. Utilisez `markdown.tables` pour contrôler la conversion par canal (et par compte).

-   `code` : rendre les tableaux sous forme de blocs de code (par défaut pour la plupart des canaux).
-   `bullets` : convertir chaque ligne en points à puces (par défaut pour Signal + WhatsApp).
-   `off` : désactiver l'analyse et la conversion des tableaux ; le texte brut du tableau passe tel quel.

Clés de configuration :

```yaml
channels:
  discord:
    markdown:
      tables: code
    accounts:
      work:
        markdown:
          tables: off
```

## Règles de découpage

-   Les limites de segment proviennent des adaptateurs/configuration des canaux et sont appliquées au texte de l'IR.
-   Les blocs de code délimités sont préservés en un seul bloc avec un saut de ligne final pour que les canaux les rendent correctement.
-   Les préfixes de liste et de citation font partie du texte de l'IR, donc le découpage ne les divise pas en leur milieu.
-   Les styles en ligne (gras/italique/barré/code en ligne/spoiler) ne sont jamais divisés entre les segments ; le moteur de rendu rouvre les styles à l'intérieur de chaque segment.

Pour plus d'informations sur le comportement du découpage entre les canaux, consultez [Streaming + découpage](./streaming.md).

## Politique des liens

-   **Slack :** `[label](url)` -> `<url|label>` ; les URL brutes restent brutes. L'autolink est désactivé lors de l'analyse pour éviter les doubles liens.
-   **Telegram :** `[label](url)` -> `label
` (mode d'analyse HTML).
-   **Signal :** `[label](url)` -> `label (url)` sauf si le libellé correspond à l'URL.

## Spoilers

Les marqueurs de spoiler (`||spoiler||`) sont analysés uniquement pour Signal, où ils correspondent aux plages de style SPOILER. Les autres canaux les traitent comme du texte brut.

## Comment ajouter ou mettre à jour un formateur de canal

1.  **Analyser une fois :** utilisez l'utilitaire partagé `markdownToIR(...)` avec des options appropriées au canal (autolink, style de titre, préfixe de citation).
2.  **Rendu :** implémentez un moteur de rendu avec `renderMarkdownWithMarkers(...)` et une carte de marqueurs de style (ou des plages de style Signal).
3.  **Découpage :** appelez `chunkMarkdownIR(...)` avant le rendu ; effectuez le rendu de chaque segment.
4.  **Câblage de l'adaptateur :** mettez à jour l'adaptateur sortant du canal pour utiliser le nouveau découpeur et moteur de rendu.
5.  **Test :** ajoutez ou mettez à jour les tests de format et un test de livraison sortante si le canal utilise le découpage.

## Pièges courants

-   Les jetons entre chevrons de Slack (`<@U123>`, `<#C123>`, `<https://...>`) doivent être préservés ; échappez le HTML brut de manière sécurisée.
-   Le HTML de Telegram nécessite d'échapper le texte en dehors des balises pour éviter un balisage cassé.
-   Les plages de style de Signal dépendent des décalages UTF-16 ; n'utilisez pas de décalages en points de code.
-   Préservez les sauts de ligne finaux pour les blocs de code délimités afin que les marqueurs de fermeture se trouvent sur leur propre ligne.

[TypeBox](./typebox.md)[Indicateurs de saisie](./typing-indicators.md)