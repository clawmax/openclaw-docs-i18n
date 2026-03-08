

  Experimentos

  
# Exploración de Configuración de Modelos

Este documento captura **ideas** para futuras configuraciones de modelos. No es una especificación de lanzamiento. Para el comportamiento actual, consulta:

-   [Modelos](../../concepts/models.md)
-   [Conmutación por error de modelos](../../concepts/model-failover.md)
-   [OAuth + perfiles](../../concepts/oauth.md)

## Motivación

Los operadores desean:

-   Múltiples perfiles de autenticación por proveedor (personal vs trabajo).
-   Selección simple de `/model` con respaldos predecibles.
-   Separación clara entre modelos de texto y modelos con capacidad de imagen.

## Posible dirección (alto nivel)

-   Mantener la selección de modelos simple: `proveedor/modelo` con alias opcionales.
-   Permitir que los proveedores tengan múltiples perfiles de autenticación, con un orden explícito.
-   Usar una lista global de respaldo para que todas las sesiones fallen de manera consistente.
-   Solo anular el enrutamiento de imágenes cuando esté configurado explícitamente.

## Preguntas abiertas

-   ¿La rotación de perfiles debe ser por proveedor o por modelo?
-   ¿Cómo debería la interfaz de usuario presentar la selección de perfil para una sesión?
-   ¿Cuál es la ruta de migración más segura desde las claves de configuración heredadas?

[Investigación sobre Memoria del Espacio de Trabajo](../research/memory.md)

---