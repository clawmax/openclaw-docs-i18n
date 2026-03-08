title: "Sémantique des justificatifs d'authentification et règles d'éligibilité OpenClaw"
description: "Apprenez les règles canoniques d'éligibilité et de résolution pour les justificatifs de jeton dans OpenClaw, y compris les codes de raison stables et la logique de validation pour les profils d'authentification."
keywords: ["sémantique des justificatifs d'authentification", "règles d'éligibilité des jetons", "resolveauthprofileorder", "tokenref", "validation expires", "authentification openclaw", "résolution des justificatifs", "sonde de profil d'authentification"]
---

  Configuration et opérations

  
# Sémantique des justificatifs d'authentification

Ce document définit la sémantique canonique d'éligibilité et de résolution des justificatifs utilisée dans :

-   `resolveAuthProfileOrder`
-   `resolveApiKeyForProfile`
-   `models status --probe`
-   `doctor-auth`

L'objectif est de maintenir alignés le comportement au moment de la sélection et le comportement à l'exécution.

## Codes de Raison Stables

-   `ok`
-   `missing_credential`
-   `invalid_expires`
-   `expired`
-   `unresolved_ref`

## Justificatifs de Jeton

Les justificatifs de jeton (`type: "token"`) prennent en charge `token` en ligne et/ou `tokenRef`.

### Règles d'éligibilité

1.  Un profil de jeton est inéligible lorsque `token` et `tokenRef` sont tous deux absents.
2.  `expires` est facultatif.
3.  Si `expires` est présent, il doit s'agir d'un nombre fini supérieur à `0`.
4.  Si `expires` est invalide (`NaN`, `0`, négatif, non fini ou mauvais type), le profil est inéligible avec `invalid_expires`.
5.  Si `expires` est dans le passé, le profil est inéligible avec `expired`.
6.  `tokenRef` ne contourne pas la validation de `expires`.

### Règles de résolution

1.  La sémantique du résolveur correspond à la sémantique d'éligibilité pour `expires`.
2.  Pour les profils éligibles, le matériel du jeton peut être résolu à partir de la valeur en ligne ou de `tokenRef`.
3.  Les références irrésolvables produisent `unresolved_ref` dans la sortie de `models status --probe`.

## Messagerie Compatible avec l'Ancien Système

Pour la compatibilité des scripts, les erreurs de sonde conservent cette première ligne inchangée : `Auth profile credentials are missing or expired.` Des détails conviviaux et des codes de raison stables peuvent être ajoutés sur les lignes suivantes.

[Authentification](./gateway/authentication.md)[Gestion des secrets](./gateway/secrets.md)

---