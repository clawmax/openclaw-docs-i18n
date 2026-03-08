title: "Journalisation macOS pour le débogage et le diagnostic de l'application OpenClaw"
description: "Apprenez à activer les journaux de fichiers rotatifs et la journalisation unifiée avec données privées pour déboguer l'application macOS OpenClaw. Configurez la verbosité et visualisez les charges utiles sensibles."
keywords: ["journalisation macos", "diagnostic openclaw", "fichier journal rotatif", "journalisation unifiée", "débogage application mac", "journalisation données privées", "journal jsonl", "configuration journal"]
---

  Application compagnon macOS

  
# Journalisation macOS

## Journal de diagnostic rotatif (fichier) (Panneau de débogage)

OpenClaw achemine les journaux de l'application macOS via swift-log (journalisation unifiée par défaut) et peut écrire un journal rotatif local sur le disque lorsque vous avez besoin d'une capture durable.

-   Verbosité : **Panneau de débogage → Journaux → Journalisation de l'app → Verbosité**
-   Activer : **Panneau de débogage → Journaux → Journalisation de l'app → “Écrire le journal de diagnostic rotatif (JSONL)”**
-   Emplacement : `~/Library/Logs/OpenClaw/diagnostics.jsonl` (rotation automatique ; les anciens fichiers sont suffixés par `.1`, `.2`, …)
-   Effacer : **Panneau de débogage → Journaux → Journalisation de l'app → “Effacer”**

Notes :

-   Ceci est **désactivé par défaut**. Activez-le uniquement pendant un débogage actif.
-   Traitez ce fichier comme sensible ; ne le partagez pas sans l'avoir examiné.

## Données privées dans la journalisation unifiée sur macOS

La journalisation unifiée masque la plupart des charges utiles à moins qu'un sous-système n'opte pour `privacy -off`. Comme l'explique Peter dans son article sur les [magouilles de confidentialité de la journalisation macOS](https://steipete.me/posts/2025/logging-privacy-shenanigans) (2025), ceci est contrôlé par un fichier plist dans `/Library/Preferences/Logging/Subsystems/` indexé par le nom du sous-système. Seules les nouvelles entrées de journal prennent en compte ce drapeau, activez-le donc avant de reproduire un problème.

## Activer pour OpenClaw (ai.openclaw)

-   Écrivez d'abord le fichier plist dans un fichier temporaire, puis installez-le de manière atomique en tant que root :

```bash
cat <<'EOF' >/tmp/ai.openclaw.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>DEFAULT-OPTIONS</key>
    <dict>
        <key>Enable-Private-Data</key>
        <true/>
    </dict>
</dict>
</plist>
EOF
sudo install -m 644 -o root -g wheel /tmp/ai.openclaw.plist /Library/Preferences/Logging/Subsystems/ai.openclaw.plist
```

-   Aucun redémarrage n'est requis ; logd détecte le fichier rapidement, mais seules les nouvelles lignes de journal incluront les charges utiles privées.
-   Visualisez la sortie enrichie avec l'utilitaire existant, par exemple `./scripts/clawlog.sh --category WebChat --last 5m`.

## Désactiver après le débogage

-   Supprimez la substitution : `sudo rm /Library/Preferences/Logging/Subsystems/ai.openclaw.plist`.
-   Exécutez éventuellement `sudo log config --reload` pour forcer logd à abandonner immédiatement la substitution.
-   N'oubliez pas que cette surface peut inclure des numéros de téléphone et des corps de messages ; gardez le fichier plist en place uniquement pendant que vous avez activement besoin de ces détails supplémentaires.

[Icône de la barre de menus](./icon.md)[Autorisations macOS](./permissions.md)