

  Comandos CLI

  
# configure

Asistente interactivo para configurar credenciales, dispositivos y valores predeterminados de agentes. Nota: La sección **Modelo** ahora incluye una selección múltiple para la lista de permitidos `agents.defaults.models` (lo que aparece en `/model` y el selector de modelos). Consejo: `openclaw config` sin un subcomando abre el mismo asistente. Usa `openclaw config get|set|unset` para ediciones no interactivas. Relacionado:

-   Referencia de configuración del Gateway: [Configuración](../gateway/configuration.md)
-   Config CLI: [Config](./config.md)

Notas:

-   Elegir dónde se ejecuta el Gateway siempre actualiza `gateway.mode`. Puedes seleccionar "Continuar" sin otras secciones si eso es todo lo que necesitas.
-   Los servicios orientados a canales (Slack/Discord/Matrix/Microsoft Teams) solicitan listas de canales/salas permitidas durante la configuración. Puedes ingresar nombres o IDs; el asistente resuelve nombres a IDs cuando es posible.
-   Si ejecutas el paso de instalación del daemon, la autenticación por token requiere un token, y `gateway.auth.token` está gestionado por SecretRef. Configure valida el SecretRef pero no persiste los valores de token en texto plano resueltos en los metadatos del entorno del servicio supervisor.
-   Si la autenticación por token requiere un token y el SecretRef configurado no está resuelto, configure bloquea la instalación del daemon con orientación de remediación accionable.
-   Si tanto `gateway.auth.token` como `gateway.auth.password` están configurados y `gateway.auth.mode` no está establecido, configure bloquea la instalación del daemon hasta que el modo se establezca explícitamente.

## Ejemplos

```bash
openclaw configure
openclaw configure --section model --section channels
```

[config](./config.md)[cron](./cron.md)

---