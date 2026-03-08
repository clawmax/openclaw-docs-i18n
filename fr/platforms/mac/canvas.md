title: "Documentation et guide d'utilisation de l'application OpenClaw Canvas pour macOS"
description: "Apprenez à utiliser le panneau Canvas d'OpenClaw sur macOS pour les espaces de travail HTML/CSS/JS et A2UI. Ce guide couvre le stockage des fichiers, l'API agent, les commandes CLI et la sécurité."
keywords: ["openclaw canvas", "panneau canvas macos", "intégration a2ui", "api agent", "wkwebview", "espace de travail canvas", "documentation openclaw", "commandes cli canvas"]
---

  Application compagnon macOS

  
# Canvas

L'application macOS intègre un **panneau Canvas** contrôlé par un agent en utilisant `WKWebView`. C'est un espace de travail visuel léger pour HTML/CSS/JS, A2UI et de petites surfaces d'interface utilisateur interactives.

## Où se trouve Canvas

L'état de Canvas est stocké sous Application Support :

-   `~/Library/Application Support/OpenClaw/canvas//...`

Le panneau Canvas sert ces fichiers via un **schéma d'URL personnalisé** :

-   `openclaw-canvas:///`

Exemples :

-   `openclaw-canvas://main/` → `/main/index.html`
-   `openclaw-canvas://main/assets/app.css` → `/main/assets/app.css`
-   `openclaw-canvas://main/widgets/todo/` → `/main/widgets/todo/index.html`

Si aucun `index.html` n'existe à la racine, l'application affiche une **page d'échafaudage intégrée**.

## Comportement du panneau

-   Panneau sans bordure, redimensionnable, ancré près de la barre de menus (ou du curseur de la souris).
-   Mémorise la taille/position par session.
-   Recharge automatiquement lorsque les fichiers locaux de canvas changent.
-   Un seul panneau Canvas est visible à la fois (la session est changée si nécessaire).

Canvas peut être désactivé depuis Paramètres → **Autoriser Canvas**. Lorsqu'il est désactivé, les commandes du nœud canvas renvoient `CANVAS_DISABLED`.

## Surface d'API de l'agent

Canvas est exposé via le **WebSocket de la passerelle**, permettant à l'agent de :

-   afficher/masquer le panneau
-   naviguer vers un chemin ou une URL
-   évaluer du JavaScript
-   capturer une image instantanée

Exemples CLI :

```bash
openclaw nodes canvas present --node <id>
openclaw nodes canvas navigate --node <id> --url "/"
openclaw nodes canvas eval --node <id> --js "document.title"
openclaw nodes canvas snapshot --node <id>
```

Notes :

-   `canvas.navigate` accepte les **chemins locaux de canvas**, les URL `http(s)` et les URL `file://`.
-   Si vous passez `"/"`, le Canvas affiche l'échafaudage local ou `index.html`.

## A2UI dans Canvas

A2UI est hébergé par l'hôte canvas de la passerelle et rendu à l'intérieur du panneau Canvas. Lorsque la passerelle annonce un hôte Canvas, l'application macOS navigue automatiquement vers la page hôte A2UI à la première ouverture. URL hôte A2UI par défaut :

```
http://<gateway-host>:18789/__openclaw__/a2ui/
```

### Commandes A2UI (v0.8)

Canvas accepte actuellement les messages serveur→client **A2UI v0.8** :

-   `beginRendering`
-   `surfaceUpdate`
-   `dataModelUpdate`
-   `deleteSurface`

`createSurface` (v0.9) n'est pas pris en charge. Exemple CLI :

```bash
cat > /tmp/a2ui-v0.8.jsonl <<'EOFA2'
{"surfaceUpdate":{"surfaceId":"main","components":[{"id":"root","component":{"Column":{"children":{"explicitList":["title","content"]}}}},{"id":"title","component":{"Text":{"text":{"literalString":"Canvas (A2UI v0.8)"},"usageHint":"h1"}}},{"id":"content","component":{"Text":{"text":{"literalString":"If you can read this, A2UI push works."},"usageHint":"body"}}}]}}
{"beginRendering":{"surfaceId":"main","root":"root"}}
EOFA2

openclaw nodes canvas a2ui push --jsonl /tmp/a2ui-v0.8.jsonl --node <id>
```

Test rapide :

```bash
openclaw nodes canvas a2ui push --node <id> --text "Hello from A2UI"
```

## Déclencher des exécutions d'agent depuis Canvas

Canvas peut déclencher de nouvelles exécutions d'agent via des liens profonds :

-   `openclaw://agent?...`

Exemple (en JS) :

```bash
window.location.href = "openclaw://agent?message=Review%20this%20design";
```

L'application demande une confirmation sauf si une clé valide est fournie.

## Notes de sécurité

-   Le schéma Canvas bloque la traversée de répertoires ; les fichiers doivent résider sous la racine de la session.
-   Le contenu local de Canvas utilise un schéma personnalisé (aucun serveur local requis).
-   Les URL externes `http(s)` ne sont autorisées que lorsqu'elles sont explicitement naviguées.

[WebChat](./webchat.md)[Cycle de vie de la passerelle](./child-process.md)

---