

  Herramientas integradas

  
# Herramienta apply_patch

Aplica cambios en archivos usando un formato de parche estructurado. Esto es ideal para ediciones multiarchivo o con múltiples fragmentos donde una sola llamada `edit` sería frágil. La herramienta acepta una única cadena `input` que envuelve una o más operaciones de archivo:

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

## Parámetros

-   `input` (requerido): Contenido completo del parche incluyendo `*** Begin Patch` y `*** End Patch`.

## Notas

-   Las rutas en el parche admiten rutas relativas (desde el directorio del espacio de trabajo) y rutas absolutas.
-   `tools.exec.applyPatch.workspaceOnly` tiene como valor predeterminado `true` (contenido en el espacio de trabajo). Establécelo en `false` solo si intencionalmente quieres que `apply_patch` escriba/elimine fuera del directorio del espacio de trabajo.
-   Usa `*** Move to:` dentro de un fragmento `*** Update File:` para renombrar archivos.
-   `*** End of File` marca una inserción solo al final del archivo cuando sea necesario.
-   Experimental y deshabilitado por defecto. Habilítalo con `tools.exec.applyPatch.enabled`.
-   Solo para OpenAI (incluyendo OpenAI Codex). Opcionalmente, restringe por modelo mediante `tools.exec.applyPatch.allowModels`.
-   La configuración está solo bajo `tools.exec`.

## Ejemplo

```json
{
  "tool": "apply_patch",
  "input": "*** Begin Patch\n*** Update File: src/index.ts\n@@\n-const foo = 1\n+const foo = 2\n*** End Patch"
}
```

[Herramientas](../tools.md)[Brave Search](../brave-search.md)