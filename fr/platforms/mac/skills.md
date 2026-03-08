

  Application compagnon macOS

  
# Compétences

L'application macOS expose les compétences OpenClaw via la passerelle ; elle n'analyse pas les compétences localement.

## Source des données

-   `skills.status` (passerelle) renvoie toutes les compétences ainsi que leur éligibilité et les exigences manquantes (y compris les blocages de liste d'autorisation pour les compétences groupées).
-   Les exigences sont dérivées de `metadata.openclaw.requires` dans chaque fichier `SKILL.md`.

## Actions d'installation

-   `metadata.openclaw.install` définit les options d'installation (brew/node/go/uv).
-   L'application appelle `skills.install` pour exécuter les installateurs sur l'hôte de la passerelle.
-   La passerelle n'expose qu'un seul installateur préféré lorsque plusieurs sont fournis (brew lorsqu'il est disponible, sinon le gestionnaire node depuis `skills.install`, npm par défaut).

## Variables d'environnement / Clés API

-   L'application stocke les clés dans `~/.openclaw/openclaw.json` sous `skills.entries.`.
-   `skills.update` met à jour les champs `enabled`, `apiKey` et `env`.

## Mode distant

-   L'installation et les mises à jour de configuration se font sur l'hôte de la passerelle (et non sur le Mac local).

[IPC macOS](./xpc.md)[Pont Peekaboo](./peekaboo.md)

---