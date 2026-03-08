title: "Liste des plugins communautaires OpenClaw et guide de soumission"
description: "Découvrez des plugins communautaires de haute qualité pour OpenClaw et apprenez à soumettre les vôtres. Trouvez les commandes d'installation et les liens vers les dépôts."
keywords: ["plugins openclaw", "plugins communautaires", "soumission de plugin", "npm openclaw", "dépôt github", "installation de plugin", "extensions", "intégration wechat"]
---

  Extensions

  
# Plugins communautaires

Cette page recense les **plugins maintenus par la communauté** de haute qualité pour OpenClaw. Nous acceptons les PR qui ajoutent des plugins communautaires ici lorsqu'ils atteignent le niveau de qualité requis.

## Conditions requises pour figurer dans la liste

-   Le package du plugin est publié sur npmjs (installable via `openclaw plugins install <npm-spec>`).
-   Le code source est hébergé sur GitHub (dépôt public).
-   Le dépôt inclut une documentation d'installation/utilisation et un suivi des problèmes.
-   Le plugin présente un signal clair de maintenance (mainteneur actif, mises à jour récentes, ou traitement réactif des problèmes).

## Comment soumettre

Ouvrez une PR qui ajoute votre plugin à cette page avec :

-   Le nom du plugin
-   Le nom du package npm
-   L'URL du dépôt GitHub
-   Une description en une ligne
-   La commande d'installation

## Critères d'examen

Nous privilégions les plugins utiles, documentés et sûrs à utiliser. Les enveloppes basiques, les projets à la propriété incertaine ou les packages non maintenus peuvent être refusés.

## Format pour les candidats

Utilisez ce format lors de l'ajout d'entrées :

-   **Nom du Plugin** — courte description npm: `@scope/package` repo: `https://github.com/org/repo` install: `openclaw plugins install @scope/package`

## Plugins listés

-   **WeChat** — Connecte OpenClaw aux comptes personnels WeChat via WeChatPadPro (protocole iPad). Prend en charge l'échange de texte, d'images et de fichiers avec des conversations déclenchées par mot-clé. npm: `@icesword760/openclaw-wechat` repo: `https://github.com/icesword0760/openclaw-wechat` install: `openclaw plugins install @icesword760/openclaw-wechat`

[Plugins](../tools/plugin.md)[Plugin Appel Vocal](./voice-call.md)

---