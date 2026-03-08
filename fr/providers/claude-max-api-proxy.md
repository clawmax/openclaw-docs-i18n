

  Fournisseurs

  
# Proxy API Claude Max

**claude-max-api-proxy** est un outil communautaire qui expose votre abonnement Claude Max/Pro en tant que point de terminaison d'API compatible OpenAI. Cela vous permet d'utiliser votre abonnement avec tout outil qui prend en charge le format d'API OpenAI.

> **⚠️** Cette voie est une compatibilité technique uniquement. Anthropic a déjà bloqué certaines utilisations d'abonnement en dehors de Claude Code par le passé. Vous devez décider vous-même de l'utiliser et vérifier les conditions actuelles d'Anthropic avant de vous y fier.

## Pourquoi l'utiliser ?

| Approche | Coût | Idéal pour |
| --- | --- | --- |
| API Anthropic | Paiement par token (~15/M d'entrée, 75/M de sortie pour Opus) | Applications en production, volume élevé |
| Abonnement Claude Max | 200 $/mois forfaitaire | Usage personnel, développement, utilisation illimitée |

Si vous avez un abonnement Claude Max et souhaitez l'utiliser avec des outils compatibles OpenAI, ce proxy peut réduire les coûts pour certains flux de travail. Les clés API restent la voie la plus claire en termes de politique pour un usage en production.

## Comment ça fonctionne

```
Votre App → claude-max-api-proxy → CLI Claude Code → Anthropic (via abonnement)
     (format OpenAI)              (convertit le format)      (utilise votre connexion)
```

Le proxy :

1.  Accepte les requêtes au format OpenAI à l'adresse `http://localhost:3456/v1/chat/completions`
2.  Les convertit en commandes CLI Claude Code
3.  Renvoie les réponses au format OpenAI (streaming pris en charge)

## Installation

```bash
# Nécessite Node.js 20+ et le CLI Claude Code
npm install -g claude-max-api-proxy

# Vérifiez que le CLI Claude est authentifié
claude --version
```

## Utilisation

### Démarrer le serveur

```
claude-max-api
# Le serveur tourne sur http://localhost:3456
```

### Tester

```bash
# Vérification de santé
curl http://localhost:3456/health

# Lister les modèles
curl http://localhost:3456/v1/models

# Complétion de chat
curl http://localhost:3456/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-opus-4",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

### Avec OpenClaw

Vous pouvez pointer OpenClaw vers le proxy en tant que point de terminaison personnalisé compatible OpenAI :

```json
{
  env: {
    OPENAI_API_KEY: "not-needed",
    OPENAI_BASE_URL: "http://localhost:3456/v1",
  },
  agents: {
    defaults: {
      model: { primary: "openai/claude-opus-4" },
    },
  },
}
```

## Modèles disponibles

| ID Modèle | Correspond à |
| --- | --- |
| `claude-opus-4` | Claude Opus 4 |
| `claude-sonnet-4` | Claude Sonnet 4 |
| `claude-haiku-4` | Claude Haiku 4 |

## Démarrage automatique sur macOS

Créez un LaunchAgent pour exécuter le proxy automatiquement :

```bash
cat > ~/Library/LaunchAgents/com.claude-max-api.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>com.claude-max-api</string>
  <key>RunAtLoad</key>
  <true/>
  <key>KeepAlive</key>
  <true/>
  <key>ProgramArguments</key>
  <array>
    <string>/usr/local/bin/node</string>
    <string>/usr/local/lib/node_modules/claude-max-api-proxy/dist/server/standalone.js</string>
  </array>
  <key>EnvironmentVariables</key>
  <dict>
    <key>PATH</key>
    <string>/usr/local/bin:/opt/homebrew/bin:~/.local/bin:/usr/bin:/bin</string>
  </dict>
</dict>
</plist>
EOF

launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.claude-max-api.plist
```

## Liens

-   **npm :** [https://www.npmjs.com/package/claude-max-api-proxy](https://www.npmjs.com/package/claude-max-api-proxy)
-   **GitHub :** [https://github.com/atalovesyou/claude-max-api-proxy](https://github.com/atalovesyou/claude-max-api-proxy)
-   **Problèmes :** [https://github.com/atalovesyou/claude-max-api-proxy/issues](https://github.com/atalovesyou/claude-max-api-proxy/issues)

## Notes

-   Il s'agit d'un **outil communautaire**, non officiellement supporté par Anthropic ou OpenClaw
-   Nécessite un abonnement Claude Max/Pro actif avec le CLI Claude Code authentifié
-   Le proxy s'exécute localement et n'envoie pas de données à des serveurs tiers
-   Les réponses en streaming sont entièrement prises en charge

## Voir aussi

-   [Fournisseur Anthropic](./anthropic.md) - Intégration native OpenClaw avec Claude setup-token ou clés API
-   [Fournisseur OpenAI](./openai.md) - Pour les abonnements OpenAI/Codex

[Cloudflare AI Gateway](./cloudflare-ai-gateway.md)[Deepgram](./deepgram.md)