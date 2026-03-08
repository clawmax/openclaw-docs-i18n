

  Proveedores

  
# Proxy de API Claude Max

**claude-max-api-proxy** es una herramienta comunitaria que expone tu suscripción a Claude Max/Pro como un endpoint de API compatible con OpenAI. Esto te permite usar tu suscripción con cualquier herramienta que soporte el formato de API de OpenAI.

> **⚠️** Esta ruta es solo para compatibilidad técnica. Anthropic ha bloqueado en el pasado algunos usos de suscripciones fuera de Claude Code. Debes decidir por ti mismo si usarla y verificar los términos actuales de Anthropic antes de confiar en ella.

## ¿Por qué usar esto?

| Enfoque | Costo | Mejor para |
| --- | --- | --- |
| API de Anthropic | Pago por token (~15/M entrada, 75/M salida para Opus) | Aplicaciones en producción, alto volumen |
| Suscripción Claude Max | $200/mes fijo | Uso personal, desarrollo, uso ilimitado |

Si tienes una suscripción a Claude Max y quieres usarla con herramientas compatibles con OpenAI, este proxy puede reducir costos para algunos flujos de trabajo. Las claves de API siguen siendo el camino de política más claro para uso en producción.

## Cómo funciona

```
Tu App → claude-max-api-proxy → CLI de Claude Code → Anthropic (vía suscripción)
     (Formato OpenAI)              (convierte formato)      (usa tu inicio de sesión)
```

El proxy:

1.  Acepta solicitudes en formato OpenAI en `http://localhost:3456/v1/chat/completions`
2.  Las convierte en comandos del CLI de Claude Code
3.  Devuelve respuestas en formato OpenAI (se soporta streaming)

## Instalación

```bash
# Requiere Node.js 20+ y CLI de Claude Code
npm install -g claude-max-api-proxy

# Verifica que el CLI de Claude esté autenticado
claude --version
```

## Uso

### Iniciar el servidor

```
claude-max-api
# El servidor corre en http://localhost:3456
```

### Probarlo

```bash
# Verificación de estado
curl http://localhost:3456/health

# Listar modelos
curl http://localhost:3456/v1/models

# Completado de chat
curl http://localhost:3456/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-opus-4",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

### Con OpenClaw

Puedes apuntar OpenClaw al proxy como un endpoint personalizado compatible con OpenAI:

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

## Modelos disponibles

| ID del Modelo | Se Mapea A |
| --- | --- |
| `claude-opus-4` | Claude Opus 4 |
| `claude-sonnet-4` | Claude Sonnet 4 |
| `claude-haiku-4` | Claude Haiku 4 |

## Inicio Automático en macOS

Crea un LaunchAgent para ejecutar el proxy automáticamente:

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

## Enlaces

-   **npm:** [https://www.npmjs.com/package/claude-max-api-proxy](https://www.npmjs.com/package/claude-max-api-proxy)
-   **GitHub:** [https://github.com/atalovesyou/claude-max-api-proxy](https://github.com/atalovesyou/claude-max-api-proxy)
-   **Problemas:** [https://github.com/atalovesyou/claude-max-api-proxy/issues](https://github.com/atalovesyou/claude-max-api-proxy/issues)

## Notas

-   Esta es una **herramienta comunitaria**, no soportada oficialmente por Anthropic o OpenClaw
-   Requiere una suscripción activa a Claude Max/Pro con el CLI de Claude Code autenticado
-   El proxy se ejecuta localmente y no envía datos a servidores de terceros
-   Las respuestas en streaming están totalmente soportadas

## Ver también

-   [Proveedor Anthropic](./anthropic.md) - Integración nativa de OpenClaw con configuración de token o claves de API de Claude
-   [Proveedor OpenAI](./openai.md) - Para suscripciones a OpenAI/Codex

[Cloudflare AI Gateway](./cloudflare-ai-gateway.md)[Deepgram](./deepgram.md)

---