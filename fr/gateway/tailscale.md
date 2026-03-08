

  Accès distant

  
# Tailscale

OpenClaw peut configurer automatiquement Tailscale **Serve** (tailnet) ou **Funnel** (public) pour le tableau de bord de la passerelle et le port WebSocket. Cela maintient la passerelle liée à l'adresse de bouclage (loopback) tandis que Tailscale fournit le HTTPS, le routage et (pour Serve) les en-têtes d'identité.

## Modes

-   `serve` : Serve uniquement sur le tailnet via `tailscale serve`. La passerelle reste sur `127.0.0.1`.
-   `funnel` : HTTPS public via `tailscale funnel`. OpenClaw nécessite un mot de passe partagé.
-   `off` : Par défaut (pas d'automatisation Tailscale).

## Authentification

Définissez `gateway.auth.mode` pour contrôler la poignée de main :

-   `token` (par défaut lorsque `OPENCLAW_GATEWAY_TOKEN` est défini)
-   `password` (secret partagé via `OPENCLAW_GATEWAY_PASSWORD` ou la configuration)

Lorsque `tailscale.mode = "serve"` et que `gateway.auth.allowTailscale` est `true`, l'authentification de l'interface de contrôle/WebSocket peut utiliser les en-têtes d'identité Tailscale (`tailscale-user-login`) sans fournir de token/mot de passe. OpenClaw vérifie l'identité en résolvant l'adresse `x-forwarded-for` via le démon Tailscale local (`tailscale whois`) et en la faisant correspondre à l'en-tête avant de l'accepter. OpenClaw ne traite une requête comme provenant de Serve que lorsqu'elle arrive depuis l'adresse de bouclage avec les en-têtes Tailscale `x-forwarded-for`, `x-forwarded-proto` et `x-forwarded-host`. Les points de terminaison de l'API HTTP (par exemple `/v1/*`, `/tools/invoke` et `/api/channels/*`) nécessitent toujours une authentification par token/mot de passe. Ce flux sans token suppose que l'hôte de la passerelle est de confiance. Si du code local non fiable peut s'exécuter sur le même hôte, désactivez `gateway.auth.allowTailscale` et exigez plutôt une authentification par token/mot de passe. Pour exiger des identifiants explicites, définissez `gateway.auth.allowTailscale: false` ou forcez `gateway.auth.mode: "password"`.

## Exemples de configuration

### Uniquement sur le tailnet (Serve)

```json
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" },
  },
}
```

Ouvrir : `https:///` (ou votre `gateway.controlUi.basePath` configuré)

### Uniquement sur le tailnet (liaison à l'IP Tailnet)

Utilisez ceci lorsque vous voulez que la passerelle écoute directement sur l'IP Tailnet (sans Serve/Funnel).

```json
{
  gateway: {
    bind: "tailnet",
    auth: { mode: "token", token: "your-token" },
  },
}
```

Se connecter depuis un autre appareil du tailnet :

-   Interface de contrôle : `http://<tailscale-ip>:18789/`
-   WebSocket : `ws://<tailscale-ip>:18789`

Note : l'adresse de bouclage (`http://127.0.0.1:18789`) ne fonctionnera **pas** dans ce mode.

### Internet public (Funnel + mot de passe partagé)

```json
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password", password: "replace-me" },
  },
}
```

Préférez `OPENCLAW_GATEWAY_PASSWORD` plutôt que de commettre un mot de passe sur le disque.

## Exemples en ligne de commande

```bash
openclaw gateway --tailscale serve
openclaw gateway --tailscale funnel --auth password
```

## Notes

-   Tailscale Serve/Funnel nécessite que la CLI `tailscale` soit installée et connectée.
-   `tailscale.mode: "funnel"` refuse de démarrer à moins que le mode d'authentification soit `password` pour éviter une exposition publique.
-   Définissez `gateway.tailscale.resetOnExit` si vous voulez qu'OpenClaw annule la configuration `tailscale serve` ou `tailscale funnel` à l'arrêt.
-   `gateway.bind: "tailnet"` est une liaison directe au tailnet (pas de HTTPS, pas de Serve/Funnel).
-   `gateway.bind: "auto"` préfère l'adresse de bouclage ; utilisez `tailnet` si vous voulez uniquement le tailnet.
-   Serve/Funnel n'exposent que **l'interface de contrôle de la passerelle + WS**. Les nœuds se connectent via le même point de terminaison WS de la passerelle, donc Serve peut fonctionner pour l'accès aux nœuds.

## Contrôle par navigateur (passerelle distante + navigateur local)

Si vous exécutez la passerelle sur une machine mais que vous voulez piloter un navigateur sur une autre machine, exécutez un **hôte de nœud** sur la machine du navigateur et gardez les deux sur le même tailnet. La passerelle proxyfiera les actions du navigateur vers le nœud ; aucun serveur de contrôle séparé ou URL Serve n'est nécessaire. Évitez Funnel pour le contrôle par navigateur ; traitez l'appairage de nœud comme un accès opérateur.

## Prérequis et limites de Tailscale

-   Serve nécessite le HTTPS activé pour votre tailnet ; la CLI vous le demande s'il est manquant.
-   Serve injecte les en-têtes d'identité Tailscale ; Funnel ne le fait pas.
-   Funnel nécessite Tailscale v1.38.3+, MagicDNS, HTTPS activé et un attribut de nœud funnel.
-   Funnel ne prend en charge que les ports `443`, `8443` et `10000` en TLS.
-   Funnel sur macOS nécessite la variante open-source de l'application Tailscale.

## Pour en savoir plus

-   Aperçu de Tailscale Serve : [https://tailscale.com/kb/1312/serve](https://tailscale.com/kb/1312/serve)
-   Commande `tailscale serve` : [https://tailscale.com/kb/1242/tailscale-serve](https://tailscale.com/kb/1242/tailscale-serve)
-   Aperçu de Tailscale Funnel : [https://tailscale.com/kb/1223/tailscale-funnel](https://tailscale.com/kb/1223/tailscale-funnel)
-   Commande `tailscale funnel` : [https://tailscale.com/kb/1311/tailscale-funnel](https://tailscale.com/kb/1311/tailscale-funnel)

[Configuration de la passerelle distante](./remote-gateway-readme.md)[Vérification formelle (modèles de sécurité)](../security/formal-verification.md)