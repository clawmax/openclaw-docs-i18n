title: "Guide de configuration et référence des paramètres des compétences OpenClaw"
description: "Apprenez à configurer les compétences d'IA OpenClaw. Définissez des listes autorisées, chargez des répertoires de compétences personnalisés, gérez les paramètres par compétence et configurez les variables d'environnement."
keywords: ["compétences openclaw", "configuration des compétences", "compétences de l'agent", "paramètres des compétences", "répertoires de compétences", "variables d'environnement des compétences", "surcharges des compétences", "compétences en bac à sable"]
---

  Compétences

  
# Configuration des compétences

Toute la configuration liée aux compétences se trouve sous `skills` dans `~/.openclaw/openclaw.json`.

```json
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills", "~/Projects/oss/some-skill-pack/skills"],
      watch: true,
      watchDebounceMs: 250,
    },
    install: {
      preferBrew: true,
      nodeManager: "npm", // npm | pnpm | yarn | bun (Gateway runtime still Node; bun not recommended)
    },
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: { source: "env", provider: "default", id: "GEMINI_API_KEY" }, // or plaintext string
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

## Champs

-   `allowBundled` : liste autorisée optionnelle pour les compétences **intégrées** uniquement. Lorsqu'elle est définie, seules les compétences intégrées de la liste sont éligibles (les compétences gérées/workspace ne sont pas affectées).
-   `load.extraDirs` : répertoires de compétences supplémentaires à analyser (priorité la plus basse).
-   `load.watch` : surveiller les dossiers de compétences et rafraîchir l'instantané des compétences (par défaut : true).
-   `load.watchDebounceMs` : délai de temporisation pour les événements du surveillant de compétences en millisecondes (par défaut : 250).
-   `install.preferBrew` : préférer les installateurs brew lorsqu'ils sont disponibles (par défaut : true).
-   `install.nodeManager` : préférence d'installateur Node (`npm` | `pnpm` | `yarn` | `bun`, par défaut : npm). Cela n'affecte que **l'installation des compétences** ; l'environnement d'exécution Gateway doit rester Node (Bun n'est pas recommandé pour WhatsApp/Telegram).
-   `entries.` : surcharges par compétence.

Champs par compétence :

-   `enabled` : définir à `false` pour désactiver une compétence même si elle est intégrée/installée.
-   `env` : variables d'environnement injectées pour l'exécution de l'agent (uniquement si elles ne sont pas déjà définies).
-   `apiKey` : commodité optionnelle pour les compétences qui déclarent une variable d'environnement principale. Prend en charge une chaîne en clair ou un objet SecretRef (`{ source, provider, id }`).

## Notes

-   Les clés sous `entries` correspondent par défaut au nom de la compétence. Si une compétence définit `metadata.openclaw.skillKey`, utilisez cette clé à la place.
-   Les modifications apportées aux compétences sont prises en compte au prochain tour de l'agent lorsque le surveillant est activé.

### Compétences en bac à sable + variables d'environnement

Lorsqu'une session est **en bac à sable**, les processus des compétences s'exécutent à l'intérieur de Docker. Le bac à sable **n'hérite pas** du `process.env` de l'hôte. Utilisez l'une des options suivantes :

-   `agents.defaults.sandbox.docker.env` (ou par agent `agents.list[].sandbox.docker.env`)
-   intégrez les variables d'environnement dans votre image de bac à sable personnalisée

Les `env` globaux et `skills.entries..env/apiKey` s'appliquent uniquement aux exécutions sur **l'hôte**.

[Compétences](./skills.md)[ClawHub](./clawhub.md)

---