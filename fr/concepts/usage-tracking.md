

  Fonctionnement interne des concepts

  
# Suivi d'utilisation

## Qu'est-ce que c'est

-   Récupère l'utilisation/les quotas directement depuis les points de terminaison d'utilisation des fournisseurs.
-   Pas de coûts estimés ; uniquement les fenêtres rapportées par le fournisseur.

## Où cela apparaît

-   `/status` dans les chats : carte d'état riche en emojis avec les tokens de session + coût estimé (clé API uniquement). L'utilisation du fournisseur s'affiche pour le **fournisseur de modèle actuel** quand disponible.
-   `/usage off|tokens|full` dans les chats : pied de page d'utilisation par réponse (OAuth n'affiche que les tokens).
-   `/usage cost` dans les chats : résumé local des coûts agrégé à partir des journaux de session OpenClaw.
-   CLI : `openclaw status --usage` affiche un détail complet par fournisseur.
-   CLI : `openclaw channels list` affiche le même instantané d'utilisation à côté de la configuration du fournisseur (utilisez `--no-usage` pour l'ignorer).
-   Barre de menus macOS : section "Utilisation" sous Contexte (uniquement si disponible).

## Fournisseurs + identifiants

-   **Anthropic (Claude)**: Tokens OAuth dans les profils d'authentification.
-   **GitHub Copilot**: Tokens OAuth dans les profils d'authentification.
-   **Gemini CLI**: Tokens OAuth dans les profils d'authentification.
-   **Antigravity**: Tokens OAuth dans les profils d'authentification.
-   **OpenAI Codex**: Tokens OAuth dans les profils d'authentification (accountId utilisé quand présent).
-   **MiniMax**: Clé API (clé du plan de codage ; `MINIMAX_CODE_PLAN_KEY` ou `MINIMAX_API_KEY`) ; utilise la fenêtre du plan de codage de 5 heures.
-   **z.ai**: Clé API via env/config/auth store.

L'utilisation est masquée si aucune identifiant OAuth/clé API correspondant n'existe.

[Indicateurs de frappe](./typing-indicators.md)[Fuseaux horaires](./timezone.md)