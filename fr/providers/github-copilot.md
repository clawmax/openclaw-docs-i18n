title: "Comment utiliser GitHub Copilot comme fournisseur de modèle dans OpenClaw"
description: "Apprenez deux façons d'intégrer GitHub Copilot avec OpenClaw : le fournisseur intégré et le plugin Copilot Proxy. Inclut les commandes de configuration."
keywords: ["github copilot", "openclaw", "assistant de codage IA", "fournisseur de modèle", "copilot proxy", "configuration cli", "authentification login", "gpt-4o"]
---

  Fournisseurs

  
# GitHub Copilot

## Qu'est-ce que GitHub Copilot ?

GitHub Copilot est l'assistant de codage IA de GitHub. Il donne accès aux modèles Copilot pour votre compte et votre abonnement GitHub. OpenClaw peut utiliser Copilot comme fournisseur de modèle de deux manières différentes.

## Deux façons d'utiliser Copilot dans OpenClaw

### 1) Fournisseur GitHub Copilot intégré (github-copilot)

Utilisez le flux de connexion par appareil natif pour obtenir un jeton GitHub, puis échangez-le contre des jetons d'API Copilot lors de l'exécution d'OpenClaw. C'est le chemin **par défaut** et le plus simple car il ne nécessite pas VS Code.

### 2) Plugin Copilot Proxy (copilot-proxy)

Utilisez l'extension VS Code **Copilot Proxy** comme pont local. OpenClaw communique avec le point de terminaison `/v1` du proxy et utilise la liste de modèles que vous y configurez. Choisissez cette option lorsque vous exécutez déjà Copilot Proxy dans VS Code ou devez passer par lui. Vous devez activer le plugin et garder l'extension VS Code en cours d'exécution. Utilisez GitHub Copilot comme fournisseur de modèle (`github-copilot`). La commande de connexion exécute le flux d'appareil GitHub, enregistre un profil d'authentification et met à jour votre configuration pour utiliser ce profil.

## Configuration en CLI

```bash
openclaw models auth login-github-copilot
```

Il vous sera demandé de visiter une URL et d'entrer un code à usage unique. Gardez le terminal ouvert jusqu'à la fin de l'opération.

### Options facultatives

```bash
openclaw models auth login-github-copilot --profile-id github-copilot:work
openclaw models auth login-github-copilot --yes
```

## Définir un modèle par défaut

```bash
openclaw models set github-copilot/gpt-4o
```

### Extrait de configuration

```json
{
  agents: { defaults: { model: { primary: "github-copilot/gpt-4o" } } },
}
```

## Notes

-   Nécessite un TTY interactif ; exécutez-la directement dans un terminal.
-   La disponibilité des modèles Copilot dépend de votre abonnement ; si un modèle est rejeté, essayez un autre identifiant (par exemple `github-copilot/gpt-4.1`).
-   La connexion stocke un jeton GitHub dans le magasin de profils d'authentification et l'échange contre un jeton d'API Copilot lors de l'exécution d'OpenClaw.

[Deepgram](./deepgram.md)[Hugging Face (Inference)](./huggingface.md)