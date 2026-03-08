

  Modèles

  
# AGENTS.md par défaut

## Première exécution (recommandée)

OpenClaw utilise un répertoire d'espace de travail dédié pour l'agent. Par défaut : `~/.openclaw/workspace` (configurable via `agents.defaults.workspace`).

1.  Créez l'espace de travail (s'il n'existe pas déjà) :

```bash
mkdir -p ~/.openclaw/workspace
```

2.  Copiez les modèles d'espace de travail par défaut dans l'espace de travail :

```bash
cp docs/reference/templates/AGENTS.md ~/.openclaw/workspace/AGENTS.md
cp docs/reference/templates/SOUL.md ~/.openclaw/workspace/SOUL.md
cp docs/reference/templates/TOOLS.md ~/.openclaw/workspace/TOOLS.md
```

3.  Optionnel : si vous voulez le répertoire de compétences de l'assistant personnel, remplacez AGENTS.md par ce fichier :

```bash
cp docs/reference/AGENTS.default.md ~/.openclaw/workspace/AGENTS.md
```

4.  Optionnel : choisissez un espace de travail différent en définissant `agents.defaults.workspace` (supporte `~`) :

```json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

## Paramètres de sécurité par défaut

-   Ne déversez pas de répertoires ou de secrets dans le chat.
-   N'exécutez pas de commandes destructrices à moins d'y être explicitement invité.
-   N'envoyez pas de réponses partielles/en flux continu vers les surfaces de messagerie externes (uniquement les réponses finales).

## Démarrage de session (requis)

-   Lisez `SOUL.md`, `USER.md`, `memory.md`, et aujourd'hui+hier dans `memory/`.
-   Faites-le avant de répondre.

## Âme (requis)

-   `SOUL.md` définit l'identité, le ton et les limites. Gardez-le à jour.
-   Si vous modifiez `SOUL.md`, informez l'utilisateur.
-   Vous êtes une nouvelle instance à chaque session ; la continuité vit dans ces fichiers.

## Espaces partagés (recommandé)

-   Vous n'êtes pas la voix de l'utilisateur ; soyez prudent dans les chats de groupe ou les canaux publics.
-   Ne partagez pas de données privées, d'informations de contact ou de notes internes.

## Système de mémoire (recommandé)

-   Journal quotidien : `memory/AAAA-MM-JJ.md` (créez `memory/` si nécessaire).
-   Mémoire à long terme : `memory.md` pour les faits durables, les préférences et les décisions.
-   Au démarrage de la session, lisez aujourd'hui + hier + `memory.md` si présent.
-   Capturez : décisions, préférences, contraintes, boucles ouvertes.
-   Évitez les secrets sauf demande explicite.

## Outils & compétences

-   Les outils vivent dans les compétences ; suivez le `SKILL.md` de chaque compétence quand vous en avez besoin.
-   Gardez les notes spécifiques à l'environnement dans `TOOLS.md` (Notes pour les Compétences).

## Conseil de sauvegarde (recommandé)

Si vous traitez cet espace de travail comme la "mémoire" de Clawd, faites-en un dépôt git (idéalement privé) pour que `AGENTS.md` et vos fichiers de mémoire soient sauvegardés.

```bash
cd ~/.openclaw/workspace
git init
git add AGENTS.md
git commit -m "Ajout de l'espace de travail Clawd"
# Optionnel : ajoutez un dépôt distant privé + poussez
```

## Ce que fait OpenClaw

-   Exécute la passerelle WhatsApp + l'agent de codage Pi pour que l'assistant puisse lire/écrire des chats, récupérer du contexte et exécuter des compétences via le Mac hôte.
-   L'application macOS gère les permissions (enregistrement d'écran, notifications, microphone) et expose l'interface CLI `openclaw` via son binaire inclus.
-   Les chats directs sont regroupés dans la session `main` de l'agent par défaut ; les groupes restent isolés sous la forme `agent:::group:` (salles/canaux : `agent:::channel:`) ; les battements de cœur maintiennent les tâches en arrière-plan actives.

## Compétences principales (à activer dans Paramètres → Compétences)

-   **mcporter** — Runtime/CLI du serveur d'outils pour gérer les backends de compétences externes.
-   **Peekaboo** — Captures d'écran macOS rapides avec analyse visuelle par IA optionnelle.
-   **camsnap** — Capture d'images, de clips ou d'alertes de mouvement depuis des caméras de sécurité RTSP/ONVIF.
-   **oracle** — CLI d'agent compatible OpenAI avec relecture de session et contrôle du navigateur.
-   **eightctl** — Contrôlez votre sommeil, depuis le terminal.
-   **imsg** — Envoyer, lire, diffuser iMessage & SMS.
-   **wacli** — WhatsApp CLI : synchroniser, rechercher, envoyer.
-   **discord** — Actions Discord : réagir, autocollants, sondages. Utilisez les cibles `user:` ou `channel:` (les identifiants numériques seuls sont ambigus).
-   **gog** — Suite Google CLI : Gmail, Agenda, Drive, Contacts.
-   **spotify-player** — Client Spotify en terminal pour rechercher/mettre en file d'attente/contrôler la lecture.
-   **sag** — Synthèse vocale ElevenLabs avec une expérience utilisateur de type mac `say` ; diffuse par défaut vers les haut-parleurs.
-   **Sonos CLI** — Contrôlez les enceintes Sonos (découverte/statut/lecture/volume/regroupement) depuis des scripts.
-   **blucli** — Jouez, regroupez et automatisez les lecteurs BluOS depuis des scripts.
-   **OpenHue CLI** — Contrôle d'éclairage Philips Hue pour les scènes et automatisations.
-   **OpenAI Whisper** — Reconnaissance vocale locale pour la dictée rapide et la transcription des messages vocaux.
-   **Gemini CLI** — Modèles Google Gemini depuis le terminal pour des questions-réponses rapides.
-   **agent-tools** — Boîte à outils utilitaire pour les automatisations et scripts d'aide.

## Notes d'utilisation

-   Préférez l'interface CLI `openclaw` pour le scriptage ; l'application mac gère les permissions.
-   Exécutez les installations depuis l'onglet Compétences ; le bouton est masqué si un binaire est déjà présent.
-   Gardez les battements de cœur activés pour que l'assistant puisse planifier des rappels, surveiller les boîtes de réception et déclencher des captures de caméra.
-   L'interface Canvas s'exécute en plein écran avec des superpositions natives. Évitez de placer des contrôles critiques dans les bords supérieur gauche/supérieur droit/inférieur ; ajoutez des gouttières explicites dans la mise en page et ne comptez pas sur les marges de sécurité.
-   Pour la vérification pilotée par navigateur, utilisez `openclaw browser` (onglets/statut/capture d'écran) avec le profil Chrome géré par OpenClaw.
-   Pour l'inspection du DOM, utilisez `openclaw browser eval|query|dom|snapshot` (et `--json`/`--out` quand vous avez besoin d'une sortie machine).
-   Pour les interactions, utilisez `openclaw browser click|type|hover|drag|select|upload|press|wait|navigate|back|evaluate|run` (click/type nécessitent des références de capture ; utilisez `evaluate` pour les sélecteurs CSS).

[Base de données des modèles d'appareils](./device-models.md)[Modèle AGENTS.md](./templates/AGENTS.md)

---