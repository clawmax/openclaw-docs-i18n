

  Fournisseurs

  
# Qwen

Qwen propose un flux OAuth gratuit pour les modèles Qwen Coder et Qwen Vision (2 000 requêtes/jour, soumis aux limites de débit de Qwen).

## Activer le plugin

```bash
openclaw plugins enable qwen-portal-auth
```

Redémarrez la Gateway après l'activation.

## S'authentifier

```bash
openclaw models auth login --provider qwen-portal --set-default
```

Cela exécute le flux OAuth par code d'appareil de Qwen et écrit une entrée de fournisseur dans votre `models.json` (plus un alias `qwen` pour une commutation rapide).

## ID de modèles

-   `qwen-portal/coder-model`
-   `qwen-portal/vision-model`

Changez de modèle avec :

```bash
openclaw models set qwen-portal/coder-model
```

## Réutiliser la connexion CLI Qwen Code

Si vous êtes déjà connecté avec la CLI Qwen Code, OpenClaw synchronisera les informations d'identification depuis `~/.qwen/oauth_creds.json` lors du chargement du magasin d'authentification. Vous avez toujours besoin d'une entrée `models.providers.qwen-portal` (utilisez la commande de connexion ci-dessus pour en créer une).

## Notes

-   Les jetons se rafraîchissent automatiquement ; réexécutez la commande de connexion si le rafraîchissement échoue ou si l'accès est révoqué.
-   URL de base par défaut : `https://portal.qwen.ai/v1` (remplacez par `models.providers.qwen-portal.baseUrl` si Qwen fournit un point de terminaison différent).
-   Voir [Fournisseurs de modèles](../concepts/model-providers.md) pour les règles générales des fournisseurs.

[Qianfan](./qianfan.md)[Synthétique](./synthetic.md)

---