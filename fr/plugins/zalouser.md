title: "Guide d'installation et d'utilisation du plugin Zalo Personnel pour OpenClaw"
description: "Apprenez à installer, configurer et utiliser le plugin Zalo Personnel pour OpenClaw afin d'automatiser un compte utilisateur Zalo personnel via l'interface CLI et les outils d'agent."
keywords: ["plugin zalo personnel", "openclaw zalouser", "automatisation zalo", "installation plugin openclaw", "compte utilisateur zalo", "configuration de canal", "openclaw cli", "chatbot zalo"]
---

  Extensions

  
# Plugin Zalo Personnel

Prise en charge de Zalo Personnel pour OpenClaw via un plugin, utilisant le module natif `zca-js` pour automatiser un compte utilisateur Zalo normal.

> **Avertissement :** L'automatisation non officielle peut entraîner la suspension ou le bannissement du compte. Utilisez à vos propres risques.

## Dénomination

L'identifiant du canal est `zalouser` pour indiquer explicitement qu'il automatise un **compte utilisateur Zalo personnel** (non officiel). Nous réservons `zalo` pour une éventuelle future intégration officielle de l'API Zalo.

## Où il s'exécute

Ce plugin s'exécute **dans le processus de la Passerelle (Gateway)**. Si vous utilisez une Passerelle distante, installez-le/configurez-le sur la **machine exécutant la Passerelle**, puis redémarrez la Passerelle. Aucun binaire CLI externe `zca`/`openzca` n'est requis.

## Installation

### Option A : installer depuis npm

```bash
openclaw plugins install @openclaw/zalouser
```

Redémarrez la Passerelle ensuite.

### Option B : installer depuis un dossier local (développement)

```bash
openclaw plugins install ./extensions/zalouser
cd ./extensions/zalouser && pnpm install
```

Redémarrez la Passerelle ensuite.

## Configuration

La configuration du canal se trouve sous `channels.zalouser` (et non `plugins.entries.*`) :

```json
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing",
    },
  },
}
```

## CLI

```bash
openclaw channels login --channel zalouser
openclaw channels logout --channel zalouser
openclaw channels status --probe
openclaw message send --channel zalouser --target <threadId> --message "Hello from OpenClaw"
openclaw directory peers list --channel zalouser --query "name"
```

## Outil d'agent

Nom de l'outil : `zalouser` Actions : `send`, `image`, `link`, `friends`, `groups`, `me`, `status` Les actions de message du canal prennent également en charge `react` pour les réactions aux messages.

[Plugin Appel Vocal](./voice-call.md)[Manifeste du Plugin](./manifest.md)

---