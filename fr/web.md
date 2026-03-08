title: "Interfaces Web OpenClaw Contrôle UI Webhooks et Sécurité"
description: "Apprenez à configurer l'interface web de la passerelle OpenClaw, l'UI de contrôle, les webhooks et l'accès Tailscale. Configurez la sécurité, l'authentification et les modes de déploiement."
keywords: ["interface web openclaw", "ui de contrôle passerelle", "intégration tailscale", "configuration webhooks", "sécurité passerelle", "liaison loopback", "déploiement funnel", "jeton d'authentification"]
---

  Interfaces Web

  
# Web

La Passerelle sert une petite **UI de contrôle navigateur** (Vite + Lit) depuis le même port que le WebSocket de la Passerelle :

-   par défaut : `http://<hôte>:18789/`
-   préfixe optionnel : définir `gateway.controlUi.basePath` (ex. `/openclaw`)

Les fonctionnalités sont détaillées dans [l'UI de contrôle](./web/control-ui.md). Cette page se concentre sur les modes de liaison, la sécurité et les surfaces exposées au web.

## Webhooks

Lorsque `hooks.enabled=true`, la Passerelle expose également un petit point de terminaison webhook sur le même serveur HTTP. Voir [Configuration de la Passerelle](./gateway/configuration.md) → `hooks` pour l'authentification et les charges utiles.

## Configuration (activé par défaut)

L'UI de contrôle est **activée par défaut** lorsque les ressources sont présentes (`dist/control-ui`). Vous pouvez la contrôler via la configuration :

```json
{
  gateway: {
    controlUi: { enabled: true, basePath: "/openclaw" }, // basePath optionnel
  },
}
```

## Accès Tailscale

### Serve intégré (recommandé)

Gardez la Passerelle en loopback et laissez Tailscale Serve la proxifier :

```json
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" },
  },
}
```

Puis démarrez la passerelle :

```bash
openclaw gateway
```

Ouvrez :

-   `https:///` (ou votre `gateway.controlUi.basePath` configuré)

### Liaison Tailnet + jeton

```json
{
  gateway: {
    bind: "tailnet",
    controlUi: { enabled: true },
    auth: { mode: "token", token: "your-token" },
  },
}
```

Puis démarrez la passerelle (jeton requis pour les liaisons non-loopback) :

```bash
openclaw gateway
```

Ouvrez :

-   `http://<tailscale-ip>:18789/` (ou votre `gateway.controlUi.basePath` configuré)

### Internet public (Funnel)

```json
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password" }, // ou OPENCLAW_GATEWAY_PASSWORD
  },
}
```

## Notes de sécurité

-   L'authentification de la passerelle est requise par défaut (jeton/mot de passe ou en-têtes d'identité Tailscale).
-   Les liaisons non-loopback **nécessitent** toujours un jeton/mot de passe partagé (`gateway.auth` ou variable d'environnement).
-   L'assistant génère un jeton de passerelle par défaut (même en loopback).
-   L'UI envoie `connect.params.auth.token` ou `connect.params.auth.password`.
-   Pour les déploiements d'UI de contrôle non-loopback, définissez explicitement `gateway.controlUi.allowedOrigins` (origines complètes). Sans cela, le démarrage de la passerelle est refusé par défaut.
-   `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback=true` active le mode de secours basé sur l'en-tête Host, mais constitue une dégradation de sécurité dangereuse.
-   Avec Serve, les en-têtes d'identité Tailscale peuvent satisfaire l'authentification de l'UI de contrôle/WebSocket lorsque `gateway.auth.allowTailscale` est `true` (aucun jeton/mot de passe requis). Les points de terminaison de l'API HTTP nécessitent toujours un jeton/mot de passe. Définissez `gateway.auth.allowTailscale: false` pour exiger des identifiants explicites. Voir [Tailscale](./gateway/tailscale.md) et [Sécurité](./gateway/security.md). Ce flux sans jeton suppose que l'hôte de la passerelle est de confiance.
-   `gateway.tailscale.mode: "funnel"` nécessite `gateway.auth.mode: "password"` (mot de passe partagé).

## Construction de l'UI

La Passerelle sert les fichiers statiques depuis `dist/control-ui`. Construisez-les avec :

```bash
pnpm ui:build # installe automatiquement les dépendances de l'UI au premier lancement
```

[MODÈLE DE MENACE CONTRIBUTION](./security/CONTRIBUTING-THREAT-MODEL.md)[UI de contrôle](./web/control-ui.md)

---