title: "Référence d'hygiène des transcriptions OpenClaw pour les correctifs spécifiques aux fournisseurs"
description: "Découvrez les règles d'hygiène des transcriptions spécifiques aux fournisseurs et les correctifs en mémoire appliqués au contexte du modèle dans OpenClaw, y compris l'assainissement des appels d'outils et la gestion des charges utiles d'images."
keywords: ["hygiène des transcriptions", "correctifs spécifiques aux fournisseurs", "assainissement des appels d'outils", "assainissement des images", "historique des sessions", "référence openclaw", "contexte du modèle", "réparation des transcriptions"]
---

  Référence technique

  
# Hygiène des transcriptions

Ce document décrit les **correctifs spécifiques aux fournisseurs** appliqués aux transcriptions avant une exécution (construction du contexte du modèle). Il s'agit d'ajustements **en mémoire** utilisés pour satisfaire les exigences strictes des fournisseurs. Ces étapes d'hygiène ne **réécrivent pas** la transcription JSONL stockée sur le disque ; cependant, une passe de réparation séparée des fichiers de session peut réécrire les fichiers JSONL malformés en supprimant les lignes invalides avant que la session ne soit chargée. Lorsqu'une réparation se produit, le fichier original est sauvegardé à côté du fichier de session. Le périmètre comprend :

-   L'assainissement des identifiants d'appel d'outils
-   La validation des entrées d'appel d'outils
-   La réparation des paires résultat/appel d'outils
-   La validation / réorganisation des tours de parole
-   Le nettoyage des signatures de raisonnement
-   L'assainissement des charges utiles d'images
-   L'étiquetage de provenance des entrées utilisateur (pour les invites routées entre sessions)

Si vous avez besoin de détails sur le stockage des transcriptions, consultez :

-   [/reference/session-management-compaction](./session-management-compaction.md)

* * *

## Où cela s'exécute

Toute l'hygiène des transcriptions est centralisée dans l'exécuteur embarqué :

-   Sélection de la politique : `src/agents/transcript-policy.ts`
-   Application de l'assainissement/réparation : `sanitizeSessionHistory` dans `src/agents/pi-embedded-runner/google.ts`

La politique utilise `provider`, `modelApi` et `modelId` pour décider quoi appliquer. Séparément de l'hygiène des transcriptions, les fichiers de session sont réparés (si nécessaire) avant le chargement :

-   `repairSessionFileIfNeeded` dans `src/agents/session-file-repair.ts`
-   Appelé depuis `run/attempt.ts` et `compact.ts` (exécuteur embarqué)

* * *

## Règle globale : assainissement des images

Les charges utiles d'images sont toujours assainies pour éviter le rejet par le fournisseur en raison de limites de taille (réduction/compression des images base64 trop volumineuses). Cela aide également à contrôler la pression sur les tokens induite par les images pour les modèles compatibles avec la vision. Des dimensions maximales plus faibles réduisent généralement l'utilisation de tokens ; des dimensions plus élevées préservent les détails. Implémentation :

-   `sanitizeSessionMessagesImages` dans `src/agents/pi-embedded-helpers/images.ts`
-   `sanitizeContentBlocksImages` dans `src/agents/tool-images.ts`
-   Le côté maximal de l'image est configurable via `agents.defaults.imageMaxDimensionPx` (par défaut : `1200`).

* * *

## Règle globale : appels d'outils malformés

Les blocs d'appel d'outils de l'assistant qui manquent à la fois `input` et `arguments` sont supprimés avant la construction du contexte du modèle. Cela empêche les rejets du fournisseur dus à des appels d'outils partiellement persistants (par exemple, après un échec de limite de débit). Implémentation :

-   `sanitizeToolCallInputs` dans `src/agents/session-transcript-repair.ts`
-   Appliqué dans `sanitizeSessionHistory` dans `src/agents/pi-embedded-runner/google.ts`

* * *

## Règle globale : provenance des entrées inter-sessions

Lorsqu'un agent envoie une invite dans une autre session via `sessions_send` (y compris les étapes de réponse/annonce d'agent à agent), OpenClaw persiste le tour utilisateur créé avec :

-   `message.provenance.kind = "inter_session"`

Ces métadonnées sont écrites au moment de l'ajout à la transcription et ne changent pas le rôle (`role: "user"` reste pour la compatibilité avec les fournisseurs). Les lecteurs de transcriptions peuvent les utiliser pour éviter de traiter les invites internes routées comme des instructions de l'utilisateur final. Lors de la reconstruction du contexte, OpenClaw ajoute également en mémoire un court marqueur `[Message inter-session]` au début de ces tours utilisateur afin que le modèle puisse les distinguer des instructions externes de l'utilisateur final.

* * *

## Matrice des fournisseurs (comportement actuel)

**OpenAI / OpenAI Codex**

-   Assainissement des images uniquement.
-   Suppression des signatures de raisonnement orphelines (éléments de raisonnement isolés sans bloc de contenu suivant) pour les transcriptions OpenAI Responses/Codex.
-   Pas d'assainissement des identifiants d'appel d'outils.
-   Pas de réparation des paires résultat/appel d'outils.
-   Pas de validation ou de réorganisation des tours de parole.
-   Pas de résultats d'outils synthétiques.
-   Pas de suppression des signatures de raisonnement.

**Google (Generative AI / Gemini CLI / Antigravity)**

-   Assainissement des identifiants d'appel d'outils : alphanumérique strict.
-   Réparation des paires résultat/appel d'outils et résultats d'outils synthétiques.
-   Validation des tours (alternance de style Gemini).
-   Correction de l'ordre des tours Google (ajout d'un minuscule amorçage utilisateur si l'historique commence par l'assistant).
-   Antigravity Claude : normalisation des signatures de raisonnement ; suppression des blocs de raisonnement non signés.

**Anthropic / Minimax (compatible Anthropic)**

-   Réparation des paires résultat/appel d'outils et résultats d'outils synthétiques.
-   Validation des tours (fusion des tours utilisateur consécutifs pour satisfaire l'alternance stricte).

**Mistral (y compris la détection basée sur l'identifiant du modèle)**

-   Assainissement des identifiants d'appel d'outils : strict9 (alphanumérique de longueur 9).

**OpenRouter Gemini**

-   Nettoyage des signatures de raisonnement : suppression des valeurs `thought_signature` non base64 (conserver le base64).

**Tout le reste**

-   Assainissement des images uniquement.

* * *

## Comportement historique (pré-2026.1.22)

Avant la version 2026.1.22, OpenClaw appliquait plusieurs couches d'hygiène des transcriptions :

-   Une **extension transcript-sanitize** s'exécutait à chaque construction de contexte et pouvait :
    -   Réparer les paires utilisation/résultat d'outils.
    -   Assainir les identifiants d'appel d'outils (y compris un mode non strict qui préservait `_`/`-`).
-   L'exécuteur effectuait également un assainissement spécifique au fournisseur, ce qui dupliquait le travail.
-   Des mutations supplémentaires se produisaient en dehors de la politique du fournisseur, notamment :
    -   La suppression des balises `` du texte de l'assistant avant la persistance.
    -   La suppression des tours d'erreur de l'assistant vides.
    -   L'élagage du contenu de l'assistant après les appels d'outils.

Cette complexité a causé des régressions inter-fournisseurs (notamment l'appariement `call_id|fc_id` pour `openai-responses`). Le nettoyage de la version 2026.1.22 a supprimé l'extension, centralisé la logique dans l'exécuteur et rendu OpenAI **non modifiable** au-delà de l'assainissement des images.

[Utilisation et coûts de l'API](./api-usage-costs.md)[Date et heure](../date-time.md)