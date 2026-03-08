title: "Outil Apply Patch pour les modifications multi-fichiers dans OpenClaw AI"
description: "Apprenez à utiliser l'outil apply_patch pour effectuer des modifications de code structurées et multi-fichiers en toute sécurité. Idéal pour les modifications complexes utilisant un format de patch."
keywords: ["apply_patch", "outil de patch", "modifications multi-fichiers", "patch structuré", "outils openclaw", "opérations sur fichiers", "édition de code", "gestion d'espace de travail"]
---

  Outils intégrés

  
# Outil apply_patch

Appliquez des modifications de fichiers en utilisant un format de patch structuré. Cet outil est idéal pour les modifications multi-fichiers ou multi-blocs où un simple appel `edit` serait fragile. L'outil accepte une seule chaîne `input` qui encapsule une ou plusieurs opérations sur fichiers :

```markdown
*** Begin Patch
*** Add File: path/to/file.txt
+line 1
+line 2
*** Update File: src/app.ts
@@
-old line
+new line
*** Delete File: obsolete.txt
*** End Patch
```

## Paramètres

-   `input` (requis) : Contenu complet du patch incluant `*** Begin Patch` et `*** End Patch`.

## Notes

-   Les chemins dans le patch prennent en charge les chemins relatifs (depuis le répertoire de l'espace de travail) et les chemins absolus.
-   `tools.exec.applyPatch.workspaceOnly` est par défaut `true` (contenu dans l'espace de travail). Ne le définissez à `false` que si vous souhaitez intentionnellement que `apply_patch` écrive/supprime en dehors du répertoire de l'espace de travail.
-   Utilisez `*** Move to:` dans un bloc `*** Update File:` pour renommer des fichiers.
-   `*** End of File` marque une insertion uniquement en fin de fichier si nécessaire.
-   Expérimental et désactivé par défaut. Activez-le avec `tools.exec.applyPatch.enabled`.
-   Uniquement pour OpenAI (y compris OpenAI Codex). Optionnellement restreint par modèle via `tools.exec.applyPatch.allowModels`.
-   La configuration se trouve uniquement sous `tools.exec`.

## Exemple

```json
{
  "tool": "apply_patch",
  "input": "*** Begin Patch\n*** Update File: src/index.ts\n@@\n-const foo = 1\n+const foo = 2\n*** End Patch"
}
```

[Outils](../tools.md)[Brave Search](../brave-search.md)