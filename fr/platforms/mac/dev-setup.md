

  Application compagnon macOS

  
# Configuration de développement macOS

Ce guide couvre les étapes nécessaires pour construire et exécuter l'application macOS OpenClaw à partir des sources.

## Prérequis

Avant de construire l'application, assurez-vous d'avoir installé les éléments suivants :

1.  **Xcode 26.2+** : Requis pour le développement Swift.
2.  **Node.js 22+ & pnpm** : Requis pour la passerelle, la CLI et les scripts d'empaquetage.

## 1\. Installer les dépendances

Installez les dépendances globales du projet :

```bash
pnpm install
```

## 2\. Construire et empaqueter l'application

Pour construire l'application macOS et l'empaqueter dans `dist/OpenClaw.app`, exécutez :

```
./scripts/package-mac-app.sh
```

Si vous ne possédez pas de certificat Apple Developer ID, le script utilisera automatiquement la **signature ad-hoc** (`-`). Pour les modes d'exécution de développement, les drapeaux de signature et le dépannage de l'ID d'équipe, consultez le README de l'application macOS : [https://github.com/openclaw/openclaw/blob/main/apps/macos/README.md](https://github.com/openclaw/openclaw/blob/main/apps/macos/README.md)

> **Note** : Les applications signées ad-hoc peuvent déclencher des alertes de sécurité. Si l'application plante immédiatement avec "Abort trap 6", consultez la section [Dépannage](#dépannage).

## 3\. Installer la CLI

L'application macOS s'attend à une installation globale de la CLI `openclaw` pour gérer les tâches en arrière-plan. **Pour l'installer (recommandé) :**

1.  Ouvrez l'application OpenClaw.
2.  Allez dans l'onglet des paramètres **Général**.
3.  Cliquez sur **"Installer la CLI"**.

Alternativement, installez-la manuellement :

```bash
npm install -g openclaw@<version>
```

## Dépannage

### La construction échoue : Incompatibilité de chaîne d'outils ou SDK

La construction de l'application macOS nécessite le dernier SDK macOS et la chaîne d'outils Swift 6.2. **Dépendances système (requises) :**

-   **Dernière version de macOS disponible dans Mise à jour logicielle** (requise par les SDK Xcode 26.2)
-   **Xcode 26.2** (chaîne d'outils Swift 6.2)

**Vérifications :**

```bash
xcodebuild -version
xcrun swift --version
```

Si les versions ne correspondent pas, mettez à jour macOS/Xcode et relancez la construction.

### L'application plante lors de l'octroi d'une autorisation

Si l'application plante lorsque vous essayez d'autoriser l'accès à la **Reconnaissance vocale** ou au **Microphone**, cela peut être dû à un cache TCC corrompu ou à une incompatibilité de signature. **Solution :**

1.  Réinitialisez les permissions TCC :
    
    Copier
    
    ```bash
    tccutil reset All ai.openclaw.mac.debug
    ```
    
2.  Si cela échoue, modifiez temporairement le `BUNDLE_ID` dans [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) pour forcer un "état vierge" de la part de macOS.

### La passerelle reste sur "Démarrage…" indéfiniment

Si le statut de la passerelle reste sur "Démarrage…", vérifiez si un processus zombie occupe le port :

```bash
openclaw gateway status
openclaw gateway stop

# Si vous n'utilisez pas de LaunchAgent (mode dev / exécutions manuelles), trouvez l'écouteur :
lsof -nP -iTCP:18789 -sTCP:LISTEN
```

Si une exécution manuelle occupe le port, arrêtez ce processus (Ctrl+C). En dernier recours, tuez le PID trouvé ci-dessus.

[Raspberry Pi](../raspberry-pi.md)[Barre de menu](./menu-bar.md)