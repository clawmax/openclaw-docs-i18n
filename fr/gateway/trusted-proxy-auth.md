

  Configuration et opérations

  
# Authentification par proxy de confiance

> ⚠️ **Fonctionnalité sensible pour la sécurité.** Ce mode délègue entièrement l'authentification à votre proxy inverse. Une mauvaise configuration peut exposer votre Gateway à un accès non autorisé. Lisez cette page attentivement avant de l'activer.

## Quand l'utiliser

Utilisez le mode d'authentification `trusted-proxy` lorsque :

-   Vous exécutez OpenClaw derrière un **proxy avec identité** (Pomerium, Caddy + OAuth, nginx + oauth2-proxy, Traefik + forward auth)
-   Votre proxy gère toute l'authentification et transmet l'identité de l'utilisateur via des en-têtes
-   Vous êtes dans un environnement Kubernetes ou conteneurisé où le proxy est le seul chemin d'accès à la Gateway
-   Vous rencontrez des erreurs WebSocket `1008 unauthorized` car les navigateurs ne peuvent pas transmettre de jetons dans les charges utiles WS

## Quand NE PAS l'utiliser

-   Si votre proxy n'authentifie pas les utilisateurs (juste un terminateur TLS ou un équilibreur de charge)
-   S'il existe un chemin d'accès à la Gateway qui contourne le proxy (trous de pare-feu, accès réseau interne)
-   Si vous n'êtes pas sûr que votre proxy supprime/remplace correctement les en-têtes transmis
-   Si vous avez seulement besoin d'un accès personnel mono-utilisateur (envisagez Tailscale Serve + loopback pour une configuration plus simple)

## Comment cela fonctionne

1.  Votre proxy inverse authentifie les utilisateurs (OAuth, OIDC, SAML, etc.)
2.  Le proxy ajoute un en-tête avec l'identité de l'utilisateur authentifié (par exemple, `x-forwarded-user: nick@example.com`)
3.  OpenClaw vérifie que la requête provient d'une **IP de proxy de confiance** (configurée dans `gateway.trustedProxies`)
4.  OpenClaw extrait l'identité de l'utilisateur de l'en-tête configuré
5.  Si tout est vérifié, la requête est autorisée

## Comportement d'appariement de l'interface de contrôle

Lorsque `gateway.auth.mode = "trusted-proxy"` est actif et que la requête passe les vérifications du proxy de confiance, les sessions WebSocket de l'interface de contrôle peuvent se connecter sans identité d'appariement d'appareil. Implications :

-   L'appariement n'est plus la porte d'entrée principale pour l'accès à l'interface de contrôle dans ce mode.
-   La politique d'authentification de votre proxy inverse et `allowUsers` deviennent le contrôle d'accès effectif.
-   Gardez l'ingress de la gateway verrouillée uniquement aux IPs des proxies de confiance (`gateway.trustedProxies` + pare-feu).

## Configuration

```json
{
  gateway: {
    // Utilisez loopback pour les configurations de proxy sur le même hôte ; utilisez lan/custom pour les hôtes proxy distants
    bind: "loopback",

    // CRITIQUE : Ajoutez uniquement l'(les) IP(s) de votre proxy ici
    trustedProxies: ["10.0.0.1", "172.17.0.1"],

    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        // En-tête contenant l'identité de l'utilisateur authentifié (requis)
        userHeader: "x-forwarded-user",

        // Optionnel : en-têtes qui DOIVENT être présents (vérification du proxy)
        requiredHeaders: ["x-forwarded-proto", "x-forwarded-host"],

        // Optionnel : restreindre à des utilisateurs spécifiques (vide = autoriser tous)
        allowUsers: ["nick@example.com", "admin@company.org"],
      },
    },
  },
}
```

Si `gateway.bind` est `loopback`, incluez une adresse proxy loopback dans `gateway.trustedProxies` (`127.0.0.1`, `::1`, ou un CIDR loopback équivalent).

### Référence de configuration

| Champ | Requis | Description |
| --- | --- | --- |
| `gateway.trustedProxies` | Oui | Tableau des adresses IP des proxies à considérer comme de confiance. Les requêtes provenant d'autres IPs sont rejetées. |
| `gateway.auth.mode` | Oui | Doit être `"trusted-proxy"` |
| `gateway.auth.trustedProxy.userHeader` | Oui | Nom de l'en-tête contenant l'identité de l'utilisateur authentifié |
| `gateway.auth.trustedProxy.requiredHeaders` | Non | En-têtes supplémentaires qui doivent être présents pour que la requête soit considérée comme de confiance |
| `gateway.auth.trustedProxy.allowUsers` | Non | Liste blanche des identités utilisateur. Vide signifie autoriser tous les utilisateurs authentifiés. |

## Terminaison TLS et HSTS

Utilisez un point de terminaison TLS et appliquez HSTS à cet endroit.

### Modèle recommandé : terminaison TLS au proxy

Lorsque votre proxy inverse gère le HTTPS pour `https://control.example.com`, définissez `Strict-Transport-Security` au niveau du proxy pour ce domaine.

-   Convient bien aux déploiements exposés à Internet.
-   Garde le certificat et la politique de durcissement HTTP en un seul endroit.
-   OpenClaw peut rester sur HTTP loopback derrière le proxy.

Exemple de valeur d'en-tête :

```yaml
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

### Terminaison TLS à la Gateway

Si OpenClaw sert lui-même le HTTPS directement (sans proxy terminant TLS), définissez :

```json
{
  gateway: {
    tls: { enabled: true },
    http: {
      securityHeaders: {
        strictTransportSecurity: "max-age=31536000; includeSubDomains",
      },
    },
  },
}
```

`strictTransportSecurity` accepte une valeur d'en-tête sous forme de chaîne, ou `false` pour désactiver explicitement.

### Guide de déploiement

-   Commencez par une durée de vie maximale courte d'abord (par exemple `max-age=300`) pendant que vous validez le trafic.
-   Augmentez vers des valeurs de longue durée (par exemple `max-age=31536000`) seulement après avoir une confiance élevée.
-   Ajoutez `includeSubDomains` uniquement si chaque sous-domaine est prêt pour le HTTPS.
-   Utilisez preload seulement si vous remplissez intentionnellement les exigences de préchargement pour l'ensemble complet de vos domaines.
-   Le développement local en loopback uniquement ne bénéficie pas de HSTS.

## Exemples de configuration de proxy

### Pomerium

Pomerium transmet l'identité dans `x-pomerium-claim-email` (ou d'autres en-têtes de claim) et un JWT dans `x-pomerium-jwt-assertion`.

```json
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // IP de Pomerium
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-pomerium-claim-email",
        requiredHeaders: ["x-pomerium-jwt-assertion"],
      },
    },
  },
}
```

Extrait de configuration Pomerium :

```yaml
routes:
  - from: https://openclaw.example.com
    to: http://openclaw-gateway:18789
    policy:
      - allow:
          or:
            - email:
                is: nick@example.com
    pass_identity_headers: true
```

### Caddy avec OAuth

Caddy avec le plugin `caddy-security` peut authentifier les utilisateurs et transmettre les en-têtes d'identité.

```json
{
  gateway: {
    bind: "lan",
    trustedProxies: ["127.0.0.1"], // IP de Caddy (si sur le même hôte)
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

Extrait de Caddyfile :

```
openclaw.example.com {
    authenticate with oauth2_provider
    authorize with policy1

    reverse_proxy openclaw:18789 {
        header_up X-Forwarded-User {http.auth.user.email}
    }
}
```

### nginx + oauth2-proxy

oauth2-proxy authentifie les utilisateurs et transmet l'identité dans `x-auth-request-email`.

```json
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // IP de nginx/oauth2-proxy
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-auth-request-email",
      },
    },
  },
}
```

Extrait de configuration nginx :

```nginx
location / {
    auth_request /oauth2/auth;
    auth_request_set $user $upstream_http_x_auth_request_email;

    proxy_pass http://openclaw:18789;
    proxy_set_header X-Auth-Request-Email $user;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

### Traefik avec Forward Auth

```json
{
  gateway: {
    bind: "lan",
    trustedProxies: ["172.17.0.1"], // IP du conteneur Traefik
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

## Liste de contrôle de sécurité

Avant d'activer l'authentification trusted-proxy, vérifiez :

-   [ ]  **Le proxy est le seul chemin** : Le port de la Gateway est protégé par un pare-feu contre tout sauf votre proxy
-   [ ]  **trustedProxies est minimal** : Seulement les IPs réelles de votre proxy, pas des sous-réseaux entiers
-   [ ]  **Le proxy supprime les en-têtes** : Votre proxy remplace (n'ajoute pas) les en-têtes `x-forwarded-*` des clients
-   [ ]  **Terminaison TLS** : Votre proxy gère le TLS ; les utilisateurs se connectent via HTTPS
-   [ ]  **allowUsers est défini** (recommandé) : Restreindre aux utilisateurs connus plutôt que d'autoriser toute personne authentifiée

## Audit de sécurité

`openclaw security audit` signalera l'authentification trusted-proxy avec une constatation de sévérité **critique**. C'est intentionnel — cela sert de rappel que vous déléguez la sécurité à votre configuration de proxy. L'audit vérifie :

-   L'absence de configuration `trustedProxies`
-   L'absence de configuration `userHeader`
-   Un `allowUsers` vide (autorise tout utilisateur authentifié)

## Dépannage

### ”trusted\_proxy\_untrusted\_source”

La requête ne provient pas d'une IP dans `gateway.trustedProxies`. Vérifiez :

-   L'IP du proxy est-elle correcte ? (Les IPs des conteneurs Docker peuvent changer)
-   Y a-t-il un équilibreur de charge devant votre proxy ?
-   Utilisez `docker inspect` ou `kubectl get pods -o wide` pour trouver les IPs réelles

### ”trusted\_proxy\_user\_missing”

L'en-tête utilisateur était vide ou manquant. Vérifiez :

-   Votre proxy est-il configuré pour transmettre les en-têtes d'identité ?
-   Le nom de l'en-tête est-il correct ? (insensible à la casse, mais l'orthographe compte)
-   L'utilisateur est-il réellement authentifié au niveau du proxy ?

### “trustedproxy\_missing\_header\*”

Un en-tête requis n'était pas présent. Vérifiez :

-   Votre configuration de proxy pour ces en-têtes spécifiques
-   Si les en-têtes sont supprimés quelque part dans la chaîne

### ”trusted\_proxy\_user\_not\_allowed”

L'utilisateur est authentifié mais n'est pas dans `allowUsers`. Soit ajoutez-le, soit supprimez la liste blanche.

### WebSocket échoue toujours

Assurez-vous que votre proxy :

-   Prend en charge les mises à niveau WebSocket (`Upgrade: websocket`, `Connection: upgrade`)
-   Transmet les en-têtes d'identité sur les requêtes de mise à niveau WebSocket (pas seulement HTTP)
-   N'a pas de chemin d'authentification séparé pour les connexions WebSocket

## Migration depuis l'authentification par jeton

Si vous passez de l'authentification par jeton à trusted-proxy :

1.  Configurez votre proxy pour authentifier les utilisateurs et transmettre les en-têtes
2.  Testez la configuration du proxy indépendamment (curl avec en-têtes)
3.  Mettez à jour la configuration OpenClaw avec l'authentification trusted-proxy
4.  Redémarrez la Gateway
5.  Testez les connexions WebSocket depuis l'interface de contrôle
6.  Exécutez `openclaw security audit` et examinez les constatations

## Liens connexes

-   [Sécurité](./security.md) — guide complet de sécurité
-   [Configuration](./configuration.md) — référence de configuration
-   [Accès distant](./remote.md) — autres modèles d'accès distant
-   [Tailscale](./tailscale.md) — alternative plus simple pour un accès limité au tailnet

[Contrat de plan d'application des secrets](./secrets-plan-contract.md)[Vérifications de santé](./health.md)