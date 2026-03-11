

  Navigateur

  
# Connexion au navigateur

## Connexion manuelle (recommandée)

Lorsqu'un site nécessite une connexion, **connectez-vous manuellement** dans le profil du navigateur **hôte** (le navigateur openclaw). **Ne donnez pas** vos identifiants au modèle. Les connexions automatisées déclenchent souvent des défenses anti‑bots et peuvent bloquer le compte. Retour à la documentation principale du navigateur : [Navigateur](./browser.md).

## Quel profil Chrome est utilisé ?

OpenClaw contrôle un **profil Chrome dédié** (nommé `openclaw`, interface orange). Celui-ci est séparé de votre profil de navigation quotidien. Deux façons simples d'y accéder :

1.  **Demandez à l'agent d'ouvrir le navigateur** puis connectez-vous vous-même.
2.  **Ouvrez-le via CLI** :

```bash
openclaw browser start
openclaw browser open https://x.com
```

Si vous avez plusieurs profils, utilisez `--browser-profile ` (la valeur par défaut est `openclaw`).

## X/Twitter : flux recommandé

-   **Lire/rechercher/fils de discussion :** utilisez le navigateur **hôte** (connexion manuelle).
-   **Publier des mises à jour :** utilisez le navigateur **hôte** (connexion manuelle).

## Sandboxing + accès au navigateur hôte

Les sessions de navigateur en sandbox sont **plus susceptibles** de déclencher une détection de bot. Pour X/Twitter (et d'autres sites stricts), préférez le navigateur **hôte**. Si l'agent est en sandbox, l'outil navigateur utilise par défaut le sandbox. Pour autoriser le contrôle de l'hôte :

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        browser: {
          allowHostControl: true,
        },
      },
    },
  },
}
```

Puis ciblez le navigateur hôte :

```bash
openclaw browser open https://x.com --browser-profile openclaw --target host
```

Ou désactivez le sandboxing pour l'agent qui publie les mises à jour.

[Navigateur (géré par OpenClaw)](./browser.md)[Extension Chrome](./chrome-extension.md)

---