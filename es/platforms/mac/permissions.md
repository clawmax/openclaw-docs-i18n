

  Aplicación complementaria de macOS

  
# Permisos de macOS

Las concesiones de permisos de macOS son frágiles. TCC asocia una concesión de permiso con la firma de código de la app, su identificador de paquete y su ruta en el disco. Si alguno de esos elementos cambia, macOS trata la app como nueva y puede eliminar u ocultar los avisos.

## Requisitos para permisos estables

-   Misma ruta: ejecuta la app desde una ubicación fija (para OpenClaw, `dist/OpenClaw.app`).
-   Mismo identificador de paquete: cambiar el ID del paquete crea una nueva identidad de permiso.
-   App firmada: las compilaciones sin firmar o firmadas ad-hoc no conservan los permisos.
-   Firma consistente: usa un certificado real de Desarrollo de Apple o Developer ID para que la firma se mantenga estable entre recompilaciones.

Las firmas ad-hoc generan una nueva identidad en cada compilación. macOS olvidará las concesiones anteriores y los avisos pueden desaparecer por completo hasta que se borren las entradas obsoletas.

## Lista de recuperación cuando los avisos desaparecen

1.  Cierra la aplicación.
2.  Elimina la entrada de la app en Configuración del Sistema -> Privacidad y Seguridad.
3.  Vuelve a lanzar la app desde la misma ruta y vuelve a conceder los permisos.
4.  Si el aviso aún no aparece, restablece las entradas de TCC con `tccutil` e inténtalo de nuevo.
5.  Algunos permisos solo reaparecen después de un reinicio completo de macOS.

Ejemplos de restablecimiento (reemplaza el ID del paquete según sea necesario):

```bash
sudo tccutil reset Accessibility ai.openclaw.mac
sudo tccutil reset ScreenCapture ai.openclaw.mac
sudo tccutil reset AppleEvents
```

## Permisos de archivos y carpetas (Escritorio/Documentos/Descargas)

macOS también puede restringir el acceso a Escritorio, Documentos y Descargas para procesos de terminal/en segundo plano. Si las lecturas de archivos o los listados de directorios se bloquean, concede acceso al mismo contexto de proceso que realiza las operaciones de archivo (por ejemplo, Terminal/iTerm, una app lanzada por LaunchAgent o un proceso SSH). Solución alternativa: mueve los archivos al espacio de trabajo de OpenClaw (`~/.openclaw/workspace`) si quieres evitar concesiones por carpeta. Si estás probando permisos, firma siempre con un certificado real. Las compilaciones ad-hoc solo son aceptables para ejecuciones locales rápidas donde los permisos no importan.

[Registro de macOS](./logging.md)[Control Remoto](./remote.md)