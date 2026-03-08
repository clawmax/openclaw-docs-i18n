

  Contribuyendo

  
# Pipeline CI

La CI se ejecuta en cada push a `main` y en cada pull request. Utiliza un alcance inteligente para omitir trabajos costosos cuando solo cambian documentación o código nativo.

## Resumen de Trabajos

| Trabajo | Propósito | Cuándo se ejecuta |
| --- | --- | --- |
| `docs-scope` | Detectar cambios solo en documentación | Siempre |
| `changed-scope` | Detectar qué áreas cambiaron (node/macos/android/windows) | PRs que no son de documentación |
| `check` | Tipos de TypeScript, lint, formato | Push a `main`, o PRs con cambios relevantes para Node |
| `check-docs` | Lint de Markdown + verificación de enlaces rotos | Documentación cambiada |
| `code-analysis` | Verificación de umbral de LOC (1000 líneas) | Solo PRs |
| `secrets` | Detectar secretos filtrados | Siempre |
| `build-artifacts` | Construir dist una vez, compartir con otros trabajos | Cambios no de documentación, cambios en node |
| `release-check` | Validar contenido de npm pack | Después de la construcción |
| `checks` | Pruebas Node/Bun + verificación de protocolo | Cambios no de documentación, cambios en node |
| `checks-windows` | Pruebas específicas de Windows | Cambios no de documentación, cambios relevantes para windows |
| `macos` | Lint/construcción/pruebas de Swift + pruebas TS | PRs con cambios en macos |
| `android` | Construcción Gradle + pruebas | Cambios no de documentación, cambios en android |

## Orden de Fallo Rápido

Los trabajos están ordenados para que las verificaciones baratas fallen antes de que se ejecuten las costosas:

1.  `docs-scope` + `code-analysis` + `check` (paralelo, ~1-2 min)
2.  `build-artifacts` (bloqueado por lo anterior)
3.  `checks`, `checks-windows`, `macos`, `android` (bloqueado por la construcción)

La lógica de alcance reside en `scripts/ci-changed-scope.mjs` y está cubierta por pruebas unitarias en `src/scripts/ci-changed-scope.test.ts`.

## Ejecutores

| Ejecutor | Trabajos |
| --- | --- |
| `blacksmith-16vcpu-ubuntu-2404` | La mayoría de trabajos Linux, incluyendo detección de alcance |
| `blacksmith-32vcpu-windows-2025` | `checks-windows` |
| `macos-latest` | `macos`, `ios` |

## Equivalentes Locales

```bash
pnpm check          # tipos + lint + formato
pnpm test           # pruebas vitest
pnpm check:docs     # formato docs + lint + enlaces rotos
pnpm release:check  # validar npm pack
```

[Flujo de Desarrollo Pi](./pi-dev.md)[Hubs de Documentación](./start/hubs.md)

---