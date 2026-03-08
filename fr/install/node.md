

  Runtime Node

  
# Node.js

OpenClaw nécessite **Node 22 ou une version plus récente**. Le [script d'installation](../install.md#install-methods) détectera et installera Node automatiquement — cette page est destinée aux cas où vous souhaitez configurer Node vous-même et vous assurer que tout est correctement configuré (versions, PATH, installations globales).

## Vérifiez votre version

```bash
node -v
```

Si cela affiche `v22.x.x` ou une version supérieure, c'est bon. Si Node n'est pas installé ou si la version est trop ancienne, choisissez une méthode d'installation ci-dessous.

## Installer Node

 

Les gestionnaires de versions vous permettent de basculer facilement entre les versions de Node. Options populaires :

-   [**fnm**](https://github.com/Schniz/fnm) — rapide, multiplateforme
-   [**nvm**](https://github.com/nvm-sh/nvm) — largement utilisé sur macOS/Linux
-   [**mise**](https://mise.jdx.dev/) — polyglotte (Node, Python, Ruby, etc.)

Exemple avec fnm :

```bash
fnm install 22
fnm use 22
```

Assurez-vous que votre gestionnaire de versions est initialisé dans votre fichier de démarrage du shell (`~/.zshrc` ou `~/.bashrc`). Si ce n'est pas le cas, `openclaw` risque de ne pas être trouvé dans les nouvelles sessions de terminal car le PATH n'inclura pas le répertoire bin de Node.

## Dépannage

### openclaw : commande introuvable

Cela signifie presque toujours que le répertoire bin global de npm n'est pas dans votre PATH.

### Étape 1 : Trouvez votre préfixe global npm

```bash
npm prefix -g
```

### Étape 2 : Vérifiez s'il est dans votre PATH

```bash
echo "$PATH"
```

Recherchez `<npm-prefix>/bin` (macOS/Linux) ou `<npm-prefix>` (Windows) dans la sortie.

### Étape 3 : Ajoutez-le à votre fichier de démarrage du shell

```bash
export PATH="$(npm prefix -g)/bin:$PATH"
```

Ajoutez la sortie de `npm prefix -g` à votre PATH système via Paramètres → Système → Variables d'environnement.

### Erreurs de permission sur npm install -g (Linux)

Si vous voyez des erreurs `EACCES`, changez le préfixe global de npm vers un répertoire accessible en écriture par l'utilisateur :

```bash
mkdir -p "$HOME/.npm-global"
npm config set prefix "$HOME/.npm-global"
export PATH="$HOME/.npm-global/bin:$PATH"
```

Ajoutez la ligne `export PATH=...` à votre `~/.bashrc` ou `~/.zshrc` pour la rendre permanente.

[Diagnostics Flags](../diagnostics/flags.md)[Session Management Deep Dive](../reference/session-management-compaction.md)

---