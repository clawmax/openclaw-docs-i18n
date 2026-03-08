

  Premières étapes

  
# Intégration (Application macOS)

Ce document décrit le flux d'intégration actuel du **premier lancement**. L'objectif est une expérience "jour 0" fluide : choisir où la passerelle s'exécute, connecter l'authentification, exécuter l'assistant et laisser l'agent s'amorcer lui-même. Pour une vue d'ensemble des parcours d'intégration, consultez [Vue d'ensemble de l'intégration](./onboarding-overview.md).

### Étape 1 : Approuver l'avertissement macOS

![](../images/start-01-macos-warning.jpeg.md)

### Étape 2 : Approuver la recherche de réseaux locaux

![](../images/start-02-local-networks.jpeg.md)

### Étape 3 : Message de bienvenue et avis de sécurité

![](../images/start-03-security-notice.png.md)

Modèle de confiance de sécurité :

-   Par défaut, OpenClaw est un agent personnel : une seule limite d'opérateur de confiance.
-   Les configurations partagées/multi-utilisateurs nécessitent un verrouillage (séparation des limites de confiance, limitation de l'accès aux outils et suivi des [bonnes pratiques de sécurité](../gateway/security.md)).
-   L'intégration locale définit désormais par défaut les nouvelles configurations sur `tools.profile: "coding"` afin que les installations locales fraîches conservent les outils système de fichiers/exécution sans forcer le profil non restreint `full`.
-   Si des hooks/webhooks ou d'autres flux de contenu non fiables sont activés, utilisez un modèle moderne de niveau élevé et maintenez une politique d'outils stricte avec sandboxing.

### Étape 4 : Local vs Distant

![](../images/start-04-choose-gateway.png.md)

Où la **Passerelle** s'exécute-t-elle ?

-   **Ce Mac (Local uniquement) :** l'intégration peut configurer l'authentification et écrire les identifiants localement.
-   **Distant (via SSH/Tailnet) :** l'intégration **ne** configure **pas** l'authentification locale ; les identifiants doivent exister sur l'hôte de la passerelle.
-   **Configurer plus tard :** ignorer la configuration et laisser l'application non configurée.

> **💡** **Astuce d'authentification de la passerelle :**
>
> -   L'assistant génère désormais un **jeton** même pour la boucle locale, donc les clients WS locaux doivent s'authentifier.
> -   Si vous désactivez l'authentification, tout processus local peut se connecter ; utilisez cela uniquement sur des machines entièrement fiables.
> -   Utilisez un **jeton** pour un accès multi‑machines ou des liaisons non locales.

### Étape 5 : Autorisations

![](../images/start-05-permissions.png.md)

L'intégration demande les autorisations TCC nécessaires pour :

-   Automatisation (AppleScript)
-   Notifications
-   Accessibilité
-   Enregistrement d'écran
-   Microphone
-   Reconnaissance vocale
-   Caméra
-   Localisation

### Étape 6 : CLI

> **ℹ️** Cette étape est facultative

 L'application peut installer le CLI global `openclaw` via npm/pnpm afin que les workflows en terminal et les tâches launchd fonctionnent immédiatement.

### Étape 7 : Chat d'intégration (session dédiée)

Après la configuration, l'application ouvre une session de chat d'intégration dédiée afin que l'agent puisse se présenter et guider les prochaines étapes. Cela garde les conseils du premier lancement séparés de votre conversation normale. Voir [Amorçage](./bootstrapping.md) pour savoir ce qui se passe sur l'hôte de la passerelle lors du premier lancement de l'agent.

[Intégration : CLI](./wizard.md)[Configuration d'Assistant Personnel](./openclaw.md)

---