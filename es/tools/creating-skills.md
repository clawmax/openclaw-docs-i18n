title: "Cómo Crear Habilidades Personalizadas para Asistentes de IA OpenClaw"
description: "Aprende a extender tu asistente de IA OpenClaw creando habilidades personalizadas. Guía paso a paso para definir habilidades, añadir herramientas y seguir las mejores prácticas."
keywords: ["habilidades openclaw", "crear habilidad personalizada", "desarrollo de habilidades", "extensibilidad asistente ia", "tutorial skill.md", "clawhub", "herramientas multi-agente", "espacio de trabajo openclaw"]
---

  Habilidades

  
# Creando Habilidades

OpenClaw está diseñado para ser fácilmente extensible. Las "Habilidades" son la forma principal de añadir nuevas capacidades a tu asistente.

## ¿Qué es una Habilidad?

Una habilidad es un directorio que contiene un archivo `SKILL.md` (que proporciona instrucciones y definiciones de herramientas al LLM) y, opcionalmente, algunos scripts o recursos.

## Paso a Paso: Tu Primera Habilidad

### 1\. Crear el Directorio

Las habilidades residen en tu espacio de trabajo, normalmente en `~/.openclaw/workspace/skills/`. Crea una nueva carpeta para tu habilidad:

```bash
mkdir -p ~/.openclaw/workspace/skills/hello-world
```

### 2\. Definir el SKILL.md

Crea un archivo `SKILL.md` en ese directorio. Este archivo usa frontmatter YAML para metadatos y Markdown para instrucciones.

```
---
name: hello_world
description: Una habilidad simple que saluda.
---

# Habilidad Hello World

Cuando el usuario pida un saludo, usa la herramienta `echo` para decir "¡Hola desde tu habilidad personalizada!".
```

### 3\. Añadir Herramientas (Opcional)

Puedes definir herramientas personalizadas en el frontmatter o instruir al agente para que use herramientas del sistema existentes (como `bash` o `browser`).

### 4\. Actualizar OpenClaw

Pídele a tu agente que "actualice las habilidades" o reinicia el gateway. OpenClaw descubrirá el nuevo directorio e indexará el `SKILL.md`.

## Mejores Prácticas

-   **Sé Conciso**: Instruye al modelo sobre *qué* hacer, no sobre cómo ser una IA.
-   **Seguridad Primero**: Si tu habilidad usa `bash`, asegúrate de que las instrucciones no permitan la inyección de comandos arbitrarios desde entradas de usuario no confiables.
-   **Prueba Localmente**: Usa `openclaw agent --message "use my new skill"` para probar.

## Habilidades Compartidas

También puedes explorar y contribuir con habilidades en [ClawHub](https://clawhub.com).

[Sandbox y Herramientas Multi-Agente](./multi-agent-sandbox-tools.md)[Comandos de Barra](./slash-commands.md)