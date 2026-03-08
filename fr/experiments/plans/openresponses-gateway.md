

  Expériences

  
# Plan de Passerelle OpenResponses

## Contexte

OpenClaw Gateway expose actuellement un point de terminaison minimal compatible OpenAI Chat Completions à l'adresse `/v1/chat/completions` (voir [OpenAI Chat Completions](../../gateway/openai-http-api.md)). Open Responses est un standard d'inférence ouvert basé sur l'API OpenAI Responses. Il est conçu pour les flux de travail agentiques et utilise des entrées basées sur des items ainsi que des événements de streaming sémantique. La spécification OpenResponses définit `/v1/responses`, et non `/v1/chat/completions`.

## Objectifs

-   Ajouter un point de terminaison `/v1/responses` qui respecte la sémantique OpenResponses.
-   Conserver Chat Completions comme une couche de compatibilité facile à désactiver et à supprimer à terme.
-   Standardiser la validation et l'analyse avec des schémas isolés et réutilisables.

## Non-objectifs

-   Une parité complète des fonctionnalités OpenResponses dans la première version (images, fichiers, outils hébergés).
-   Remplacer la logique d'exécution interne des agents ou l'orchestration d'outils.
-   Modifier le comportement existant de `/v1/chat/completions` pendant la première phase.

## Résumé de la Recherche

Sources : OpenAPI OpenResponses, site de spécification OpenResponses, et l'article de blog Hugging Face. Points clés extraits :

-   `POST /v1/responses` accepte les champs `CreateResponseBody` comme `model`, `input` (chaîne ou `ItemParam[]`), `instructions`, `tools`, `tool_choice`, `stream`, `max_output_tokens`, et `max_tool_calls`.
-   `ItemParam` est une union discriminée de :
    -   items `message` avec les rôles `system`, `developer`, `user`, `assistant`
    -   `function_call` et `function_call_output`
    -   `reasoning`
    -   `item_reference`
-   Les réponses réussies renvoient une `ResponseResource` avec `object: "response"`, `status`, et des items `output`.
-   Le streaming utilise des événements sémantiques tels que :
    -   `response.created`, `response.in_progress`, `response.completed`, `response.failed`
    -   `response.output_item.added`, `response.output_item.done`
    -   `response.content_part.added`, `response.content_part.done`
    -   `response.output_text.delta`, `response.output_text.done`
-   La spécification requiert :
    -   `Content-Type: text/event-stream`
    -   `event:` doit correspondre au champ JSON `type`
    -   l'événement terminal doit être le littéral `[DONE]`
-   Les items de raisonnement peuvent exposer `content`, `encrypted_content`, et `summary`.
-   Les exemples HF incluent `OpenResponses-Version: latest` dans les requêtes (en-tête optionnel).

## Architecture Proposée

-   Ajouter `src/gateway/open-responses.schema.ts` contenant uniquement des schémas Zod (pas d'imports de passerelle).
-   Ajouter `src/gateway/openresponses-http.ts` (ou `open-responses-http.ts`) pour `/v1/responses`.
-   Conserver `src/gateway/openai-http.ts` intact comme adaptateur de compatibilité hérité.
-   Ajouter la configuration `gateway.http.endpoints.responses.enabled` (par défaut `false`).
-   Conserver `gateway.http.endpoints.chatCompletions.enabled` indépendant ; permettre aux deux points de terminaison d'être activés/désactivés séparément.
-   Émettre un avertissement au démarrage lorsque Chat Completions est activé pour signaler son statut hérité.

## Voie de Dépréciation pour Chat Completions

-   Maintenir des limites de module strictes : aucun type de schéma partagé entre responses et chat completions.
-   Rendre Chat Completions optionnel par configuration pour pouvoir le désactiver sans modifications de code.
-   Mettre à jour la documentation pour étiqueter Chat Completions comme hérité une fois que `/v1/responses` est stable.
-   Étape future optionnelle : mapper les requêtes Chat Completions vers le gestionnaire Responses pour une voie de suppression plus simple.

## Sous-ensemble Supporté en Phase 1

-   Accepter `input` comme chaîne ou `ItemParam[]` avec les rôles de message et `function_call_output`.
-   Extraire les messages système et développeur dans `extraSystemPrompt`.
-   Utiliser le dernier `user` ou `function_call_output` comme message courant pour les exécutions d'agent.
-   Rejeter les parties de contenu non supportées (image/fichier) avec `invalid_request_error`.
-   Renvoyer un seul message assistant avec un contenu `output_text`.
-   Renvoyer `usage` avec des valeurs à zéro jusqu'à ce que la comptabilisation des tokens soit connectée.

## Stratégie de Validation (Sans SDK)

-   Implémenter des schémas Zod pour le sous-ensemble supporté de :
    -   `CreateResponseBody`
    -   `ItemParam` + unions de parties de contenu de message
    -   `ResponseResource`
    -   Formes d'événements de streaming utilisées par la passerelle
-   Conserver les schémas dans un seul module isolé pour éviter la divergence et permettre une génération de code future.

## Implémentation du Streaming (Phase 1)

-   Lignes SSE avec à la fois `event:` et `data:`.
-   Séquence requise (minimale viable) :
    -   `response.created`
    -   `response.output_item.added`
    -   `response.content_part.added`
    -   `response.output_text.delta` (répéter au besoin)
    -   `response.output_text.done`
    -   `response.content_part.done`
    -   `response.completed`
    -   `[DONE]`

## Plan de Tests et Vérification

-   Ajouter une couverture e2e pour `/v1/responses` :
    -   Authentification requise
    -   Forme de réponse non-stream
    -   Ordre des événements de stream et `[DONE]`
    -   Routage de session avec en-têtes et `user`
-   Conserver `src/gateway/openai-http.test.ts` inchangé.
-   Manuel : curl vers `/v1/responses` avec `stream: true` et vérifier l'ordre des événements et le terminal `[DONE]`.

## Mises à Jour de la Documentation (À Suivre)

-   Ajouter une nouvelle page de documentation pour l'utilisation et les exemples de `/v1/responses`.
-   Mettre à jour `/gateway/openai-http-api` avec une note "hérité" et un renvoi vers `/v1/responses`.

[Refactorisation CDP Browser Evaluate](./browser-evaluate-cdp-refactor.md)[Plan PTY et Supervision de Processus](./pty-process-supervision.md)