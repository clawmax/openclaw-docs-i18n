

  Primeros pasos

  
# Primeros Pasos

Objetivo: ir de cero a un primer chat funcional con una configuración mínima.

> **ℹ️** Chat más rápido: abre la Interfaz de Control (no se necesita configurar canales). Ejecuta `openclaw dashboard` y chatea en el navegador, o abre `http://127.0.0.1:18789/` en el host del gateway. Documentación: [Panel](../web/dashboard.md) e [Interfaz de Control](../web/control-ui.md).

## Requisitos Previos

-   Node 22 o más reciente

> **💡** Verifica tu versión de Node con `node --version` si no estás seguro.

## Configuración Rápida (CLI)

### Paso 1: Instalar OpenClaw (recomendado)

> **ℹ️** Otros métodos de instalación y requisitos: [Instalar](../install.md).

### Paso 2: Ejecutar el asistente de configuración inicial

```bash
openclaw onboard --install-daemon
```

El asistente configura la autenticación, los ajustes del gateway y canales opcionales. Consulta [Asistente de Configuración Inicial](./wizard.md) para más detalles.

### Paso 3: Verificar el Gateway

Si instalaste el servicio, ya debería estar en ejecución:

```bash
openclaw gateway status
```

### Paso 4: Abrir la Interfaz de Control

```bash
openclaw dashboard
```

 

> **✅** Si la Interfaz de Control se carga, tu Gateway está listo para usar.

## Comprobaciones y extras opcionales

Útil para pruebas rápidas o solución de problemas.

```bash
openclaw gateway --port 18789
```

Requiere un canal configurado.

```bash
openclaw message send --target +15555550123 --message "Hello from OpenClaw"
```

## Variables de entorno útiles

Si ejecutas OpenClaw como una cuenta de servicio o quieres ubicaciones personalizadas para configuración/estado:

-   `OPENCLAW_HOME` establece el directorio principal utilizado para la resolución de rutas internas.
-   `OPENCLAW_STATE_DIR` sobrescribe el directorio de estado.
-   `OPENCLAW_CONFIG_PATH` sobrescribe la ruta del archivo de configuración.

Referencia completa de variables de entorno: [Variables de entorno](../help/environment.md).

## Profundizar

## Lo que tendrás

-   Un Gateway en ejecución
-   Autenticación configurada
-   Acceso a la Interfaz de Control o un canal conectado

## Próximos pasos

-   Seguridad y aprobaciones en mensajes directos: [Vinculación](../channels/pairing.md)
-   Conectar más canales: [Canales](../channels.md)
-   Flujos de trabajo avanzados y desde el código fuente: [Configuración](./setup.md)

[Características](../concepts/features.md)[Descripción General de la Configuración Inicial](./onboarding-overview.md)

---