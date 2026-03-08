

  Initialisation

  
# Initialisation de l'Agent

L'initialisation est le rituel du **premier démarrage** qui prépare un espace de travail d'agent et collecte les détails d'identité. Cela se produit après l'intégration, lorsque l'agent démarre pour la première fois.

## Ce que fait l'initialisation

Lors du premier démarrage de l'agent, OpenClaw initialise l'espace de travail (par défaut `~/.openclaw/workspace`) :

-   Crée les fichiers `AGENTS.md`, `BOOTSTRAP.md`, `IDENTITY.md`, `USER.md`.
-   Exécute un court rituel de questions-réponses (une question à la fois).
-   Écrit l'identité + les préférences dans `IDENTITY.md`, `USER.md`, `SOUL.md`.
-   Supprime `BOOTSTRAP.md` une fois terminé pour qu'il ne s'exécute qu'une seule fois.

## Où cela s'exécute

L'initialisation s'exécute toujours sur **l'hôte passerelle**. Si l'application macOS se connecte à une passerelle distante, l'espace de travail et les fichiers d'initialisation résident sur cette machine distante.

> **ℹ️** Lorsque la passerelle s'exécute sur une autre machine, modifiez les fichiers de l'espace de travail sur l'hôte passerelle (par exemple, `user@gateway-host:~/.openclaw/workspace`).

## Documentation associée

-   Intégration de l'application macOS : [Intégration](./onboarding.md)
-   Organisation de l'espace de travail : [Espace de travail de l'agent](../concepts/agent-workspace.md)

[OAuth](../concepts/oauth.md)[Gestion de session](../concepts/session.md)

---