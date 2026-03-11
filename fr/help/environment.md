

  Environnement et débogage

  
# Variables d'environnement

OpenClaw récupère les variables d'environnement depuis plusieurs sources. La règle est de **ne jamais écraser les valeurs existantes**.

## Précédence (la plus haute → la plus basse)

1.  **Environnement du processus** (ce que le processus Gateway possède déjà depuis le shell/daemon parent).
2.  **`.env` dans le répertoire de travail courant** (dotenv par défaut ; n'écrase pas).
3.  **`.env` global** à `~/.openclaw/.env` (alias `$OPENCLAW_STATE_DIR/.env` ; n'écrase pas).
4.  **Bloc `env` de la configuration** dans `~/.openclaw/openclaw.json` (appliqué uniquement si la variable est manquante).
5.  **Import optionnel du shell de connexion** (`env.shellEnv.enabled` ou `OPENCLAW_LOAD_SHELL_ENV=1`), appliqué uniquement pour les clés attendues manquantes.

Si le fichier de configuration est entièrement absent, l'étape 4 est ignorée ; l'import du shell s'exécute toujours s'il est activé.

## Bloc env de la configuration

Deux manières équivalentes de définir des variables d'environnement en ligne (les deux n'écrasent pas) :

```json
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
  },
}
```

## Import de l'environnement shell

`env.shellEnv` exécute votre shell de connexion et importe uniquement les clés attendues **manquantes** :

```json
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

Équivalents en variables d'environnement :

-   `OPENCLAW_LOAD_SHELL_ENV=1`
-   `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

## Variables d'environnement injectées à l'exécution

OpenClaw injecte également des marqueurs de contexte dans les processus enfants lancés :

-   `OPENCLAW_SHELL=exec` : défini pour les commandes exécutées via l'outil `exec`.
-   `OPENCLAW_SHELL=acp` : défini pour les lancements de processus backend d'exécution ACP (par exemple `acpx`).
-   `OPENCLAW_SHELL=acp-client` : défini pour `openclaw acp client` lorsqu'il lance le processus de pont ACP.
-   `OPENCLAW_SHELL=tui-local` : défini pour les commandes shell `!` locales de l'interface utilisateur textuelle.

Ce sont des marqueurs d'exécution (pas une configuration utilisateur requise). Ils peuvent être utilisés dans la logique shell/profil pour appliquer des règles spécifiques au contexte.

## Substitution de variables d'environnement dans la configuration

Vous pouvez référencer des variables d'environnement directement dans les valeurs de chaîne de configuration en utilisant la syntaxe `${VAR_NAME}` :

```json
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}",
      },
    },
  },
}
```

Voir [Configuration : Substitution de variables d'environnement](../gateway/configuration.md#env-var-substitution-in-config) pour plus de détails.

## Références secrètes vs chaînes `\${ENV}`

OpenClaw prend en charge deux modèles basés sur l'environnement :

-   Substitution de chaîne `${VAR}` dans les valeurs de configuration.
-   Objets SecretRef (`{ source: "env", provider: "default", id: "VAR" }`) pour les champs qui prennent en charge les références de secrets.

Les deux se résolvent à partir de l'environnement du processus au moment de l'activation. Les détails de SecretRef sont documentés dans [Gestion des secrets](../gateway/secrets.md).

## Variables d'environnement liées aux chemins

| Variable | Objectif |
| --- | --- |
| `OPENCLAW_HOME` | Remplace le répertoire personnel utilisé pour toutes les résolutions de chemin internes (`~/.openclaw/`, répertoires d'agents, sessions, identifiants). Utile lors de l'exécution d'OpenClaw en tant qu'utilisateur de service dédié. |
| `OPENCLAW_STATE_DIR` | Remplace le répertoire d'état (par défaut `~/.openclaw`). |
| `OPENCLAW_CONFIG_PATH` | Remplace le chemin du fichier de configuration (par défaut `~/.openclaw/openclaw.json`). |

## Journalisation

| Variable | Objectif |
| --- | --- |
| `OPENCLAW_LOG_LEVEL` | Remplace le niveau de journalisation pour le fichier et la console (par ex. `debug`, `trace`). A la priorité sur `logging.level` et `logging.consoleLevel` dans la configuration. Les valeurs invalides sont ignorées avec un avertissement. |

### OPENCLAW\_HOME

Lorsqu'elle est définie, `OPENCLAW_HOME` remplace le répertoire personnel système (`$HOME` / `os.homedir()`) pour toutes les résolutions de chemin internes. Cela permet une isolation complète du système de fichiers pour les comptes de service sans interface. **Précédence :** `OPENCLAW_HOME` > `$HOME` > `USERPROFILE` > `os.homedir()` **Exemple** (macOS LaunchDaemon) :

```
<key>EnvironmentVariables</key>
<dict>
  <key>OPENCLAW_HOME</key>
  <string>/Users/kira</string>
</dict>
```

`OPENCLAW_HOME` peut également être défini sur un chemin avec tilde (par ex. `~/svc`), qui est développé en utilisant `$HOME` avant utilisation.

## Voir aussi

-   [Configuration de la passerelle](../gateway/configuration.md)
-   [FAQ : variables d'environnement et chargement .env](./faq.md#env-vars-and-env-loading)
-   [Vue d'ensemble des modèles](../concepts/models.md)

[Lore OpenClaw](../start/lore.md)[Débogage](./debugging.md)