

  RPC et API

  
# Base de données des modèles d'appareils

L'application compagnon macOS affiche des noms de modèles d'appareils Apple conviviaux dans l'interface **Instances** en associant les identifiants de modèles Apple (par ex. `iPad16,6`, `Mac16,6`) à des noms lisibles. Ce mappage est fourni sous forme de JSON dans :

-   `apps/macos/Sources/OpenClaw/Resources/DeviceModels/`

## Source des données

Nous utilisons actuellement le mappage provenant du dépôt sous licence MIT :

-   `kyle-seongwoo-jun/apple-device-identifiers`

Pour garantir la reproductibilité des builds, les fichiers JSON sont épinglés à des commits spécifiques de la source (enregistrés dans `apps/macos/Sources/OpenClaw/Resources/DeviceModels/NOTICE.md`).

## Mise à jour de la base de données

1.  Choisissez les commits source auxquels vous souhaitez vous épingler (un pour iOS, un pour macOS).
2.  Mettez à jour les hachages de commit dans `apps/macos/Sources/OpenClaw/Resources/DeviceModels/NOTICE.md`.
3.  Téléchargez à nouveau les fichiers JSON, épinglés à ces commits :

```
IOS_COMMIT="<commit sha for ios-device-identifiers.json>"
MAC_COMMIT="<commit sha for mac-device-identifiers.json>"

curl -fsSL "https://raw.githubusercontent.com/kyle-seongwoo-jun/apple-device-identifiers/${IOS_COMMIT}/ios-device-identifiers.json" \
  -o apps/macos/Sources/OpenClaw/Resources/DeviceModels/ios-device-identifiers.json

curl -fsSL "https://raw.githubusercontent.com/kyle-seongwoo-jun/apple-device-identifiers/${MAC_COMMIT}/mac-device-identifiers.json" \
  -o apps/macos/Sources/OpenClaw/Resources/DeviceModels/mac-device-identifiers.json
```

4.  Assurez-vous que `apps/macos/Sources/OpenClaw/Resources/DeviceModels/LICENSE.apple-device-identifiers.txt` correspond toujours à la licence source (remplacez-la si la licence source change).
5.  Vérifiez que l'application macOS se compile sans erreur (aucun avertissement) :

```bash
swift build --package-path apps/macos
```

[Adaptateurs RPC](./rpc.md)[AGENTS.md par défaut](./AGENTS.default.md)