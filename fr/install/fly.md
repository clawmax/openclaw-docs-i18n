

  Hébergement et déploiement

  
# Fly.io

**Objectif :** OpenClaw Gateway fonctionnant sur une machine [Fly.io](https://fly.io) avec stockage persistant, HTTPS automatique et accès à Discord/aux canaux.

## Ce dont vous avez besoin

-   [flyctl CLI](https://fly.io/docs/hands-on/install-flyctl/) installé
-   Compte Fly.io (le niveau gratuit fonctionne)
-   Authentification du modèle : clé API pour votre fournisseur de modèle choisi
-   Identifiants du canal : jeton de bot Discord, jeton Telegram, etc.

## Chemin rapide pour débutant

1.  Cloner le dépôt → personnaliser `fly.toml`
2.  Créer l'application + le volume → définir les secrets
3.  Déployer avec `fly deploy`
4.  Se connecter en SSH pour créer la configuration ou utiliser l'interface de contrôle

## 1) Créer l'application Fly

```bash
# Cloner le dépôt
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# Créer une nouvelle application Fly (choisissez votre propre nom)
fly apps create my-openclaw

# Créer un volume persistant (1GB est généralement suffisant)
fly volumes create openclaw_data --size 1 --region iad
```

**Astuce :** Choisissez une région proche de vous. Options courantes : `lhr` (Londres), `iad` (Virginie), `sjc` (San Jose).

## 2) Configurer fly.toml

Modifiez `fly.toml` pour correspondre au nom de votre application et à vos besoins. **Note de sécurité :** La configuration par défaut expose une URL publique. Pour un déploiement renforcé sans IP publique, consultez [Déploiement Privé](#private-deployment-hardened) ou utilisez `fly.private.toml`.

```
app = "my-openclaw"  # Votre nom d'application
primary_region = "iad"

[build]
  dockerfile = "Dockerfile"

[env]
  NODE_ENV = "production"
  OPENCLAW_PREFER_PNPM = "1"
  OPENCLAW_STATE_DIR = "/data"
  NODE_OPTIONS = "--max-old-space-size=1536"

[processes]
  app = "node dist/index.js gateway --allow-unconfigured --port 3000 --bind lan"

[http_service]
  internal_port = 3000
  force_https = true
  auto_stop_machines = false
  auto_start_machines = true
  min_machines_running = 1
  processes = ["app"]

[[vm]]
  size = "shared-cpu-2x"
  memory = "2048mb"

[mounts]
  source = "openclaw_data"
  destination = "/data"
```

**Paramètres clés :**

| Paramètre | Pourquoi |
| --- | --- |
| `--bind lan` | Se lie à `0.0.0.0` pour que le proxy de Fly puisse atteindre la passerelle |
| `--allow-unconfigured` | Démarre sans fichier de configuration (vous en créerez un après) |
| `internal_port = 3000` | Doit correspondre à `--port 3000` (ou `OPENCLAW_GATEWAY_PORT`) pour les vérifications de santé de Fly |
| `memory = "2048mb"` | 512MB est trop faible ; 2GB recommandé |
| `OPENCLAW_STATE_DIR = "/data"` | Persiste l'état sur le volume |

## 3) Définir les secrets

```bash
# Requis : Jeton de la passerelle (pour une liaison non-loopback)
fly secrets set OPENCLAW_GATEWAY_TOKEN=$(openssl rand -hex 32)

# Clés API des fournisseurs de modèles
fly secrets set ANTHROPIC_API_KEY=sk-ant-...

# Optionnel : Autres fournisseurs
fly secrets set OPENAI_API_KEY=sk-...
fly secrets set GOOGLE_API_KEY=...

# Jetons des canaux
fly secrets set DISCORD_BOT_TOKEN=MTQ...
```

**Notes :**

-   Les liaisons non-loopback (`--bind lan`) nécessitent `OPENCLAW_GATEWAY_TOKEN` pour la sécurité.
-   Traitez ces jetons comme des mots de passe.
-   **Préférez les variables d'environnement au fichier de configuration** pour toutes les clés API et jetons. Cela garde les secrets hors de `openclaw.json` où ils pourraient être accidentellement exposés ou enregistrés.

## 4) Déployer

```bash
fly deploy
```

Le premier déploiement construit l'image Docker (~2-3 minutes). Les déploiements suivants sont plus rapides. Après le déploiement, vérifiez :

```bash
fly status
fly logs
```

Vous devriez voir :

```ini
[gateway] listening on ws://0.0.0.0:3000 (PID xxx)
[discord] logged in to discord as xxx
```

## 5) Créer le fichier de configuration

Connectez-vous en SSH dans la machine pour créer une configuration appropriée :

```bash
fly ssh console
```

Créez le répertoire et le fichier de configuration :

```bash
mkdir -p /data
cat > /data/openclaw.json << 'EOF'
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-opus-4-6",
        "fallbacks": ["anthropic/claude-sonnet-4-5", "openai/gpt-4o"]
      },
      "maxConcurrent": 4
    },
    "list": [
      {
        "id": "main",
        "default": true
      }
    ]
  },
  "auth": {
    "profiles": {
      "anthropic:default": { "mode": "token", "provider": "anthropic" },
      "openai:default": { "mode": "token", "provider": "openai" }
    }
  },
  "bindings": [
    {
      "agentId": "main",
      "match": { "channel": "discord" }
    }
  ],
  "channels": {
    "discord": {
      "enabled": true,
      "groupPolicy": "allowlist",
      "guilds": {
        "YOUR_GUILD_ID": {
          "channels": { "general": { "allow": true } },
          "requireMention": false
        }
      }
    }
  },
  "gateway": {
    "mode": "local",
    "bind": "auto"
  },
  "meta": {
    "lastTouchedVersion": "2026.1.29"
  }
}
EOF
```

**Note :** Avec `OPENCLAW_STATE_DIR=/data`, le chemin de configuration est `/data/openclaw.json`. **Note :** Le jeton Discord peut provenir soit :

-   Variable d'environnement : `DISCORD_BOT_TOKEN` (recommandé pour les secrets)
-   Fichier de configuration : `channels.discord.token`

Si vous utilisez la variable d'environnement, inutile d'ajouter le jeton dans la configuration. La passerelle lit `DISCORD_BOT_TOKEN` automatiquement. Redémarrez pour appliquer :

```
exit
fly machine restart <machine-id>
```

## 6) Accéder à la Passerelle

### Interface de Contrôle

Ouvrez dans le navigateur :

```bash
fly open
```

Ou visitez `https://my-openclaw.fly.dev/` Collez votre jeton de passerelle (celui de `OPENCLAW_GATEWAY_TOKEN`) pour vous authentifier.

### Journaux

```bash
fly logs              # Journaux en direct
fly logs --no-tail    # Journaux récents
```

### Console SSH

```bash
fly ssh console
```

## Dépannage

### "L'application n'écoute pas sur l'adresse attendue"

La passerelle se lie à `127.0.0.1` au lieu de `0.0.0.0`. **Solution :** Ajoutez `--bind lan` à votre commande de processus dans `fly.toml`.

### Échecs des vérifications de santé / connexion refusée

Fly ne peut pas atteindre la passerelle sur le port configuré. **Solution :** Assurez-vous que `internal_port` correspond au port de la passerelle (définissez `--port 3000` ou `OPENCLAW_GATEWAY_PORT=3000`).

### OOM / Problèmes de Mémoire

Le conteneur redémarre continuellement ou est tué. Signes : `SIGABRT`, `v8::internal::Runtime_AllocateInYoungGeneration`, ou redémarrages silencieux. **Solution :** Augmentez la mémoire dans `fly.toml` :

```
[[vm]]
  memory = "2048mb"
```

Ou mettez à jour une machine existante :

```bash
fly machine update <machine-id> --vm-memory 2048 -y
```

**Note :** 512MB est trop faible. 1GB peut fonctionner mais peut provoquer un OOM sous charge ou avec une journalisation verbeuse. **2GB est recommandé.**

### Problèmes de Verrouillage de la Passerelle

La passerelle refuse de démarrer avec des erreurs "déjà en cours d'exécution". Cela se produit lorsque le conteneur redémarre mais que le fichier de verrouillage PID persiste sur le volume. **Solution :** Supprimez le fichier de verrouillage :

```bash
fly ssh console --command "rm -f /data/gateway.*.lock"
fly machine restart <machine-id>
```

Le fichier de verrouillage se trouve à `/data/gateway.*.lock` (pas dans un sous-répertoire).

### Configuration Non Lue

Si vous utilisez `--allow-unconfigured`, la passerelle crée une configuration minimale. Votre configuration personnalisée à `/data/openclaw.json` devrait être lue au redémarrage. Vérifiez que la configuration existe :

```bash
fly ssh console --command "cat /data/openclaw.json"
```

### Écrire la Configuration via SSH

La commande `fly ssh console -C` ne prend pas en charge la redirection shell. Pour écrire un fichier de configuration :

```bash
# Utiliser echo + tee (pipe du local vers le distant)
echo '{"your":"config"}' | fly ssh console -C "tee /data/openclaw.json"

# Ou utiliser sftp
fly sftp shell
> put /local/path/config.json /data/openclaw.json
```

**Note :** `fly sftp` peut échouer si le fichier existe déjà. Supprimez-le d'abord :

```bash
fly ssh console --command "rm /data/openclaw.json"
```

### État Non Persistant

Si vous perdez les identifiants ou les sessions après un redémarrage, le répertoire d'état écrit sur le système de fichiers du conteneur. **Solution :** Assurez-vous que `OPENCLAW_STATE_DIR=/data` est défini dans `fly.toml` et redéployez.

## Mises à jour

```bash
# Récupérer les dernières modifications
git pull

# Redéployer
fly deploy

# Vérifier l'état de santé
fly status
fly logs
```

### Mettre à jour la Commande de la Machine

Si vous devez modifier la commande de démarrage sans un redéploiement complet :

```bash
# Obtenir l'ID de la machine
fly machines list

# Mettre à jour la commande
fly machine update <machine-id> --command "node dist/index.js gateway --port 3000 --bind lan" -y

# Ou avec augmentation de la mémoire
fly machine update <machine-id> --vm-memory 2048 --command "node dist/index.js gateway --port 3000 --bind lan" -y
```

**Note :** Après `fly deploy`, la commande de la machine peut revenir à ce qui est dans `fly.toml`. Si vous avez apporté des modifications manuelles, réappliquez-les après le déploiement.

## Déploiement Privé (Renforcé)

Par défaut, Fly alloue des IPs publiques, rendant votre passerelle accessible à `https://your-app.fly.dev`. C'est pratique mais signifie que votre déploiement est découvrable par les scanners internet (Shodan, Censys, etc.). Pour un déploiement renforcé **sans exposition publique**, utilisez le modèle privé.

### Quand utiliser un déploiement privé

-   Vous ne faites que des appels/messages **sortants** (pas de webhooks entrants)
-   Vous utilisez des tunnels **ngrok ou Tailscale** pour tout rappel de webhook
-   Vous accédez à la passerelle via **SSH, proxy, ou WireGuard** au lieu du navigateur
-   Vous voulez que le déploiement soit **caché des scanners internet**

### Configuration

Utilisez `fly.private.toml` au lieu de la configuration standard :

```bash
# Déployer avec la configuration privée
fly deploy -c fly.private.toml
```

Ou convertissez un déploiement existant :

```bash
# Lister les IPs actuelles
fly ips list -a my-openclaw

# Libérer les IPs publiques
fly ips release <public-ipv4> -a my-openclaw
fly ips release <public-ipv6> -a my-openclaw

# Passer à la configuration privée pour que les futurs déploiements n'allouent pas d'IPs publiques
# (supprimez [http_service] ou déployez avec le modèle privé)
fly deploy -c fly.private.toml

# Allouer une IPv6 privée uniquement
fly ips allocate-v6 --private -a my-openclaw
```

Après cela, `fly ips list` devrait montrer uniquement une IP de type `private` :

```bash
VERSION  IP                   TYPE             REGION
v6       fdaa:x:x:x:x::x      private          global
```

### Accéder à un déploiement privé

Puisqu'il n'y a pas d'URL publique, utilisez l'une de ces méthodes : **Option 1 : Proxy local (le plus simple)**

```bash
# Rediriger le port local 3000 vers l'application
fly proxy 3000:3000 -a my-openclaw

# Puis ouvrir http://localhost:3000 dans le navigateur
```

**Option 2 : VPN WireGuard**

```bash
# Créer la configuration WireGuard (une fois)
fly wireguard create

# Importer dans le client WireGuard, puis accéder via l'IPv6 interne
# Exemple : http://[fdaa:x:x:x:x::x]:3000
```

**Option 3 : SSH uniquement**

```bash
fly ssh console -a my-openclaw
```

### Webhooks avec déploiement privé

Si vous avez besoin de rappels de webhook (Twilio, Telnyx, etc.) sans exposition publique :

1.  **Tunnel ngrok** - Exécutez ngrok à l'intérieur du conteneur ou en sidecar
2.  **Tailscale Funnel** - Exposez des chemins spécifiques via Tailscale
3.  **Sortant uniquement** - Certains fournisseurs (Twilio) fonctionnent bien pour les appels sortants sans webhooks

Exemple de configuration d'appel vocal avec ngrok :

```json
{
  "plugins": {
    "entries": {
      "voice-call": {
        "enabled": true,
        "config": {
          "provider": "twilio",
          "tunnel": { "provider": "ngrok" },
          "webhookSecurity": {
            "allowedHosts": ["example.ngrok.app"]
          }
        }
      }
    }
  }
}
```

Le tunnel ngrok s'exécute à l'intérieur du conteneur et fournit une URL de webhook publique sans exposer l'application Fly elle-même. Définissez `webhookSecurity.allowedHosts` sur le nom d'hôte public du tunnel pour que les en-têtes d'hôte transmis soient acceptés.

### Avantages en matière de sécurité

| Aspect | Public | Privé |
| --- | --- | --- |
| Scanners internet | Découvrable | Caché |
| Attaques directes | Possibles | Bloquées |
| Accès à l'interface de contrôle | Navigateur | Proxy/VPN |
| Livraison des webhooks | Directe | Via tunnel |

## Notes

-   Fly.io utilise l'architecture **x86** (pas ARM)
-   Le Dockerfile est compatible avec les deux architectures
-   Pour l'intégration WhatsApp/Telegram, utilisez `fly ssh console`
-   Les données persistantes vivent sur le volume à `/data`
-   Signal nécessite Java + signal-cli ; utilisez une image personnalisée et gardez la mémoire à 2GB+.

## Coût

Avec la configuration recommandée (`shared-cpu-2x`, 2GB RAM) :

-   ~10-15 $/mois selon l'utilisation
-   Le niveau gratuit inclut une certaine allocation

Voir [Tarification Fly.io](https://fly.io/docs/about/pricing/) pour plus de détails.

[Hébergement VPS](../vps.md)[Hetzner](./hetzner.md)