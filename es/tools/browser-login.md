

  Navegador

  
# Inicio de Sesión en el Navegador

## Inicio de sesión manual (recomendado)

Cuando un sitio requiere inicio de sesión, **inicia sesión manualmente** en el perfil del navegador **host** (el navegador openclaw). **No** le des tus credenciales al modelo. Los inicios de sesión automatizados a menudo activan defensas anti‑bots y pueden bloquear la cuenta. Volver a la documentación principal del navegador: [Navegador](./browser.md).

## ¿Qué perfil de Chrome se utiliza?

OpenClaw controla un **perfil de Chrome dedicado** (llamado `openclaw`, interfaz de usuario de color naranja). Este es independiente de tu perfil de navegador diario. Dos formas fáciles de acceder a él:

1.  **Pídele al agente que abra el navegador** y luego inicia sesión tú mismo.
2.  **Ábrelo mediante CLI**:

```bash
openclaw browser start
openclaw browser open https://x.com
```

Si tienes múltiples perfiles, pasa `--browser-profile ` (el valor por defecto es `openclaw`).

## X/Twitter: flujo recomendado

-   **Leer/buscar/hilos:** usa el navegador **host** (inicio de sesión manual).
-   **Publicar actualizaciones:** usa el navegador **host** (inicio de sesión manual).

## Aislamiento + acceso al navegador host

Las sesiones del navegador aisladas tienen **más probabilidad** de activar la detección de bots. Para X/Twitter (y otros sitios estrictos), prefiere el navegador **host**. Si el agente está aislado, la herramienta del navegador usa por defecto el entorno aislado. Para permitir el control del host:

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        browser: {
          allowHostControl: true,
        },
      },
    },
  },
}
```

Luego, apunta al navegador host:

```bash
openclaw browser open https://x.com --browser-profile openclaw --target host
```

O desactiva el aislamiento para el agente que publica actualizaciones.

[Navegador (gestionado por OpenClaw)](./browser.md)[Extensión de Chrome](./chrome-extension.md)

---