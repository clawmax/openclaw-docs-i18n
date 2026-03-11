

  Configuration

  
# Basculement de modèle

OpenClaw gère les pannes en deux étapes :

1.  **Rotation du profil d'authentification** au sein du fournisseur actuel.
2.  **Bascule de modèle** vers le modèle suivant dans `agents.defaults.model.fallbacks`.

Ce document explique les règles d'exécution et les données qui les sous-tendent.

## Stockage des authentifications (clés + OAuth)

OpenClaw utilise des **profils d'authentification** pour les clés API et les jetons OAuth.

-   Les secrets se trouvent dans `~/.openclaw/agents//agent/auth-profiles.json` (ancien : `~/.openclaw/agent/auth-profiles.json`).
-   La configuration `auth.profiles` / `auth.order` est **uniquement des métadonnées et du routage** (pas de secrets).
-   Ancien fichier OAuth import uniquement : `~/.openclaw/credentials/oauth.json` (importé dans `auth-profiles.json` lors de la première utilisation).

Plus de détails : [/concepts/oauth](./oauth.md) Types d'identifiants :

-   `type: "api_key"` → `{ provider, key }`
-   `type: "oauth"` → `{ provider, access, refresh, expires, email? }` (+ `projectId`/`enterpriseUrl` pour certains fournisseurs)

## Identifiants de profil

Les connexions OAuth créent des profils distincts pour permettre la coexistence de plusieurs comptes.

-   Par défaut : `provider:default` lorsqu'aucun email n'est disponible.
-   OAuth avec email : `provider:` (par exemple `google-antigravity:user@gmail.com`).

Les profils se trouvent dans `~/.openclaw/agents//agent/auth-profiles.json` sous `profiles`.

## Ordre de rotation

Lorsqu'un fournisseur a plusieurs profils, OpenClaw choisit un ordre comme suit :

1.  **Configuration explicite** : `auth.order[provider]` (si défini).
2.  **Profils configurés** : `auth.profiles` filtrés par fournisseur.
3.  **Profils stockés** : entrées dans `auth-profiles.json` pour le fournisseur.

Si aucun ordre explicite n'est configuré, OpenClaw utilise un ordre en tourniquet (round‑robin) :

-   **Clé primaire :** type de profil (**OAuth avant les clés API**).
-   **Clé secondaire :** `usageStats.lastUsed` (le plus ancien d'abord, au sein de chaque type).
-   Les **profils en refroidissement/désactivés** sont déplacés à la fin, ordonnés par expiration la plus proche.

### Affinité de session (respect du cache)

OpenClaw **épingle le profil d'authentification choisi par session** pour garder les caches des fournisseurs actifs. Il ne **procède pas** à une rotation à chaque requête. Le profil épinglé est réutilisé jusqu'à ce que :

-   la session soit réinitialisée (`/new` / `/reset`)
-   une compaction se termine (le compteur de compaction s'incrémente)
-   le profil soit en refroidissement/désactivé

La sélection manuelle via `/model …@` définit un **remplacement utilisateur** pour cette session et n'est pas auto‑rotée jusqu'au début d'une nouvelle session. Les profils auto‑épinglés (sélectionnés par le routeur de session) sont traités comme une **préférence** : ils sont essayés en premier, mais OpenClaw peut basculer vers un autre profil en cas de limites de débit/timeouts. Les profils épinglés par l'utilisateur restent verrouillés sur ce profil ; s'il échoue et que des modèles de secours sont configurés, OpenClaw passe au modèle suivant au lieu de changer de profil.

### Pourquoi OAuth peut "sembler perdu"

Si vous avez à la fois un profil OAuth et un profil de clé API pour le même fournisseur, le tourniquet peut basculer entre eux d'un message à l'autre, sauf s'ils sont épinglés. Pour forcer un seul profil :

-   Épinglez avec `auth.order[provider] = ["provider:profileId"]`, ou
-   Utilisez un remplacement par session via `/model …` avec un remplacement de profil (lorsque pris en charge par votre interface/surface de chat).

## Périodes de refroidissement

Lorsqu'un profil échoue en raison d'erreurs d'authentification/limite de débit (ou d'un timeout qui ressemble à une limitation), OpenClaw le marque en refroidissement et passe au profil suivant. Les erreurs de format/de requête invalide (par exemple les échecs de validation d'ID d'appel d'outil Cloud Code Assist) sont traitées comme justifiant une bascule et utilisent les mêmes périodes de refroidissement. Les erreurs de raison d'arrêt compatibles OpenAI telles que `Unhandled stop reason: error`, `stop reason: error`, et `reason: error` sont classées comme des signaux de timeout/bascule. Les refroidissements utilisent un backoff exponentiel :

-   1 minute
-   5 minutes
-   25 minutes
-   1 heure (plafond)

L'état est stocké dans `auth-profiles.json` sous `usageStats` :

```json
{
  "usageStats": {
    "provider:profile": {
      "lastUsed": 1736160000000,
      "cooldownUntil": 1736160600000,
      "errorCount": 2
    }
  }
}
```

## Désactivations de facturation

Les échecs de facturation/crédit (par exemple "crédits insuffisants" / "solde de crédit trop faible") sont traités comme justifiant une bascule, mais ils ne sont généralement pas transitoires. Au lieu d'un court refroidissement, OpenClaw marque le profil comme **désactivé** (avec un backoff plus long) et passe au profil/fournisseur suivant. L'état est stocké dans `auth-profiles.json` :

```json
{
  "usageStats": {
    "provider:profile": {
      "disabledUntil": 1736178000000,
      "disabledReason": "billing"
    }
  }
}
```

Valeurs par défaut :

-   Le backoff de facturation commence à **5 heures**, double par échec de facturation, et est plafonné à **24 heures**.
-   Les compteurs de backoff sont réinitialisés si le profil n'a pas échoué depuis **24 heures** (configurable).

## Bascule de modèle

Si tous les profils d'un fournisseur échouent, OpenClaw passe au modèle suivant dans `agents.defaults.model.fallbacks`. Cela s'applique aux échecs d'authentification, aux limites de débit et aux timeouts ayant épuisé la rotation des profils (les autres erreurs ne déclenchent pas la bascule). Lorsqu'une exécution commence avec un remplacement de modèle (hooks ou CLI), les bascules s'arrêtent toujours à `agents.defaults.model.primary` après avoir essayé les bascules configurées.

## Configuration associée

Voir [Configuration de la passerelle](../gateway/configuration.md) pour :

-   `auth.profiles` / `auth.order`
-   `auth.cooldowns.billingBackoffHours` / `auth.cooldowns.billingBackoffHoursByProvider`
-   `auth.cooldowns.billingMaxHours` / `auth.cooldowns.failureWindowHours`
-   `agents.defaults.model.primary` / `agents.defaults.model.fallbacks`
-   routage `agents.defaults.imageModel`

Voir [Modèles](./models.md) pour un aperçu plus large de la sélection et de la bascule de modèle.

[Fournisseurs de modèles](./model-providers.md)[Anthropic](../providers/anthropic.md)