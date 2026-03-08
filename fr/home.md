title: "OpenClaw - Passerelle d'agent IA auto-hébergée pour WhatsApp Telegram Discord"
description: "OpenClaw est une passerelle auto-hébergée connectant WhatsApp, Telegram, Discord et iMessage à des agents d'IA de codage. Apprenez à installer et configurer votre assistant IA personnel."
keywords: ["openclaw", "ia auto-hébergée", "passerelle d'agent ia", "bot whatsapp", "bot telegram", "bot discord", "assistant ia", "passerelle multi-canaux"]
---

  Accueil

  
# OpenClaw

  ![](../images/openclaw-logo-text-dark.png)
  ![](../images/openclaw-logo-text.png)

> *"EXFOLIEZ ! EXFOLIEZ !"* — Un homard spatial, probablement

  **Une passerelle multi-OS pour les agents IA sur WhatsApp, Telegram, Discord, iMessage, et plus.** Envoyez un message, recevez une réponse d'agent depuis votre poche. Les plugins ajoutent Mattermost et plus.

  
  
  

## Qu'est-ce qu'OpenClaw ?

OpenClaw est une **passerelle auto-hébergée** qui connecte vos applications de chat préférées — WhatsApp, Telegram, Discord, iMessage, et plus — à des agents d'IA de codage comme Pi. Vous exécutez un seul processus Passerelle sur votre propre machine (ou un serveur), et il devient le pont entre vos applications de messagerie et un assistant IA toujours disponible.

**Pour qui est-ce ?** Les développeurs et utilisateurs avancés qui veulent un assistant IA personnel auquel ils peuvent envoyer des messages depuis n'importe où — sans abandonner le contrôle de leurs données ni dépendre d'un service hébergé.

**Qu'est-ce qui le différencie ?**

- **Auto-hébergé** : s'exécute sur votre matériel, selon vos règles
- **Multi-canaux** : une seule Passerelle sert WhatsApp, Telegram, Discord, et plus simultanément
- **Natif aux agents** : conçu pour les agents de codage avec utilisation d'outils, sessions, mémoire et routage multi-agents
- **Open source** : licence MIT, piloté par la communauté

**De quoi avez-vous besoin ?** Node 22+, une clé API de votre fournisseur choisi, et 5 minutes. Pour la meilleure qualité et sécurité, utilisez le modèle de dernière génération le plus puissant disponible.

## Comment ça marche

La Passerelle est la source unique de vérité pour les sessions, le routage et les connexions aux canaux.

## Fonctionnalités clés

  - **Passerelle multi-canaux** — WhatsApp, Telegram, Discord et iMessage avec un seul processus Passerelle.

  - **Canaux par plugins** — Ajoutez Mattermost et plus avec des packages d

  - **Routage multi-agents** — Sessions isolées par agent, espace de travail ou expéditeur.

  - **Support des médias** — Envoyez et recevez des images, de l

  - **Interface Web de contrôle** — Tableau de bord navigateur pour le chat, la config, les sessions et les nœuds.

  

## Démarrage rapide

  ### Installer OpenClaw

```bash
npm install -g openclaw@latest
```

  

  ### Intégrer et installer le service

```bash
openclaw onboard --install-daemon
```

  

  ### Appairer WhatsApp et démarrer la Passerelle

```bash
openclaw channels login
openclaw gateway --port 18789
```

  

Besoin de l'installation complète et de la configuration dev ? Voir [Démarrage rapide](./start/quickstart.md).

## Tableau de bord

Ouvrez l'interface de contrôle navigateur après le démarrage de la Passerelle.

- Local par défaut : [http://127.0.0.1:18789/](http://127.0.0.1:18789/)
- Accès distant : [Surfaces Web](./web.md) et [Tailscale](./gateway/tailscale.md)

  ![](../images/whatsapp-openclaw.jpg)

## Configuration (optionnelle)

La configuration se trouve dans `~/.openclaw/openclaw.json`.

- Si vous **ne faites rien**, OpenClaw utilise le binaire Pi intégré en mode RPC avec des sessions par expéditeur.
- Si vous voulez le sécuriser, commencez par `channels.whatsapp.allowFrom` et (pour les groupes) les règles de mention.

Exemple :

```json
{
  "channels": {
    "whatsapp": {
      "allowFrom": ["+15555550123"],
      "groups": { "*": { "requireMention": true } }
    }
  },
  "messages": { "groupChat": { "mentionPatterns": ["@openclaw"] } }
}
```

## Commencez ici

  
  
  
  
  
  

## En savoir plus

  
  
  
  
  

---